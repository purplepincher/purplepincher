# Prior art check: voice-triggered GitHub publishing (the "second boat")

Research method: two targeted web searches via `mmx search query`
(2026-07-05), then an honest synthesis pass via `mmx text chat`
(MiniMax-M2.7). This is the companion check to
`ESP32_VOICE_TO_FIRMWARE_PRIOR_ART.md` — that doc checked prior art for
the ESP32/firmware-generation half of the vision; this one checks the
other new piece added to the essay/roadmap ("The second boat" /
"The sharing chapter"): a captain says *publish this* and the agent
creates a real, public GitHub repo under the captain's own account.

## Real findings from web search

- **GetStream.io, "Build a Voice-Controlled GitHub Agent in Python"** —
  the closest real precedent found. A real tutorial building a voice
  agent (Vision Agents + OpenAI Realtime API) that can query branches/
  issues/PRs by voice and, per the search snippet, "create things via
  voice." A genuine proof that voice-to-GitHub-API plumbing works.
- **`livekit/agents`** — a real, general-purpose framework for
  realtime voice AI agents. Not GitHub-specific.
- **LangGraph/Copilot/Claude-Code-adjacent tutorials** ("Make Your
  Agent Work With Git," "Automating GitHub Repo Maintenance with AI
  Agents") — real content showing AI agents operating on git/GitHub
  workflows, but these are about *maintaining existing repos*
  (branches, PRs, issue triage), not *creating a new public repo from
  a captain's local work on request*.
- **PR-Agent** — a real, established open-source PR reviewer. Reviews,
  doesn't create/publish.

## Honest verdict (mmx synthesis, lightly edited)

**Voice-triggered GitHub repo creation/publishing is not documented
working prior art as an integrated product — it is modular prior art.**
The individual primitives are all real and commoditized: voice
interfaces (livekit/agents, OpenAI Realtime API), GitHub repo-creation
APIs (well-documented REST/GraphQL), and AI coding agents that
understand project structure (Copilot, Claude Code, Cursor). But no
single found system chains all of them into "captain says *publish
this*, a real public repo appears under their account, packaged from
the local work." The GetStream tutorial is the closest — it proves the
voice-to-GitHub-API path is technically feasible — but demonstrating
that a wiring is *possible* is a different claim than a *shipped
product* doing it reliably (credentials, naming conflicts, packaging
compiled firmware vs. arbitrary local state, confirmation flows before
a public action).

## What this means for the org's own honesty discipline

This corroborates the "second boat" section's own stated position —
`docs/PARADIGM.md` and `ROADMAP.md`'s "sharing chapter" already say
plainly that voice-triggered publishing doesn't exist today and gets no
engineering time ahead of Step 1's gate. This check found no reason to
revise that framing in either direction: the mechanism is not already
solved elsewhere (so there's no reason to feel behind), and it's not
speculative/infeasible either (so there's no reason to doubt it's
buildable when the time comes) — the primitives are real, the
specific integration is genuinely unbuilt work, consistent with what
the org's own docs already say about themselves.

**No changes to any existing doc are recommended based on this
finding** — it confirms the current framing is accurate rather than
surfacing a correction, unlike the ESP32 mechanism check and the
tagging-precedent check, both of which did require updating existing
text.
