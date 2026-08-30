# Gap-Hunt Round 2: Agreement Drift

**Scope:** Second gap-hunt round across 7 already-hardened repos (nexus-edge-runtime, edge-compiler, flux-cross-assembler, vessel-bridge, openconstruct-esp32, plato-edge, holodeck-c). Tools: goose, opencode, kimi. Every fix independently rebuilt and retested before merge; where a fix added a regression test, the reviewer reverted the fix locally and confirmed the new test genuinely failed on the old code before trusting it.
**Date:** 2026-07-11

---

## What a gap-hunt round is, and why it keeps working

A gap-hunt dispatch gives one already-hardened repo one more real look, with an explicit honest exit: find a genuine remaining defect, or report nothing found. Nothing-found is an acceptable answer, and it has been given in earlier rounds of this mission — but every one of the seven repos in this round yielded a confirmed defect.

This is now the second gap-hunt round run against repos that had already been through the hardening pipeline, and both rounds found real bugs — not style nits, but crashes, silent control-flow corruption, and a physical pin driven to the opposite of its documented state. We won't speculate here about how many more rounds would keep paying off; two data points say only that "already hardened" has not yet meant "exhausted." What the two rounds do offer is a pattern in *what kind* of bug survives hardening, and that pattern is the point of this document.

## The defect class: agreement drift

Every bug in this round has the same shape. Two (or more) parts of the same codebase are each individually well-formed — you can review either side alone and find nothing wrong — but they depend on an unstated, unenforced agreement with each other, and that agreement has silently drifted (or was never true). The failure is not a logic error inside either component. It is the gap between them.

Call it **agreement drift**. The pairings from this round, concretely:

- a bytecode safety validator and the VM it gates (nexus-edge-runtime)
- a scheduling call and the runtime's actual lifetime guarantees (edge-compiler: bare `setTimeout()` where Workers requires `ctx.waitUntil()`)
- an instruction-emit function and the size classifier, disassembler, and doc comment for the same opcode (flux-cross-assembler)
- a field's type annotation and the literal enum value assigned to it (vessel-bridge: `ActuatorType.RELAY` vs `TransportType.RELAY`, masked because both enums share the string `"relay"`)
- a documented input interface and the parser behind it (openconstruct-esp32: README advertises `"on"`/`"high"`/`"true"`; `String::toInt()` returns 0 for all of them, driving the pin LOW)
- a state flag and the operation it is supposed to gate (plato-edge: `_running = True` set *before* `socket.bind()`, so a failed bind bricks the object permanently — with the added subtlety that an out-of-range port raises `OverflowError`, which is not a subclass of `OSError`)
- a function signature and the dispatch table that calls through it (holodeck-c)

Vessel-bridge also contributed a second bug of a closely related shape: a frame decoder that checked `len(data) < 6` but then indexed at an offset taken from a length field *inside the frame itself* — trusting the frame's self-description to agree with the buffer actually received. Noisy UART input crashed the bridge with a real `IndexError`.

None of these are catchable by staring harder at one side. Each required asking whether two sides still agree.

## Three worked examples

**nexus-edge-runtime — validator vs. VM.** The Tier-4 safety validator is the last gate before bytecode deploys to an edge agent. Its valid-opcode set was hardcoded: `set(range(0x20))` plus a stray `op != 0x21` exception. The VM's real Opcode enum covers 0x00–0x20. Net effect: the validator accepted 0x21 as safe to deploy, and running it raises `VMError('Unknown opcode: 0x21')` on the agent. Both components read as plausible in isolation — the validator has a tidy allowlist, the VM has a clean enum. The bug is that the allowlist was a *copy* of the enum, made once by hand, and copies drift. The fix derives the valid set directly from the VM's Opcode enum, so drift is structurally impossible, and the new `tests/test_safety.py` (this module previously had zero test coverage) round-trips every real opcode through both the validator and the VM and asserts they agree — a test of the agreement itself, not of either side.

**flux-cross-assembler — one instruction, four opinions.** On the "edge" target, the emit function encoded LDI/MOVI as 4 bytes (opcode + register + 16-bit immediate). Three other parts of the same codebase — the opcode enum's own doc comment, the instruction-size classifier, and the disassembler — all said 3 bytes. Consequences: disassembly round-trips injected a spurious NOP, and, more seriously, label/jump offsets desynced by one byte, so a JMP placed after an LDI resolved to a stray zero byte (an accidental NOP) instead of its target — silent control-flow corruption in any program combining a load-immediate with a later jump. Note the review trap: three components voting one way and one voting another is invisible unless something forces them into the same room. The fix brought the emit function into line with the other three (3 bytes, 8-bit immediate).

**holodeck-c — signature vs. dispatch table.** `cmd_gossip(Agent*, Room*, const char*)` didn't match the generic handler type `(Agent*, const char*)` that every other command in the same dispatch table used — undefined behavior the moment the command is invoked through the table. Both the function and the table "look fine" on their own; the defect exists only in their relation. The same commit corrected a README claiming the command "broadcasts to the entire world" when the implementation only echoes locally to the sender — a documentation/implementation instance of the same drift.

## What to do differently

Three responses, all already demonstrated inside this mission rather than proposed hypothetically:

1. **Derive, don't copy.** Where one component's correctness depends on matching another's definition, generate the dependent side from the authoritative side's source of truth. The nexus-edge-runtime fix is the template: the validator now imports the VM's enum instead of maintaining a hand-copied allowlist. A copy that can drift eventually will.

2. **Test the agreement, not the sides.** A test suite where the validator passes its tests and the VM passes its tests proves nothing about whether they agree. Several fixes in this round added tests that assert the relationship directly — e.g., round-tripping every opcode through both validator and VM. When you notice an unstated contract between two components, the highest-value test is the one that would fail if they diverge.

3. **Cross-testing is the same idea at the process level.** This mission's practice of having one AI tool independently re-verify another tool's fix — never trusting a self-report — is structurally a second party checking whether two things agree (the claim and the code). It has already earned its keep: in an earlier round it caught a README asserting a signature matched a header exactly when one parameter name didn't. Agreement drift between documentation and implementation is caught by exactly the mechanism that catches it between two code components: force the two sides into direct comparison.

## What this document does not claim

Seven repos with findings, from one account, in one round of dispatches, is not a sampled defect-frequency study. We do not claim agreement drift is the dominant defect class in software generally, or even in this account — only that it is the class that survived our hardening passes and was still present when we looked again, twice. It is plausible that this reflects a selection effect (single-component logic errors get caught earlier precisely because single-component review and single-component tests catch them, leaving cross-component drift as the residue), but that is an inference from two rounds, not an established result. What we can say with confidence: every one of these bugs was verified by rebuild and retest, and every new regression test was confirmed to fail on the pre-fix code before it was believed.
