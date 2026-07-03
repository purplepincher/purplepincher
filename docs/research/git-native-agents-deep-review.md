# Deep Review: SuperInstance/git-native-agents

Verified by cloning the repo (`gh repo clone SuperInstance/git-native-agents --depth 1`), reading every
file in full (the whole repo is 560 lines: `orchestrator.sh` 265 lines, `README.md` 91 lines,
`registry/agents.txt` 5 lines, `LICENSE` 199 lines — nothing else exists, no `tests/`, no CI config), and
then **actually running the script under concurrency** to check the prior research's "likely race
conditions" claim empirically rather than by inspection alone.

Local paths used:
- Clone: `/tmp/claude-1000/-home-eileen/c25f18c4-8752-45a5-bacc-1c3e70a86ab7/scratchpad/superinstance-research/clones/git-native-agents`
- Live concurrency test: `/tmp/claude-1000/-home-eileen/c25f18c4-8752-45a5-bacc-1c3e70a86ab7/scratchpad/superinstance-research/live-test`

## 1. Precise protocol description

Everything lives in one file, `orchestrator.sh`, dispatched via a `case` statement on `$1` (lines
251–265). There is no daemon, no event loop, no polling — "tick" is a manually-invoked, synchronous,
one-shot batch-process call. Reimplementing this in another language requires reproducing exactly these
eight operations:

**Registry** (global, shared state): a flat file `registry/agents.txt`, one absolute workspace path per
line, append-only (line 53). No agent-removal command exists — dead agents are never unregistered, just
skipped at read time via `[ -d "$workspace" ] || continue`.

**`spawn(name, role="worker")`** (lines 21–56):
1. Compute `workspace = $BASE_DIR/agents/$name`. If the directory already exists, warn and abort (no
   locking around this check — see race #1 below).
2. `mkdir -p workspace; git init` **inside** that directory — i.e. each agent is a *separate, independent
   git repository*, not a branch of one repo, and it is nested inside the orchestrator's own repo tree.
3. Set `git config user.name/user.email` to `$name` / `$name@git-agents.local`.
4. Write `AGENT.yaml` with fields `name, role, spawned (ISO-8601), tick (starts at 0), status (idle), workspace`.
5. `mkdir inbox outbox memory`.
6. `git add -A && git commit -m "spawn: agent $name ($role)"`.
7. Append the workspace path to `registry/agents.txt`.

**`send(from, to, msg)`** (lines 58–83): resolve the recipient's workspace, generate
`msg_id = $(date +%s)-$RANDOM` (second-granularity Unix timestamp + a 0–32767 pseudo-random suffix,
*not* a UUID), `cd` into the **recipient's** repo, write `inbox/${msg_id}.md` containing plain
`key: value` lines (`from`, `to`, `timestamp`, `message` — this is YAML-*looking* but is parsed with
`grep`/`cut`, not a YAML parser), then `git add -A && git commit` **in the recipient's repo, as the
sender**. This is the core design idea: the sender directly writes and commits into someone else's git
history rather than going through a queue or a broker.

**`tick(name)`** (lines 86–136): the poll/consume step.
1. Count files matching `inbox/*.md` via `ls | wc -l`. If zero, report idle and return.
2. Re-glob `inbox/*.md` in a `for` loop (a second, independent glob evaluation from step 1 — see race #2).
3. For each message file: parse `from` and `message` fields with `grep`/`cut`, synthesize a canned
   response string (`"processed: $body → computed at $(date +%s)"` — there is no actual agent
   computation, no LLM call, no plugin hook; this is a stub), write it to `outbox/<same filename>`,
   then `mv` the original inbox file to `inbox/.processed-<same filename>`.
4. Read `tick:` out of `AGENT.yaml`, increment it, `sed -i` it back in, `sed -i` `status:` to `working`.
5. `git add -A && git commit -m "tick $new_tick: processed $inbox_count messages"`.

**`remember(agent, key, value)` / `recall(agent, key)`** (138–167): write `memory/$key.txt`, commit, then
`git tag -f "memory/$key" HEAD`. Recall is literally `git show memory/$key:memory/$key.txt`. Memory keys
are therefore also git tag names, so they inherit git ref-name restrictions (no spaces, no `..`, no
leading `-`, etc. — undocumented and unenforced by the script).

**`think(agent, topic)` / `decide(agent, topic)`** (169–203): `think` does `git branch thought/$topic`
(idempotent if it exists) then `git checkout` onto it and commits a stub `thought-$topic.md` file.
`decide` checks out `main` (falling back to `master`) and does `git merge thought/$topic --no-edit`,
swallowing merge failures with `|| true`-style fallback chains — a failed/conflicted merge is not
detected or reported as an error to the caller.

**`fleet()`** (206–234): reads the registry, `cd`s into each workspace, and prints `name/role/tick/status`
from `AGENT.yaml`, plus `git log --oneline | wc -l` (full history scan — O(n) per agent, so O(N·history)
for the whole fleet), inbox/memory file counts, and a count of `thought/` branches.

**`broadcast(from, msg)`** (236–249): reads the registry once and calls `send` once per other agent — O(N)
per call, exactly as the README states.

That's the entire system. There is no consensus mechanism beyond "last commit / last merge wins" — the
README's phrase "merge-based consensus" describes `decide()` performing an ordinary two-way git merge of
one branch into `main` within a single agent's own repo. It is not consensus *across* agents (no agent
ever merges another agent's repo/branch — cross-agent coordination happens only via the inbox file-drop
mechanism). This is worth flagging: the README's framing overstates what's implemented. There's no
distributed voting, no quorum, no CRDT-like merge across independent histories — just "one agent settles
an internal branch."

## 2. Concrete bugs — reproduced live, not just inferred

The prior research's "likely race conditions" was correct in conclusion but unverified. I reproduced two
distinct, concrete races by actually running the script (setup and full transcripts in
`live-test/` under the path above):

### Bug A — concurrent `send` to the same recipient corrupts delivery (orchestrator.sh:79-80)

Spawned `architect` and `builder`, then fired 12 concurrent `send architect builder "..."` calls
(`&` + `wait`). Result: **11 of 12 processes crashed** with
`fatal: Unable to create '.../agents/builder/.git/index.lock': File exists.` and exited 128
(propagated by `set -euo pipefail`). Root cause: `send_message` does `cd "$to_ws"; cat > inbox/....md;
git add -A; git commit` (lines 72–80) with **no locking whatsoever** around the recipient's `.git/index`.
Any two `send`s (or a `send` racing a `tick`, since `tick_agent` does the identical
`git add -A && git commit` pattern at lines 132–133 in the *same* repo) targeting the same agent at the
same time will collide on git's own index lock.

The practically dangerous part: the `cat > inbox/${msg_id}.md` heredoc write (line 73) happens *before*
`git add`/`git commit`, so the message file lands on disk regardless of whether the commit later fails.
In my test, `ls agents/builder/inbox/*.md | wc -l` showed **12 files on disk** but `git log --oneline`
showed only **2 commits** (init + the 1 `send` that won the lock race). So `send` reports "fatal error,
exit 128" to 11 of 12 callers even though the message was actually written — a caller that checks the
exit code and retries would end up re-sending, and a caller that ignores the exit code would think
delivery is fully git-tracked when it silently isn't (until the next `tick` incidentally sweeps it up via
`git add -A`).

### Bug B — concurrent `tick` on the same agent crashes on a TOCTOU `mv` (orchestrator.sh:96-123)

Ran two concurrent `tick builder` calls against an inbox with 12 pending messages. Both processes glob
`inbox/*.md` independently (`ls` at line 96 for the count, a fresh `for msg_file in inbox/*.md` at line
104 for the actual loop — two separate, non-atomic directory snapshots), so both processes iterate the
same 12 filenames. Observed failure:
```
mv: cannot stat 'inbox/1783100889-10381.md': No such file or directory
[1]-  Exit 1                  ./orchestrator.sh tick builder
```
Process 1 already `mv`'d that file to `inbox/.processed-...` (line 123) by the time process 2 tried the
same `mv`; process 2 crashes immediately under `set -e` mid-batch, with whatever outbox files it had
already written left uncommitted and orphaned (they get silently absorbed into the *surviving* process's
`git add -A` at the end, so nothing is lost in this instance, but nothing guarantees that in general —
e.g. if the surviving process had already finished and committed before the crashed one wrote its
outbox file, that file would sit uncommitted indefinitely with no cleanup path). The surviving process
finished cleanly and reported `tick 1, processed 12 messages` — a report that's accurate here only by
luck of timing, not by any correctness guarantee.

A closely related bug I did not need to force-reproduce because it's structurally obvious from the code:
the tick-counter update (lines 127–129) is a plain **read-modify-write with no lock**:
```
local tick=$(grep '^tick:' AGENT.yaml | cut -d' ' -f2)
local new_tick=$((tick + 1))
sed -i "s/^tick: .*/tick: $new_tick/" AGENT.yaml
```
Two `tick` invocations that both survive to this point in the same narrow window will both read
`tick: 0` and both write `tick: 1` — a classic lost update. My test didn't land in that exact window
(one process crashed on the `mv` race first), but the code has no barrier preventing it.

### Bug C — TOCTOU on `spawn` (orchestrator.sh:26-32, not separately reproduced but structurally identical to A/B)

`spawn_agent` checks `[ -d "$workspace" ]` and only *then* does `mkdir -p && git init`. Two concurrent
`spawn foo` calls (or two calls relying on the default `agent-$(date +%s)` name within the same second)
can both pass the existence check before either creates the directory, racing on `git init` in the same
path. Given the demonstrated fragility of concurrent git operations in this same repo (Bugs A/B), this is
credible, not speculative — it's the same category of bug in a third location.

### Bug D — path traversal via unsanitized agent name (orchestrator.sh:24, 64, 88, 143, 159, 173, 195)

`workspace="$BASE_DIR/agents/$name"` is built directly from user input in every function, with no
validation. `./orchestrator.sh spawn "../../../etc/cron.d/x"` would attempt `git init` outside the
intended `agents/` tree. Low severity for the trust model (this is presumably run by one operator
locally), but worth noting since a "5-50 agent fleet" implies scripted/automated spawning where a
malformed or attacker-influenced name string is plausible.

### Bug E — the repo's own example fleet is silently lost (discovered via `git ls-files -s agents/`)

Not a bug in the orchestrator's logic, but a real, verified artifact of the "each agent is a nested git
repo" design: the maintainer ran the tool once (evidenced by `agents/architect`, `agents/builder`,
`agents/analyst`, `agents/memory-keeper`, `agents/scout` appearing in the tree) and then ran
`git add -A` at the **top-level** repo. Because each `agents/<name>/` directory contains its own `.git`,
git recorded them as **gitlinks** (mode `160000`, submodule-style pointers) rather than real content —
confirmed via `git ls-files -s agents/`, which shows five `160000` entries with commit SHAs and zero
tree content. There is no `.gitmodules` file, so those SHAs are dangling: cloning this repo (as I did)
gives you **empty directories** where the maintainer's own demo fleet used to be. The entire "audit trail"
the README advertises (every message a commit, every decision a merge commit) for the example run is
unreconstructable from the published repo. This is a direct, non-hypothetical consequence of the
"agent = nested git repo" architecture: it does not compose with the outer repo unless every agent
workspace is deliberately kept out-of-tree or wired up as real git submodules, and the project itself
didn't do that.

### Minor: license mismatch

`README.md` line 91 says `MIT`; the actual `LICENSE` file is Apache License 2.0. Cosmetic, but indicates
the docs weren't checked against the actual repo contents — consistent with the general pattern of
assertions (README) outrunning implementation (script).

### What's tested

Nothing. `find . -iname "*test*"` and a full file listing turn up zero test files, zero CI config
(no `.github/workflows`, no other CI). The `.gitignore` (present!) contains `/target`, `**/*.rs.bk`,
`Cargo.lock` — Rust-project boilerplate in a pure-bash repo, a small tell that the scaffolding
(`.gitignore`, LICENSE, README) was generated generically rather than authored for this project
specifically. "Tick" processing is described in the README as "processed (simulated agent computation)"
— the code itself admits in a comment (line 112, `# Generate response (simulated agent computation)`)
that no real work happens; it's a string-template stub, not a working task-execution system.

### O(N²) scaling claim: asserted, not measured

The README states the system "scales cleanly to 5–50 agents — beyond that, the O(N²) message-routing
overhead suggests moving to a broker-based architecture," and repeats this in the "Architecture Notes"
section tying it to the account-wide "γ + η = C" framing. I looked for any supporting benchmark, load
test, or even back-of-envelope arithmetic in the repo: there is none. No benchmark script, no numbers, no
measured commit-time or lock-contention data. The only O(N²) *sounding* thing in the actual code is that
`broadcast` is O(N) per call and an N-agent fleet that broadcasts symmetrically would produce O(N²) total
messages fleet-wide over time — a true but generic observation about any all-to-all broadcast pattern,
not something specific to this implementation's cost profile. Given what I *did* measure directly (Bug A:
92% failure rate — 11/12 — on just 12 concurrent writes to a single recipient), the actual bottleneck this
system will hit first is **not** O(N²) message routing — it's git index-lock contention under any
concurrent write load, which shows up already at N=2 concurrent senders, orders of magnitude before
"50 agents" is a relevant limit. The "5–50 agents" ceiling in the README is a plausible-sounding number
with zero empirical backing in the repo, and I'd trust it less after testing than before.

## 3. Recommendation: harden-in-place vs. reimplement

**Harden in place is the right call, not a Rust/Go rewrite**, and I'd push back specifically on the
earlier research's suggestion. Reasoning:

- The actual bugs found (A, B, C) are not architectural — they're missing mutual exclusion around three
  specific `git add && git commit` call sites (`send_message` line 79-80, `tick_agent` line 132-133,
  `spawn_agent` line 49-50) and one read-modify-write (`tick_agent` lines 127-129). All four are fixable
  with `flock` around each per-agent-repo critical section (`flock "$workspace/.git/index.lock.mutex"
  -c '...'` or simpler, wrap the whole function body in `( flock 9; ...  ) 9>"$workspace/.tick.lock"`),
  which is maybe 20-30 lines of added shell, not a rewrite. Git itself already serializes at the index
  level; the fix is making the *script* wait for that instead of racing it.
- The `mv` TOCTOU (Bug B) is fixed by moving the `ls`/`for`-glob duplication to a single snapshot (e.g.
  `find inbox -maxdepth 1 -name '*.md' -print0 | sort -z` captured once) combined with the same
  per-agent flock.
- Atomic message IDs: swap `$(date +%s)-$RANDOM` for something collision-resistant if throughput ever
  matters (mktemp-style or a monotonic counter file under the same lock), though collision wasn't
  actually the failure mode observed — lock contention was.
- None of this requires abandoning "git as the coordination primitive," which is the actual interesting
  idea in the repo (auditable commit trail, branch-based speculative exploration, tag-based memory). A
  Rust/Go reimplementation would have to reproduce all of that *plus* solve the same locking problem
  (calling `git` as a subprocess or via `git2`/`go-git`, either way you still need to serialize writes to
  one repo's index) — it doesn't remove the hard part, it just moves the same 30 lines of fix into a
  different language with more ceremony (process spawning or a git library dependency, config, build
  tooling). The "hard part" here was never language performance; it's mutual exclusion, and shell has
  `flock` built in.
- Where a rewrite *would* pay off: if this needs concurrent-safe operation for real (many agents ticking
  and sending simultaneously, not just sequentially demoed), a language with a real concurrency model
  could replace ad hoc file locking with an in-process work queue per agent and batch git writes — but
  that's a bigger redesign than "port the shell to Go," and nothing in this repo suggests that need is
  proven yet given it currently supports zero real agent computation (the tick stub) and zero tests.
- Fix scope estimate: ~230 lines of actual logic to harden (locking + a snapshot-glob fix + a couple of
  input-sanitization checks on `$name` for Bug D) plus, if this is meant to go into `purplepincher`,
  writing the tests that don't currently exist (spawn/send/tick roundtrip, a concurrent-send stress test
  mirroring what I ran, a concurrent-tick stress test) — call it a half-day to a day of focused work, not
  a rewrite project.

## 4. Is it purplepincher-worthy?

The underlying idea — git objects/tags/branches as the entire coordination substrate for a small agent
fleet, with the audit trail as a first-class property — is still the cleanest *idea* in this account by a
wide margin; that assessment holds up. But "cleanest small idea" was doing work that "lightly assessed,
untested POSIX script" doesn't support as a description of the artifact itself. What I found:

- The implementation is thinner than the README implies: no real per-agent computation (the tick handler
  is a string-template stub, explicitly commented as simulated), no cross-agent consensus (just an
  intra-repo merge), zero tests, zero CI.
- The concurrency bugs are not hypothetical — I reproduced an 92% failure rate on same-recipient
  concurrent sends and a hard crash on concurrent ticks within minutes of running the script. This is a
  system whose one distinguishing claim ("distributed, fault-tolerant agent fleet") fails its most basic
  concurrency test.
- The nested-git-repo design has already eaten the maintainer's own demo data (Bug E) — a concrete
  illustration that "agent = full git repo, nested inside another git repo" has a real composability
  problem, not just a theoretical one.
- None of this is expensive to fix (see §3), and the core mechanism (once locked) genuinely works as
  described for the message/memory/thought-branch primitives — I'd stand behind forking it *after*
  hardening, not as-is.

Net: fork it, but as a **hardening task**, not a straight import, and downgrade the pitch from "the
cleanest small idea, needs a Rust/Go port" to "the cleanest small idea, currently a proof-of-concept with
no locking and no tests — a day of shell hardening plus a real test suite would make it genuinely usable
at the claimed 5-agent scale; the 50-agent ceiling and the O(N²) framing are unsupported marketing copy
that should be dropped or re-derived from an actual benchmark before repeating them in purplepincher's own
docs."
