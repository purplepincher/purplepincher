# ESP32 / Voice-to-Firmware Prior Art in SuperInstance

**Scope:** Is there real, working prior art in `SuperInstance` for the specific
PARADIGM.md direction — a user describes by voice/SL what they want a bare
ESP32 to do, the system writes real compiled firmware for *this* chip, pushes
it over USB, and tells the human how to wire it?

**How this differs from `cocapn-technical-fit.md`:** That earlier pass scored
the embedded repos against cocapn-foundation's *own* substrate requirements
(ActiveLog event schema, profile interpreter, safety supervisor, voice
pipeline). This pass asks a narrower and different question — does anything in
SuperInstance actually do **natural-language → real flashable firmware** (or
even a credible subset of that pipeline), regardless of whether it speaks
ActiveLog? The two questions share some repos but not their verdicts.

**Method:** `gh repo list SuperInstance --limit 4200` (4,200 repos retrieved,
JSON-cached locally), keyword grep across name+description for ESP32/firmware/
embedded/generate/compile/voice/flash, then code-level verification (not README
trust) of every promising hit. The Fleet Vector API
(`fleet-vector-api.casey-digennaro.workers.dev`) was reachable but indexes a
different namespace (`casey-digennaro/*` micro-Rust crates), not SuperInstance,
so it was not useful for this question. GitHub code-search
(`org:SuperInstance` qualifiers) was used to catch anything description-trawling
missed; no additional hits surfaced.

---

## Honest verdict, up front

**No single repo in SuperInstance implements the end-to-end voice-to-firmware
partnership described in PARADIGM.md. That pipeline would be built from
scratch.** But — and this is the more useful finding — the org already contains
**two real, verified pieces of the pipeline that the earlier research pass did
not flag**, because the earlier question was different:

1. `describe-device` is a genuine, working **NL → flashable .ino sketch +
   platformio.ini + wiring SVG** generator. It is narrow (single-constraint
   keyword parser, no LLM, no compile, no flash) but it is the *exact product
   shape* of the "describe it, get firmware" half, and it ships in a 57 KB
   static HTML file with zero backend.
2. `ESP-Flasher` is a real, working **browser-based USB flasher** for ESP32 /
   S2 / S3 / C3 / C6 using Web Serial, with proper esptool-style bootloader
   sync, chip identification, register reads, and serial monitor. It is a fork
   of upstream `bharanidharanrangaraj/ESP-Flasher` (so the flashing logic is
   not original SuperInstance code), but it has been re-pitched in
   `AGENT_ARCHITECTURE.md` as the agent→hardware bridge and is genuinely the
   "push firmware over USB" half.

Two other repos are real and adjacent but fill different roles:
`handy-marine-voice` does **voice → runtime commands** (regex grammar over
transcription, dispatches to an autopilot), not voice → firmware; and
`plato-vessel-core` has the *infrastructure* for an agent to push an opaque
"intelligence" string to a device and bump its capability level, but the
device never interprets or executes that string — it stops at storage.

Everything else with "ESP32" or "firmware" in the name either (a) is a
runtime/interpreter that consumes commands at runtime, not a generator
(`flux-esp32`, `flux-lcar-esp32`, `openconstruct-esp32`, `plato-engine-block-c`,
`plato-vessel-core`'s tool dispatch), or (b) is spec/markdown only
(`plato-vessel-educational`, `plato-vessel-technician`, `hav-flux-bridge`,
`vessel-bridge`, `polyglot-flux-ese`, `Vibe-Code-Agent-Gen`).

The **external prior-art check** corroborates this: no shipped or research
tool combines NL/voice input → real chip-specific compiled firmware → automatic
USB push → wiring instructions in one product. The pieces exist scattered
across the industry (MakeCode for blocks→firmware→USB; Arduino Cloud for
OTA/USB flash; Claude Code/Cursor for LLM-generated embedded C;
`masonzeng702550/chatgpt_arduino_program_creater` for a primitive NL→compile
script; `RaffaeleSpezia/detect-build-upload` for agent-driven flash/verify).
The specific combination PARADIGM.md describes — especially the voice input
and the auto-generated wiring guidance — appears genuinely novel, both inside
and outside SuperInstance.

---

## Per-repo verification (SuperInstance)

Each entry below was verified by reading the actual source, not the README.
Quote/file:line evidence is given for every claim.

### `describe-device` — REAL, narrow, exactly the right shape

- **Path:** `SuperInstance/describe-device` (HTML, size 19 KB, 1 commit,
  pushed 2026-05-09). Three files: `README.md`, `SKETCH_TEMPLATE.ino`
  (2.2 KB), `index.html` (57 KB, 1,669 lines).
- **What it actually does:** A static, zero-backend web page that takes a
  natural-language device description and produces a real, downloadable
  Arduino/ESP32 sketch + `platformio.ini` + wiring SVG.
- **The NL parser** (`index.html:707-905`, the `PARSER` object) is keyword-
  based, not LLM-based. It detects:
  - **One sensor** from 8 patterns: temperature, motion/PIR, humidity/DHT,
    light/LDR, soil moisture, pressure/BMP280, distance/HC-SR04, gas/MQ.
    Defaults to `TEMP` if nothing matches (`index.html:759-764`).
  - **One or more actuators** from 8 patterns: relay, LED, fan, valve, servo,
    buzzer, pump, heater. Defaults to `RELAY`.
  - **One numeric threshold** via a single regex,
    `/(\d+)\s*(°[cCfF]|degrees|celsius|fahrenheit)?/i` (`index.html:795`).
  - **One comparison operator** from word lists (`>`/`<`/`==`),
    (`index.html:813-818`).
  - The expressible intent is exactly: "if `<sensor> <op> <threshold>` then
    `<actuator>` = `<state>`" — one constraint per actuator, one threshold per
    device. Nothing else.
- **The constraint solver** (`index.html:623-702`, `Guard2MaskSolver`) is a
  real, working AC-3 arc-consistency port (the `revise()` and `solve()`
  methods are textbook AC-3), not a stub. It actually prunes domains and
  returns SAT/UNSAT.
- **The sketch generator** (`index.html:1122-1265`, `generateSketch()`) emits
  real, sensor-specific Arduino C: per-sensor reader code (e.g., DS18B20
  `requestTemperatures()` / `getTempCByIndex(0)`, DHT22 `readHumidity()`,
  HC-SR04 `pulseIn()` distance), per-sensor pin maps, per-actuator `pinMode`
  + `digitalWrite` logic, a PLATO tile emission over Serial, and a header
  comment that lists the exact GPIO connections. The `platformio.ini`
  generator (`index.html:1270-1293`) emits per-sensor `lib_deps` (Dallas
  Temperature, DHT, etc.). The `downloadSketch()` function
  (`index.html:1642-1648`) actually downloads the generated code as a real
  `.ino` file via a Blob.
- **What it does NOT do:** No compilation (the user takes the .ino elsewhere);
  no USB flashing; no voice input; no LLM; no chip-specific targeting (always
  `esp32-devkitc`); no safety state analysis (PARADIGM.md's "watchdog fails to
  off" idea is nowhere here); no resolution of "this" against a log; one
  threshold per device, period.
- **Why it matters:** This is the closest existing thing in SuperInstance to
  the product shape PARADIGM.md describes, and it is a real, runnable artifact
  you can open in a browser today. Its scope is the right scope for a v0:
  narrow, keyword-based, single-constraint. The leap to "voice in, compiled
  firmware out, USB push, human wiring instructions" is large but the seed is
  genuine.

### `ESP-Flasher` — REAL USB flasher, forked from upstream, re-pitched

- **Path:** `SuperInstance/ESP-Flasher` (JavaScript, size 125 KB, fork of
  `bharanidharanrangaraj/ESP-Flasher`). Files: `index.html` (21 KB),
  `index.css`, `js/{app.js, esp-device.js, esp-protocol.js, serial-manager.js}`
  (~1,529 LOC total), plus `AGENT_ARCHITECTURE.md` (9.6 KB) and a
  `Serial Monitor/` directory.
- **What it actually does (verified in source):**
  - `serial-manager.js:60-93` calls `navigator.serial.requestPort()` and
    `connect()` — real Web Serial API, real serial connection.
  - `esp-protocol.js:103-149` implements both `classicReset` (DTR/RTS
    sequencing) and `usbJtagReset`, with a comment noting these are
    "exact match to esptool-js" timing — i.e., real bootloader entry.
  - `esp-device.js:11-107` enumerates every known ESP chip-family magic value
    ("from esptool.py source") and does `readReg`/`tryReadReg`/`detectChip`/
    `_identifyESP32Variant`/`readMacAddress`/`readCrystalFreq`/`readChipRevision`/
    `readFlashId` — this is real chip identification by esptool methods.
  - `app.js:476-531` is `_flashFirmware()` — bootloader sync retries and the
    entry point for the actual flash write; `_sendData()` at line 605+ does
    the data transfer.
- **What is original vs. forked:** The flashing protocol logic is from the
  upstream repo, not written by SuperInstance. What *is* original is the
  repackaging pitch in `AGENT_ARCHITECTURE.md` and the README's "agent →
  firmware → ESP32 worker" framing, which positions the tool as the
  push-firmware-over-USB half of the PARADIGM.md direction.
- **What it does NOT do:** No firmware generation; no NL or voice input; no
  chip selection from intent; no compile step. It is a *consumer* of `.bin`
  files, not a producer. It is also a fork, so crediting it as "SuperInstance
  built a USB flasher" would be inaccurate — SuperInstance adopted one and
  re-pitched it.

### `handy-marine-voice` — REAL voice grammar, but voice → runtime commands, not firmware

- **Path:** `SuperInstance/handy-marine-voice` (Rust, size 28 KB, pushed
  2026-06-13). Source: `src/{main.rs, grammar.rs (13.9 KB), bridge.rs (17.6 KB)}`,
  `Cargo.toml`, plus five Markdown docs. README claims "29 passing tests."
- **What it actually does:** A regex-based voice-command grammar over a
  transcribed utterance. `grammar.rs:34-46` defines `enum MarineCommand` with
  variants like `HoldHeading(f64)`, `SetSpeed(f64)`, `SetDeadband(f64)`,
  `ReportDepth`, `Escalate(String)`. `parse_command()` in `grammar.rs:83+`
  matches each variant with a specific regex (e.g.,
  `r"^(hold|steer|set|go to?)\s*(heading|course)?\s*(\d{1,3})..."` for heading
  commands). `bridge.rs` then maps commands to CoCapn actions.
- **What it does NOT do:** It does not generate firmware, does not flash
  anything, does not produce code. It dispatches **runtime** commands to a
  hypothetical existing autopilot. The README itself admits (Phase 2 section)
  that the actual integration with `cocapn-marine` and `cocapn-core` is
  unimplemented — those crate dependencies are not real yet.
- **Why it matters:** This is the right pattern for the *voice-grammar* half
  of a voice-to-firmware system, and at much higher fidelity than
  `describe-device`'s keyword parser — it handles relative turns, units,
  escalations, multi-word phrasings. The grammar approach (regex-over-text
  rather than LLM-over-text) is the same approach DeckBoss's existing
  ActiveLog voice system uses, so this is consistent with the org's
  voice-first philosophy. But it is voice-to-runtime-action, not
  voice-to-firmware.

### `plato-vessel-core` — REAL embedded C, but the "intelligence push" is a stub

- **Path:** `SuperInstance/plato-vessel-core` (already examined in
  `cocapn-technical-fit.md` and `11-named-repos.md`). The narrower question
  here: does its MCP "turbo-shell" / "intelligence upgrade" mechanism actually
  push behavior (firmware or interpreted code) to the device?
- **Verified answer: no.** `plato_mcp.c:135-158` (`mcp_handle_command`) checks
  for an `intelligence` JSON field, and if present, copies the string into
  `reg->capability_json` with `strncpy`, then bumps `reg->cap_level` from
  `PLATO_CAP_RAW` toward `PLATO_CAP_ENSIGN` — that's it. The intelligence is
  *stored, never interpreted or executed*. The tool dispatch that follows
  (`plato_mcp.c:166-183`) has a comment that admits it: *"In a real
  implementation, each tool would have a handler function. Here we return
  success with the tool name echoed back."*
- **Why it matters:** This is the *infrastructure* for an agent to push
  intent/behavior to a device — there's a wire format, a registry, a
  capability-level state machine, and a handshake. But the device side of the
  handshake is a stub. If you wanted to build the PARADIGM.md direction on
  this base, you would have to (a) teach the device to actually execute the
  pushed intelligence — by either interpreting it as bytecode or compiling it
  to firmware — and (b) generate the intelligence in the first place from a
  voice transcript. Neither piece exists.

### Other ESP32/embedded-named repos in SuperInstance — verified NOT prior art

Each was inspected; none generates firmware from intent. They are listed here
to close the loop on the broader search.

- **`ESP32-Plane-Radar`** (already examined): Standalone Arduino/PlatformIO
  firmware that fetches ADS-B data over HTTPS and renders on a GC9A01 LCD.
  Real firmware, no generation, no NL layer.
- **`openmind-esp32-bridge`** (already examined): Host-side Rust RPC library
  to toggle GPIO/i2c/etc. on an ESP32 over a custom CRC-8 protocol. There is
  no ESP32 firmware in the repo. Not firmware generation.
- **`plato-vessel-technician`** (already examined): Pure documentation.
  Zero source files.
- **`flux-esp32`** (C, 7 KB): A single `main/flux_vm.{c,h}` pair (17 KB +
  9 KB) implementing a Flux bytecode VM in C. Real code, but it *consumes*
  pre-compiled bytecode — it does not generate it. It would be the
  *execution target* for a NL→Flux→ESP32 pipeline, not the generator.
- **`flux-lcar-esp32`** (C, 18 KB): LCAR ("Logical Compute Allocation &
  Routing") header + 344-byte `main.c`. Sparse; same shape — runtime, not
  generator.
- **`openconstruct-esp32`** (C++, 14 KB): Arduino library exposing
  `OpenConstructESP32 node("name")` with `registerDigitalSensor()`,
  `registerAnalogSensor()`, etc., plus a `CommandParser.cpp` that parses text
  commands like `"read door"` or `"write 4 HIGH"` over MQTT. Real, but it is
  a runtime command interpreter on the device, not a firmware generator.
- **`plato-engine-block-c`** (C, 42 KB, pushed 2026-07-04 — the most recent
  hit): Real sensor→history→alarm engine in C99 with TCP server, tick loop,
  ring buffer. Real code, recently iterated. But again: a runtime that takes
  text commands (`tick`, `history`, `actuator`) at runtime, not a generator.
- **`vessel-bridge`** (Python, 13 KB, just `LICENSE + README + src/`):
  Pitched as "Hardware abstraction layer for ESP32 to Jetson to Cloud." The
  `src/` exists but the top-level has only LICENSE and README — tiny, did
  not deep-verify; description is HAL not generator.
- **`intent-flux-bridge`** (Rust, 15 KB, single `src/lib.rs`):
  Keyword-based NL → Flux bytecode compiler. Real, but targets **GPU
  compute** patterns (`Map`, `Reduce`, `MatMul`, `Scan`, `Consensus`,
  `Attention`), not embedded firmware. Demonstrates the "LLM as compiler"
  pattern within SuperInstance for a different target.
- **`flux-vm-agent`** (Python, 27 KB): FLUX bytecode VM + assembler +
  interpreter + `lcar_bridge.py`. The description's "NLP interpreter" claim
  is misleading — `lcar_bridge.py` is an opcode-to-fleet-task translator,
  not natural-language processing.
- **`agentic-compiler`** (Python, 151 KB): A Python JIT optimizer that
  profiles hot functions and recompiles them to Numba/Rust/CUDA at runtime.
  Despite the description's "Markdown-to-runtime agentic compilation"
  framing, there is no markdown or NL input layer; it's runtime
  auto-optimization. Not embedded, not NL-driven.
- **`flux-multilingual`** (7.4 MB, mostly JSON dumps): A research/synthesis
  repo. Not a firmware generator.
- **Spec/markdown only** (no source code, verified): `plato-vessel-educational`
  ("Students focus on physical design — agents handle the firmware" — pure
  pitch, no code), `hav-flux-bridge`, `polyglot-flux-ese`, `Vibe-Code-Agent-Gen`,
  `seed-nexus-bootstrap`, `nexus-edge-runtime`. These all *sound* on-topic
  from their descriptions but contain no implementation.

---

## External prior-art sanity check

A targeted external scan (verified by fetching actual homepages and reading
actual READMEs, not search-result snippets) found the field genuinely sparse.
Ordered closest → least close to the full vision:

1. **`masonzeng702550/chatgpt_arduino_program_creater`** (verified) — A
   Python script that calls ChatGPT to produce Arduino code, then invokes
   `arduino-cli.exe` to compile it. NL → compiled firmware, but no voice, no
   USB flash, no wiring guidance. 0 stars, 3 commits; effectively invisible.
   Closest thing to "NL → real compiled firmware" the search found.
2. **`RaffaeleSpezia/detect-build-upload`** ("PROGRAMMO") (verified) — An
   ESP32 detect → configure → build → flash → verify toolkit over USB with
   JSON stdout "so agents can parse, decide, and iterate." Real and
   agent-shaped, but no NL/voice layer and no code generation. (Worth
   noting: the README's "AUTONOMOUS_TOOLS/programmo" framing reads like it
   may be adjacent to an agent-driven firmware stack; verify author
   independence before citing as independent prior art.)
3. **General LLM coding agents on embedded targets** (Claude Code, Cursor,
   Cline — verified via Arduino Project Hub tutorial using Claude Code +
   Ollama on the Arduino UNO Q): General-purpose agents can and do emit
   embedded C today, but none has embedded-specific modes, voice, or
   auto-wiring-instruction features. A workflow, not a product.
4. **Arduino Cloud** (`create.arduino.cc`, verified): Real cloud IDE + OTA
   firmware push including ESP32. Compile/flash/config platform with **no
   NL codegen layer** (`arduino.cc/en/products/ai` returns 404).
5. **Microsoft MakeCode** (`makecode.microbit.org`, verified): Block editor ↔
   JavaScript with in-browser compile to `.uf2` drag-flashed over USB. Real
   "generate → USB-flash" UX, but the input is blocks/JS, not language.
6. **Wokwi** (`wokwi.com`, verified): Browser simulator for Arduino/ESP32/
   STM32/Pico. `wokwi.com/ai` returns **404**; no AI codegen feature exists
   despite being commonly mis-cited as "Wokwi AI."
7. **`d-beamon/chatgpt-arduino-cloud`** (verified): An Arduino sketch that
   proxies prompts to OpenAI at runtime. NL → runtime responses, explicitly
   not codegen; its own roadmap lists voice + NL-action-parsing as
   aspirational.
8. **ESP-RainMaker** (`rainmaker.espressif.com`, verified): Cloud SDK +
   claim/provision for ESP32 IoT. Config/runtime platform, no NL codegen.

**Honest negatives checked:** No verifiable Microsoft "AutoFlash" or
NL-IoT shipped product. arxiv returned **0 results** for "natural language"
AND embedded AND firmware. Seven `gh search repos` queries
(`llm esp32`, `natural language arduino`, `chatgpt arduino`,
`voice arduino firmware flash`, `llm arduino codegen`,
`embedded ai agent code generation`, `iot code generation llm`) returned
almost entirely runtime-on-device projects, not generators.

**External verdict:** The individual halves exist scattered across the
industry. The specific combination in PARADIGM.md — *voice* input →
*chip-specific compiled* firmware → *automatic* USB push → *human-readable
wiring instructions* — does not exist in any shipped or research tool I could
verify. The voice-input and auto-generated-wiring-guidance pieces appear
genuinely novel.

---

## Bottom line for the mission

**Is there real code in SuperInstance worth drawing on for this direction?**

Yes — two specific, named pieces, both verifiable today, neither flagged by
the earlier cocapn-foundation pass because that pass was asking a different
question:

1. **`describe-device`** is the natural starting point for the NL→firmware
   generator half. It already does keyword parsing → constraint program →
   CSP solve → real .ino + platformio.ini + wiring SVG, end to end, in a
   single static HTML file. The path from here to PARADIGM.md is: (a) widen
   the keyword parser to an LLM call (DeckBoss already has a voice pipeline
   to reuse), (b) widen the expressible logic past one threshold per device,
   (c) add chip-specific targeting, (d) hand the generated sketch to a
   compiler, (e) hand the compiled image to `ESP-Flasher`.
2. **`ESP-Flasher`** is the natural starting point for the push-over-USB half.
   It is a fork of upstream `bharanidharanrangaraj/ESP-Flasher`, so
   attribution matters, but the flashing protocol is real and works today in
   a browser via Web Serial.

Two further real repos fill supporting roles without being direct prior art:
**`handy-marine-voice`** is the right pattern (and the right level of
fidelity) for the voice-grammar half if the org decides to keep voice
parsing deterministic rather than LLM-mediated; and **`plato-vessel-core`**
has the wire-format scaffolding for an agent to push *something* to a device,
though the device-side execution of that something would have to be built.

**Is there real code in SuperInstance that already does the full
voice-to-firmware partnership?** No. The full pipeline — voice transcript in,
compiled chip-specific firmware out, USB push, human wiring instructions —
exists nowhere in SuperInstance, and based on the external scan, nowhere
else either. The honest answer to "would this be built from scratch?" is:
*the integration* would be built from scratch, but *not from a blank page* —
the org already owns real seeds for both halves (NL→sketch in
`describe-device`; USB→chip in `ESP-Flasher`), plus real supporting patterns
in `handy-marine-voice` and `plato-vessel-core`. That is a more useful
finding than either a strained "yes, it already exists" or a deflationary
"nothing here, start from nothing."
