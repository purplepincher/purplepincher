# cocapn.ai — direct browser-to-ESP32 upload feasibility

**Status:** R&D findings, drafted 2026-07-08 on branch
`rd-esp32-webserial-2026-07-08`. Scope: whether Casey's stated vision —
"the chatbot can vibe-code the interface and create the code for uploading to
the ESP32… later apps will have uploading directly from the app, lowering the
barrier of entry" — is buildable with today's *real* web platform, and how.
This is the future-phase hardware-control work `cocapn.ai` exists for
(`ROADMAP.md` §"Step 5", `fishinglog.ai`/`cocapn.ai` framing), explicitly parked
out of the ActiveLog reference core
([`docs/ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) §7). It
must not undermine `cocapn-foundation`'s real five-layer safety design
([`SAFETY.md`](https://github.com/purplepincher/cocapn-foundation)), mirrored at
`purplepincher/deckboss/docs/cocapn-foundation-mirror/SAFETY.md`.

This document reuses the org's three honesty markers, identical in meaning to
ActiveLog's:
- **✅ Real today / buildable to the bar.** Verifiable now in shipping browsers
  or libraries, no research breakthrough required.
- **⚠️ Real but conditional.** Works, but only on some browsers, only with a
  user setup step, or only with a stated tradeoff the UI must surface honestly.
- **🔮 Later phase, not day one.** Genuinely buildable eventually, but needs a
  real toolchain artifact or an org-level decision that shouldn't be made
  silently. Never labeled "coming soon" on a shipped page.

**The brief that requested this research contains an outdated assumption worth
correcting on the record up front:** it says "Firefox and Safari have
historically not supported [WebSerial], confirm current state." Verified answer:
Safari still does not, but **Firefox shipped the Web Serial API on desktop in
Firefox 151 on 2026-05-19** — about seven weeks before this doc was written. The
Firefox gap is closed on desktop (with caveats below); the iOS gap is total;
the Android-USB gap is real. The detailed table is §1.

---

## Bottom line, up front

Direct-from-browser ESP32 flashing is **genuinely buildable today for the
flashing step, but not for the compile step** — and that split is the entire
honest story:

- **Flashing a pre-built binary to an ESP32 over USB from a browser tab: ✅ real
  today.** Espressif's own `esptool-js` does exactly this, is actively
  maintained (v0.6.0, 2026-03-26), and is the canonical library (§2).
- **Producing that binary in the first place: ⚠️ to 🔮 depending on language.**
  There is **no production-quality browser-side or WASM compiler** for ESP32
  C/C++ firmware, so a compiled-firmware path genuinely requires either a local
  toolchain the user runs, or a server-side build service this org would have to
  operate — the latter being a direct tension with the "no backend we operate"
  principle (§3). **MicroPython/CircuitPython eliminate the compile step** and
  are the real low-barrier path (§3).
- **"The chatbot vibe-codes the interface and one-click uploads to the ESP32":**
  buildable end-to-end *only* if the firmware target is MicroPython (or the
  binary is pre-built and the page just flashes it). For a C/C++ firmware
  pipeline, the honest interim UX is **the chatbot generates real, correct
  source code + clear flashing instructions, and the browser flashes an
  already-built `.bin`** — without claiming one-click source-to-device that is
  not actually possible yet (§5).

This split maps cleanly onto the org's existing honesty discipline: the
`fishinglog-agent` hardening (`ROADMAP.md`, "What we will not fork") was exactly
about a capability claimed in copy that the code never implemented. Claiming
"one-click upload from your browser" when what actually happens is "we flash a
binary you still had to compile elsewhere" would be the same failure mode,
restated for hardware.

---

## 1. Web Serial API — real, current status (2026-07-08)

The Web Serial API (`navigator.serial`) is a real, shipping WICG specification.
It is how a web page, over a user gesture, opens a raw serial port (a
USB-serial bridge like the ESP32's CP210x/CH340) and reads/writes bytes. MDN
marks the `Serial` interface **"Limited availability"** — "not Baseline because
it does not work in some of the most widely-used browsers," requires a **secure
context (HTTPS)**, and `requestPort()` requires **transient activation** (a real
user click) ([MDN, `Serial`](https://developer.mozilla.org/en-US/docs/Web/API/Serial),
last modified 2026-05-19).

### Verified browser support table (MDN browser-compat-data, read 2026-07-08)

| Browser | Status | Detail |
|---|---|---|
| Chrome desktop | ✅ 89+ | Full. |
| Edge desktop | ✅ 89+ | Mirrors Chrome. |
| Opera desktop | ✅ | Mirrors Chrome. |
| **Firefox desktop** | ✅ **151+** | **Shipped 2026-05-19** (Firefox bug 2029625). ⚠️ But each origin's access is gated behind a **synthetically generated site-permission add-on** the user must install — "the same approach used to safely manage access to WebMIDI" (Firefox 151 release notes, MDN developer notes). Heavier UX than Chrome. |
| Chrome Android | ◐ 138+, partial | ⚠️ **USB serial is NOT exposed.** "Serial ports are only available if they're provided by Bluetooth RFCOMM serial port emulation" (BCD note). So an ESP32 over a USB-OTG cable does **not** work via Web Serial on Android Chrome. |
| Firefox Android | ❌ | Not supported (BCD `version_added: false`). |
| Safari (desktop) | ❌ | Not supported (BCD `version_added: false`). |
| Safari iOS | ❌ | Not supported (mirrors Safari). |
| All iOS browsers | ❌ | iOS browsers are all WebKit-under-the-hood by App Store policy, so **no browser on iOS supports Web Serial at all** — including "Chrome" and "Firefox" on iOS, which are WebKit shells. |
| WebView Android | ❌ | Not supported. |

Mozilla's *standards position* on the Serial API is **neutral** (issue
`mozilla/standards-positions#336`, label `position: neutral`), which is a
statement about the spec, not a promise to ship — though they have now shipped
it anyway. WebKit has not published a positive position and does not implement
it. **Treat the API as "Chromium-derived engines + recent Firefox desktop, and
nothing on phones or tablets."**

### The real permission/pairing UX a user goes through

The flow a stranger actually experiences, verified against MDN and the
`esptool-js` usage notes:

1. **User gesture required.** The page cannot probe for devices on load. A real
   click (a "Connect" button) calls `navigator.serial.requestPort({ filters:
   [{ usbVendorId, usbProductId }] })`. The vendor/product filter (e.g.
   `0x10c4`/`0xea60` for the Silicon Labs CP210x on most ESP32 dev boards)
   narrows the picker to likely devices.
2. **Browser device picker.** The browser shows its native "Select a serial
   port" dialog listing OS-visible serial ports. This is the OS's port list, not
   the page's, so the page never silently enumerates hardware.
3. **Origin-scoped permission grant.** The user picks a port and approves. The
   grant is **origin-scoped and persistent** for *that specific port*: the page
   can later re-acquire it via `navigator.serial.getPorts()` (which returns only
   ports the origin already has permission for) without re-prompting. The user
   can revoke it from browser settings at any time. (Firefox additionally
   wraps this in the site-permission-add-on step noted above.)
4. **Bootloader entry is software-driven.** ESP32 boards enter download mode via
   a DTR/RTS sequence on the USB-serial bridge, not a physical BOOT button press
   on most modern dev boards. `esptool-js` performs this sequence
   (`ClassicReset`/`UsbJtagSerialReset`), so the user does not normally have to
   hold a button — but on some boards a manual BOOT/EN press is still needed,
   and the page should say so when the auto-reset fails rather than spinning.

**The honest one-liner for this section:** "works on Chrome/Edge/Firefox
desktop, over a USB cable, after a one-click browser device picker; does not
work on iPhone or iPad at all, and on Android only for Bluetooth-serial, not
USB-serial, so it is not the path for phone-based flashing."

---

## 2. Real ESP32 flashing over Web Serial — `esptool-js`

There is a real, working, maintained JS library that flashes a compiled binary
to an ESP32 over Web Serial with no desktop toolchain: **`esptool-js`**
([`espressif/esptool-js`](https://github.com/espressif/esptool-js)).

### What it is (verified against the repo, 2026-07-08)

- **First-party and canonical.** Maintained by **Espressif themselves** (the
  chip vendor), Apache-2.0, TypeScript. It is the JS port of the Python
  `esptool` every ESP32 developer already uses. ~513 stars, 176 forks, 227
  commits.
- **Actively maintained.** Latest release **v0.6.0 on 2026-03-26**, with v0.5.7
  the same day and a steady cadence (v0.5.6 2026-06-17, v0.5.5 2026-05-28…).
  Recent releases added brand-new chips: ESP32-C5 (v0.4.6, 2025-10), ESP32-C61
  (v0.4.7, 2025-10), ESP32-H2 (v0.5.6, 2026-06). This is not abandonware; it
  tracks the chip family.
- **The API surface that matters:** `navigator.serial.requestPort()` →
  `new Transport(port)` → `new ESPLoader({...})` → `esploader.main()` (connect,
  auto-detect chip family) → `esploader.writeFlash({ fileArray: [{ data:
  Uint8Array, address: 0x1000 }], flashMode, flashFreq, flashSize, compress,
  reportProgress })` → `esploader.after("hard_reset")`. Real bootloader sync,
  real chip identification by magic value, MD5 verification, compressed
  transfer, flash-mode/freq/size configuration — the same protocol `esptool.py`
  speaks. Live demo at `https://espressif.github.io/esptool-js/`.
- **The critical, load-bearing limitation, in Espressif's own words:**
  > "Unlike the Python-based esptool, `esptool-js` doesn't implement generation
  > of binary images out of ELF files, and doesn't include companion tools
  > similar to `espefuse.py` and `espsecure.py`."
  (README, emphasis added.)

  **`esptool-js` is a consumer of `.bin` files, never a producer.** It expects a
  pre-built firmware image as a `Uint8Array` plus a flash address. It does not
  compile, link, or sign. This single fact is the hinge of the whole
  feasibility question (§3).

### Firmware format it expects

A raw binary flash image at a byte offset: typically
`{ data: <firmware.bin>, address: 0x1000 }` for a classic ESP32 or ESP32-S2/S3
(the second-stage bootloader offset); the RISC-V C-series (C3/C5/C6/H2) use
`0x0`. For a full image you flash multiple parts (bootloader, partition
table, app) at their respective offsets, or one merged image. `flashMode`
(`qio`/`qout`/`dio`/`dout`), `flashFreq` (`40m`/`80m`/…), and `flashSize`
(`4MB`/…) are set explicitly. This is exactly what MicroPython's published
`.bin` is shaped for (§3).

### Org-internal corroboration

The SuperInstance survey already found this library's sibling-in-spirit:
`SuperInstance/ESP-Flasher` is a fork of upstream
`bharanidharanrangaraj/ESP-Flasher`, a real browser USB flasher for ESP32/S2/S3/
C3/C6 using Web Serial with "exact match to esptool-js timing" bootloader entry,
chip identification, and serial monitor
([`docs/research/esp32-prior-art-findings.md`](./esp32-prior-art-findings.md),
"ESP-Flasher"). So the org has already adopted and re-pitched a working flasher
once. **Attribution matters there (it's a fork, not original code); `esptool-js`
upstream is the thing to depend on directly.** The flashing half of the pipeline
is not a research problem — it is a "wire up `esptool-js`" problem.

**Verdict for §2: ✅ real today.** Flashing a pre-built ESP32 binary from a
browser tab, over USB, on Chrome/Edge/Firefox desktop, is a solved,
maintained, vendor-blessed capability.

---

## 3. "The chatbot vibe-codes the interface" — the compile step is the real crux

For the chatbot's generated code to actually reach the chip, *something* must
turn source into a flashable artifact. The options are not equivalent, and the
honesty of the whole feature depends on naming which one is real.

### 3a. Compile C/C++ → `.bin` in the browser (WASM toolchain): 🔮 not real today

Producing an ESP32 firmware binary from C/C++ requires the full ESP toolchain:
the Xtensa (or RISC-V) cross-compiler (`xtensa-esp32-elf-gcc` or
`riscv32-esp-elf-gcc` / the LLVM-based `esp-clang`), the ESP-IDF SDK or the
Arduino-ESP32 core, linker scripts, partition table generation, and the
second-stage bootloader. That is a large native toolchain.

**There is no production-quality browser-side or WASM-compiled ESP32
toolchain.** Targeted search (`gh search repos "esp32 wasm compiler"`,
`arduino compile browser wasm`) returned no such project; the org's own prior
external scan found the field empty of browser-native embedded compilers —
"arxiv returned 0 results for 'natural language' AND embedded AND firmware"
and every "browser embedded compile" hit turned out to be a simulator or a
server-backed IDE
([`docs/research/esp32-prior-art-findings.md`](./esp32-prior-art-findings.md),
"External prior-art sanity check"). The genuinely in-browser compile tools that
*do* exist are not ESP32 tools:

- **Microsoft MakeCode** compiles blocks/JS → `.uf2` for drag-flash — but targets
  the micro:bit / RP2040 / Arcade, **not ESP32**, and its input is blocks/JS,
  not natural language.
- **Wokwi** simulates ESP32/Arduino in the browser, but **compile is
  server-side** (it is a simulator, not a compiler), and `wokwi.com/ai` returns
  404 — there is no AI codegen feature despite being commonly mis-cited as one
  (same prior-art doc).

So: a real "type a description → the browser compiles C firmware for this chip
→ flashes it" pipeline **does not exist and is not imminent**. Building one
would mean compiling a cross-toolchain to WASM and shipping it — a multi-year,
high-risk effort, not a feature. 🔮 Label it a research goal, never a roadmap
claim.

### 3b. MicroPython / CircuitPython — the real "no compile" path: ⚠️ real, with a two-step caveat

MicroPython eliminates the compile step entirely. The interpreter is published
as a **pre-built `.bin`** (ESP32_GENERIC v1.28.0, 2026-04-06, with nightly
preview builds tracking v1.29.0 as of 2026-07-06;
[micropython.org/download/ESP32_GENERIC](https://micropython.org/download/ESP32_GENERIC/)),
flashed once at `0x1000` via the exact `esptool` command `esptool-js`
reproduces. Once the interpreter is on the chip, **user code is `.py` source
pushed to the filesystem** — no compilation, no toolchain. This is the genuine
low-barrier path: the chatbot emits a `main.py`, and the problem collapses to
"flash the MicroPython `.bin` once, then transfer `.py` files."

**The caveat that keeps this ⚠️ and not ✅:** the polished, standard file-push
tool (`mpremote`) is a **Python desktop program**, not a browser library. There
is no `esptool-js`-equivalent standard JS library for "upload `.py` files to a
running MicroPython over Web Serial." It is technically buildable — the
MicroPython REPL is just a serial port, and raw-mode file transfer
(`ampy`/`mpremote cp` protocol) is a thin JS layer over the same
`navigator.serial` stream `esptool-js` already opens — but today you would write
or adopt that transfer layer yourself. The two real sub-steps are:

1. **One-time: flash the MicroPython firmware `.bin`.** ✅ Use `esptool-js`,
   directly. This is the solved step.
2. **Ongoing: push `.py` user code to the live interpreter over the same serial
   port.** ⚠️ Buildable today, but the org writes the JS file-transfer shim
   (there is no vendor-canonical library for it the way `esptool-js` is for
   flashing). CircuitPython softens this further: a CircuitPython board mounts
   as a **USB mass-storage drive** (`CIRCUITPY`), so on desktop the user can
   drag `code.py` — but that path does **not** use Web Serial and does not work
   from a browser (no USB-MSC access from the web without the device already
   presenting as a drive).

**Honest framing:** "MicroPython makes the chatbot's Python code run on the
chip with no compile step — but the page still has to flash the interpreter
once and then push your `.py` over the serial port, which works on desktop
Chrome/Edge/Firefox and is not yet a polished one-click on phones."

### 3c. Server-side build service (Arduino Cloud, ESP-IDF cloud build): ⚠️ real capability, 🔮 for this org — the named tension

The realistic way to compile C/C++ firmware without the user installing a
toolchain is a **hosted build service**: Arduino Cloud
([create.arduino.cc](https://create.arduino.cc)) and Espressif's own cloud
build compile source server-side and return a `.bin`. This is real and works.

**But this is, by definition, a backend this org operates** if `cocapn.ai` does
it itself, and that is the exact tension the brief asks to be named. It is the
**same shape, and the same word, as the ActiveLog free-token-allowance
problem** ([`docs/ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) §5,
"The free-allowance problem"):

> A "limited free tokens on a cheap model" feature requires an org-operated
> endpoint… by definition, a backend this org operates — and the org has a
> documented rejection of exactly that shape.

Swap "chat-turn proxy" for "firmware-compile worker" and the structure is
identical: both are org-operated, stateless-ish endpoints that take user input
and return a result, neither touches the user's log data, both carry a cost
liability (a compile worker is CPU-expensive — minutes of build per request —
so the wallet-drain vector is worse than a chat proxy). The same three
constraints that make a chat allowance defensible apply here, *mutatis
mutandis*:

1. **Day one: no org-operated compiler.** The page flashes a `.bin` the user
   supplies, or uses MicroPython (no compile). Keeps the no-operated-backend
   property literally true.
2. **Phase 2 (optional, gated):** a build Worker only if (a) it is also
   deployable to the user's own Cloudflare account (the BYO-Cloudflare pattern
   the vision names), (b) it is hard-capped per client (compile minutes are
   expensive — the cap is more load-bearing here than for chat tokens), and
   (c) it is provably source-blind (it compiles submitted source and returns a
   binary; it never sees log data).

Note the **third-party option that dodges the operated-backend question
entirely**: route the chatbot's generated C/C++ through **Arduino Cloud** (or a
GitHub-Actions build, or the user's own ESP-IDF install) and flash *that*
returned `.bin` via `esptool-js`. That defers the backend to someone who already
operates one (Arduino/Espressif/GitHub) and is honest about it — at the cost of
an Arduino/GitHub account the user must have. This is a legitimate interim
shape and is called out in §5.

**Verdict for §3: the compile step is the honest bottleneck.** MicroPython
side-steps it; C/C++ requires either the user's own toolchain, a third-party
hosted build, or — only under the same gated constraints ActiveLog §5 already
established — an org-operated build worker that must be named as a real tension
with the "no backend we operate" principle, not smuggled in silently.

---

## 4. Safety interplay — a browser-flash path must not undermine the five-layer model

`cocapn-foundation`'s `SAFETY.md` defines a five-layer hardware defense for
voice-commanded steering (electrical; firmware watchdog + 500ms command TTL;
authenticated replay-proof single-master protocol; voice command-class
escalation; sea-trial checklist). The load-bearing question for this R&D is
whether "the browser can flash arbitrary firmware" creates a hole in that model.
The honest answer is **no new hole, provided the flashing is treated as a
provisioning action and never as an actuation path** — but three points must be
made explicit so a future implementer cannot get this wrong silently.

1. **Layers 1 and 2 are hardware/runtime guarantees that protect against a *bad*
   flash by construction.** Layer 1 is "unpowered / crashed / unplugged = open
   circuit; no command is ever latched in hardware." Layer 2 is "outputs release
   on a 500ms TTL unless a live heartbeat extends them; the hardware watchdog
   resets a stalled chip; outputs are open during boot." These guarantees are
   deliberately **independent of whether the firmware is good** — that is their
   entire purpose. A browser that flashes buggy or even malicious firmware
   cannot defeat a de-energized-safe relay wiring or a hardware watchdog, because
   those are not software. **This is why the SAFETY design is robust to a
   browser flasher: it was designed to be robust to bad firmware of *any*
   origin.** The right framing for a future `cocapn.ai` flash UI is the sentence
   `SAFETY.md` already puts on every wiring card: the unit is a remote-control
   accessory and the vessel's master remains responsible — flashing software
   from a web page does not change that line, and the page should not imply it
   does.
2. **The genuine new risk surface is "flash firmware that *removes* the
   watchdog/TTL/replay protection."** Because `esptool-js` writes an arbitrary
   `.bin` to arbitrary addresses, a malicious or merely careless flash could
   replace the supervisory firmware with code that omits layer 2's TTL or layer
   3's authenticated session. The mitigation is **not technical secrecy** (the
   bootloader is documented and the flash path cannot be lock-out-able without
   also locking out legitimate updates); it is **provenance and the separate
   sea-trial gate** (layer 5). Concretely: the firmware the page offers to flash
   should be a **named, reviewable build** (a pinned version + hash of firmware
   the org or the user audited), never an opaque blob; and a re-flashed unit
   must re-pass the guided sea trial before it is marked provisioned — exactly
   as `SAFETY.md` layer 5 already requires for any install. This is
   device-profiles-as-data territory (`ROADMAP.md` "What's actually worth
   absorbing": new device support ships as a reviewable file, never a firmware
   fork).
3. **A browser flash must never become a cloud actuation path.** `SAFETY.md`'s
   "What voice never does (v1 policy)" is categorical: "No actuation from
   dictation mode, chat mode, or any cloud path. Actuation originates only from
   the local command pipeline holding the helm token." Flashing is a
   **provisioning/maintenance action, not an actuation** — it happens while the
   chip is in bootloader mode (not steering), requires a physical USB
   connection and an explicit user gesture, and the running helm-pipeline is not
   live during it. As long as the flash path is modeled as provisioning
   (layer 5's domain) and never wired into the live command pipeline (layers
   3–4), it does not undermine the "actuation is local-only" invariant. The
   day someone proposes "let the chatbot push firmware over WiFi while the boat
   is running," that is a *new* proposal that must clear layers 3–5 on its own
   merits — and this doc should not be cited as precedent for it.

**Net:** the flashing capability in §2 is compatible with `SAFETY.md` precisely
*because* `SAFETY.md` was engineered to survive untrusted firmware at the
hardware level. The org's job is to keep flash-as-provisioning cleanly
separated from steer-as-actuation, and to surface provenance (what `.bin`, what
hash) rather than to pretend a flasher can be made "safe" by hiding the
bootloader.

---

## 5. Honest assessment and recommended interim UX

**Is direct-from-browser ESP32 flashing genuinely buildable with today's real
web platform?** Yes for the flash step (✅, §2), no for an in-browser C/C++
compile step (§3a). Therefore:

**The realistic path does *not* yet equal "one-click, the chatbot uploads my
code to the ESP32."** It equals one of two honest shapes:

### Shape A — MicroPython (the genuine low-barrier path) ⚠️→✅ as the tooling matures
1. The chatbot generates **MicroPython source** (`main.py`) from the user's
   description.
2. The page flashes the **pre-built MicroPython firmware `.bin`** once via
   `esptool-js` (solved).
3. The page pushes the generated `.py` to the chip over the same Web Serial
   port (buildable; org writes or adopts a JS file-transfer shim).
- **Honest copy:** "Generates MicroPython for your ESP32 and flashes it from
  the browser — on Chrome, Edge, or Firefox desktop, over USB. Not on phones
  or tablets yet."

### Shape B — C/C++ firmware with an explicit compile provenance ⚠️
1. The chatbot generates **real, correct C/C++ source** (`.ino`/`platformio.ini`
   or ESP-IDF) plus a wiring card — the exact product shape `SuperInstance/
   describe-device` already proved (keyword parser → constraint solve → real
   sketch + `platformio.ini` + wiring SVG, in a static HTML file; see
   [`docs/research/esp32-prior-art-findings.md`](./esp32-prior-art-findings.md),
   "describe-device").
2. The page offers **three honest compile paths, labeled for what they are:**
   - **(b1) Compile it yourself:** copy-paste the generated source into your
     local Arduino IDE / PlatformIO / ESP-IDF, build, and the page flashes the
     resulting `.bin` via `esptool-js`. (User runs the toolchain.)
   - **(b2) Compile it on Arduino Cloud / a GitHub Action:** the page hands the
     source to a **named third party** that already operates a build service,
     receives the `.bin`, and flashes it. (Backend exists, operated by Arduino/
     GitHub, not by us — say so.)
   - **(b3) We compile it:** 🔮 **off day one.** Only under the ActiveLog §5
     three-constraint test (BYO-Cloudflare-deployable, hard per-client cap on
     compile minutes, provably source-blind). Name explicitly as an operated
     backend.
3. The page flashes the `.bin` via `esptool-js`, surfaces the firmware hash,
   and — for a `cocapn.ai`/Helm-class unit — requires the re-flash to re-clear
   the `SAFETY.md` layer-5 sea trial before the unit is marked provisioned.

### What the shipped page must *not* claim on day one
- Not "one-click upload from any device" (no iOS, no Android-USB).
- Not "the chatbot compiles your C code in the browser" (no such toolchain
  exists).
- Not "upload from your phone" (USB serial over Web Serial does not work on
  Android or iOS).
- Not "safe because the flashing is locked down" (it is not, and should not be;
  safety lives in the hardware per `SAFETY.md`, not in flash secrecy).

This is the `fishinglog-agent` discipline applied to hardware: the page either
does what it says or it marks the capability 🔮 and omits it — never a "coming
soon" banner for one-click device flashing that is not actually one-click.

---

## 6. Open uncertainties (stated, with recommendations)

- **The MicroPython-over-Web-Serial file-push layer.** `esptool-js` solves
  flashing; nothing vendor-canonical solves "upload `main.py` to a running
  MicroPython from a browser." **Uncertain:** whether to adopt an existing JS
  `ampy`-equivalent, write one, or rely on the REPL paste-mode.
  **Recommendation:** before any skin ships a MicroPython "vibe-code" path,
  spike a minimal JS raw-mode file writer over `navigator.serial` and verify it
  against a real ESP32 on all three desktop engines. Treat it as the one
  genuinely new piece of code this feature needs.
- **Firefox's site-permission-add-on UX in practice.** Firefox 151 ships Web
  Serial but gates it behind an add-on install per origin. **Uncertain:** how
  much that step depresses real-world completion vs Chrome's one-click picker,
  and whether it is stable across Firefox ESR. **Recommendation:** test the
  flash flow on a current Firefox stable *and* ESR before counting Firefox as a
  first-class target; if the add-on step is fragile, surface "Chrome/Edge
  recommended; Firefox supported with an extra permission step."
- **Phone-based flashing via WebUSB instead of Web Serial.** Web Serial on
  Android is RFCOMM-only, but **WebUSB** (broader Android support) *can* reach
  the ESP32's native USB (S3/C-series with USB-OTG) — and `esptool-js` notes
  Android Chrome works "via the `web-serial-polyfill` compatibility layer"
  over WebUSB. **Uncertain:** how reliably that polyfill handles real ESP32
  boards across phone/adapter matrix. **Recommendation:** do not claim
  phone flashing until a real phone+board test passes; until then, phones are
  🔮 and the page says "desktop only."
- **The hosted-build cost cap (Shape B, b3).** A compile Worker is far more
  expensive per request than a chat-turn proxy (minutes of CPU per build vs
  milliseconds). **Uncertain:** a defensible per-client minute quota.
  **Recommendation:** if/when b3 is proposed, quantify build minutes and cost
  the cap *before* building, the way ActiveLog §5 quantified DeepSeek tokens —
  the cap is more load-bearing here, not less.

---

### Sources cited (all verified 2026-07-08)

- MDN, [`Serial`](https://developer.mozilla.org/en-US/docs/Web/API/Serial) — "Limited availability," HTTPS + transient activation required; last modified 2026-05-19.
- MDN browser-compat-data, `api/Serial.json` — chrome 89; edge/opera mirror; firefox 151; chrome_android 138 partial (RFCOMM-only); firefox_android false; safari false; webview_android false.
- [Firefox 151 release notes for developers](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/151) and [Firefox 151.0 release notes](https://www.mozilla.org/en-US/firefox/151.0/releasenotes/) — Web Serial API shipped 2026-05-19, desktop only, via site-permission add-on (Firefox bug 2029625).
- `mozilla/standards-positions` [issue #336](https://github.com/mozilla/standards-positions/issues/336) — position: neutral.
- [`espressif/esptool-js`](https://github.com/espressif/esptool-js) — official Espressif JS flasher, Apache-2.0, TypeScript; README ("doesn't implement generation of binary images out of ELF files"); [releases](https://github.com/espressif/esptool-js/releases) v0.6.0 (2026-03-26) + v0.5.x.
- [MicroPython ESP32_GENERIC firmware](https://micropython.org/download/ESP32_GENERIC/) — v1.28.0 (2026-04-06), pre-built `.bin`, flashed via `esptool.py … write_flash 0x1000`.
- Internal: [`docs/ACTIVELOG_ARCHITECTURE.md`](../ACTIVELOG_ARCHITECTURE.md) §5 (free-allowance / operated-backend tension) and §7 (ESP32 upload out of core scope); [`ROADMAP.md`](../../ROADMAP.md) §"Step 5"; [`docs/research/esp32-prior-art-findings.md`](./esp32-prior-art-findings.md) (`describe-device`, `ESP-Flasher`, external scan); [`docs/research/ESP32_VOICE_TO_FIRMWARE_PRIOR_ART.md`](./ESP32_VOICE_TO_FIRMWARE_PRIOR_ART.md); `cocapn-foundation` [`SAFETY.md`](https://github.com/purplepincher/cocapn-foundation) (five-layer model, mirrored at `purplepincher/deckboss/docs/cocapn-foundation-mirror/SAFETY.md`).

---

*Findings evidence-cited throughout; the compile-step gap (§3a) and the
phone/iOS gaps (§1) are stated as real limitations, not hand-waved, because a
hardware-control feature that overclaims its browser reach is exactly the
liability surface `SAFETY.md` exists to keep honest.*
