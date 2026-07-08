# Finish this phase — two items, in scoring distance

*Operator documentation, not decision record — see [`ROADMAP.md`](../ROADMAP.md)
for the reasoning. Everything below is Casey-executable without
re-reading the roadmap. Written 2026-07-08, synthesizing an independent
round-table review (aider, mmx/MiniMax) and a Fable synthesis pass.*

Two things close this phase: `pincher` live on crates.io, and the first
real captain using DeckBoss. Everything else is either done (CI green
across all 11 graduated repos, re-verified 2026-07-08) or deliberately
waiting behind the Step 1 gate defined in `ROADMAP.md`.

## A. Publish `pincher` to crates.io (~20 minutes, one credential)

**Precondition, confirmed satisfied as of 2026-07-08:** the formatting
fix, the `pincher-cli` → `pincher` package rename, and a full
`cargo publish --dry-run` pass are done, pushed to `main`
(commit `e84b13a`), and CI is green on that commit. There is nothing
mechanical left — only the steps below, which require Casey's own
crates.io identity.

Names confirmed available on crates.io as of 2026-07-08 via the live
API: `pincher`, `pincher-core`, `pincher-cli` all return "does not
exist." This claim goes stale with time — the publish steps below
re-check it implicitly (a taken name fails loudly at publish time).

**Steps, in order:**

1. Log in at https://crates.io with GitHub, then create an API token at
   https://crates.io/settings/tokens with `publish-new` and
   `publish-update` scopes.
2. `cargo login` — paste the token when prompted.
3. `git clone https://github.com/purplepincher/pincher && cd pincher`,
   then `cargo fmt --check && cargo test --workspace` — expect clean
   fmt and the full suite green (340 tests as of the last run; if you
   see a different count, stop and ask why before publishing).
4. Publish the library first — the CLI depends on it:
   ```bash
   cargo publish -p pincher-core --dry-run
   cargo publish -p pincher-core
   ```
5. Wait a minute or two for the index, then the binary crate:
   ```bash
   cargo publish -p pincher --dry-run
   cargo publish -p pincher
   ```
6. **Acceptance test, from a directory outside the repo:**
   ```bash
   cargo install pincher && pincher --version
   ```
   This exact command working is adoption-bar item 4 and one of the
   org's stated measures of success. It is the definition of done here.
7. **Same-day doc flips**, once step 6 actually passes (the org's own
   discipline: docs match reality the day reality changes — hand this
   to an agent once you've confirmed the install works, don't do it
   preemptively):
   - `pincher/README.md` — remove the "not yet on crates.io" honesty
     note, replace with the working install command.
   - `purplepincher/README.md` — the pincher bullet's "not yet
     published" line.
   - `purplepincher/ROADMAP.md` Step 2 — status changes from "not fully
     done" to done, adoption-bar item 4 met, dated.

## B. Get the first real captain using DeckBoss

**Step 0 — commit the draft before it evaporates.** Two recruiting
message drafts were produced in an earlier round-table (kimi's and
GLM's — mmx picked GLM's as the stronger one, for making the ask
directly rather than circling it). As of 2026-07-08 neither was
committed anywhere durable, and this mission's working scratchpad has
already been wiped by machine restarts twice. `deckboss/docs/beta-recruiting/RECRUITING_COLLATERAL.md`
(mmx's own longer-form collateral: forum post, dock flyer, SMS
templates) already exists and is committed — every claim in it is
traceable to the live landing page and `ROADMAP.md`'s own "Human action
items" section, with placeholders (`[contact info]`, `[link]`) left
intentionally for Casey to fill, not invent.

**Step 1 — prep the message (10 minutes).**
- Add a time-bound hook per mmx's review: "looking for 3–4 captains
  over the next two weeks."
- Fill the collateral's placeholders — contact info, video walkthrough
  link if one exists.

**Step 2 — send it.** First 5 names you'd actually want in the beta,
direct message or text, personally, from you. Log each ask (name, date,
channel) — `ROADMAP.md`'s new fallback clock (Step 1) runs off this
log.

**Step 3 — when someone says yes, three things in order** (from
`deckboss/ROADMAP.md`'s human action items 1–3, compressed):
1. Device census by text: what phone, what browser.
2. A 15-minute in-person setup: install the PWA to the home screen
   (not a bookmark — home-screen install is what grants persistent
   storage on iOS), connect their own Drive/R2/Oracle storage, and
   watch the first entry actually land in their storage before you
   leave. Nobody leaves the dock unsynced.
3. Week one runs in parallel with their current log. Their records risk
   nothing.

**Step 4 — the first week, operationally.**
- Daily five-minute check-in by call or text, not a form.
- Don't coach, don't pitch features, no advanced-feature conversations
  in week one. First naive contact is nonrenewable.
- **"It's working" looks like:** entries landing in their own storage
  daily; at least one recording made in genuinely loud or wet
  conditions without you asking; the captain mentioning something it
  got wrong — complaints mean use.
- **"It's not landing" looks like:** mid-week silence; "I stopped when
  my hands were wet"; needing re-teaching after setup day; logging only
  at the dock in the quiet. The moments they *don't* reach for the
  phone are the highest-grade signal available — write those down the
  day you hear them.
- Check the inbound channel weekly at `https://deckboss.net/admin/beta-signups`
  with your admin token — the signup form is live and verified working,
  and a signup nobody reads is worse than no form.

## One unrelated two-minute item while you have a browser open

kimi is locked out pending your device-code login — it's the only team
member currently down, and no agent can fix it for you.
