---
id: MCE-PRD-001
title: "PRD — MVP: Headless Blink Vertical Slice"
status: proposed
---

# PRD — MVP: Headless Blink Vertical Slice

| Field | Value |
| --- | --- |
| Status | Proposed |
| Owner | Project maintainers |
| Target | v0.1 — Headless foundation |
| Related | [Roadmap](../roadmap.md), [Architecture](../architecture.md), [Evidence](../evidence.md) |

## Summary

Build the first end-to-end slice of the product without a UI or physical hardware. A command-line host loads one example project, validates its packages and nets, runs a small `no_std` firmware against a native simulator, and writes a deterministic trace. The slice uses one reference board and three components: an LED, a push-button, and a potentiometer.

The purpose is to prove the seams that the rest of the product depends on: project schema, package catalog, logical nets, firmware ports, backend contract, and trace identity. Catalog breadth and visual editing come later.

## Problem

**User:** a maker or firmware developer evaluating the product.

**Current behavior:** nothing is implemented. There is no way to load a project, validate its packages, or run firmware against a simulated board.

**Problem:** the architecture claims that simulation and hardware can share one project and one device contract, but no vertical slice demonstrates it. Without a slice, the schemas and boundaries are untested guesses.

**Impact:** building the desktop UI or a physical backend first would hide contract mistakes behind more code. The project needs a small, headless proof before investing in either.

## Goal

- [ ] Load and validate the example project entirely from the command line.
- [ ] Run the example firmware against a native simulator for a bounded virtual time.
- [ ] Produce a deterministic trace whose hash is stable across two runs.
- [ ] Prove that a scripted button press changes the observed behavior.
- [ ] Record the results in [Evidence](../evidence.md) with the exact commands and artifacts.

## Scope

| In scope | Out of scope |
| --- | --- |
| One example project with one board, three components, and logical nets. | The desktop UI, canvas, inspector, and visual editing. |
| Project schema v1, lockfile, and catalog validation for the example. | User-authored packages, a registry, or a marketplace. |
| Native simulator with integer time, GPIO, ADC, and three component models. | I2C, SPI, UART, PWM, displays, sensors, or other buses. |
| One `no_std` Rust firmware using `embedded-hal` 1.0 traits. | Arduino/C++, MicroPython, or arbitrary firmware emulation. |
| Scripted input fixture for the button. | Interactive input from a keyboard or a physical board. |
| Headless CLI: `lint` and `run`. | Compiler/flasher adapters and a physical backend. |
| Canonical JSONL trace and SHA-256 hash. | Logic-analyzer UI and long-term trace storage. |
| Linux CI as the first evidence platform. | A cross-platform determinism claim. |
| Documentation updates for the slice. | A public release, installer, or license decision. |

## User Flow

1. User clones the repository and runs `cargo run -p playground-cli -- lint projects/examples/blink`.
2. The CLI resolves the board and component packages, validates the project, and prints a summary.
3. User runs `cargo run -p playground-cli -- run projects/examples/blink --trace out/blink.jsonl`.
4. The simulator boots the firmware, applies the scripted button input, and writes the trace.
5. The CLI prints the run duration, event count, final LED state, and trace hash.
6. User runs the same command again and compares the hash.
7. User opens the trace and sees the LED period change with the potentiometer value and the LED stop while the button is held.

## Requirements

### R1 — Project and example

- [ ] The project declares a schema version, a board package reference, component instances, logical nets, board binding, pin aliases, a firmware reference, a seed, and a duration.
- [ ] The example lives in `projects/examples/blink/` and is the only project required by this PRD.
- [ ] The board is `mce.reference-board` version `0.1.0`, a host-simulation board package with eight GPIO pins, two ADC channels, two PWM channels, and a UART. It is not a claim about a specific MCU.
- [ ] The components are `mce.led` (ports `anode`, `cathode`), `mce.push-button` (ports `a`, `b`), and `mce.potentiometer` (ports `vcc`, `wiper`, `gnd`).
- [ ] The nets are `net_status_led`, `net_button`, `net_blink_rate`, `net_3v3`, and `net_gnd`.
- [ ] The aliases are `status_led` → `net_status_led`, `user_button` → `net_button`, and `blink_rate` → `net_blink_rate`.
- [ ] A lockfile pins every package version and content hash. Opening the project with a different package version is a validation error until the lockfile is updated.
- [ ] The board binding is:
  - `gpio0` → `net_status_led`, output;
  - `gpio1` → `net_button`, input with pull-up;
  - `adc0` → `net_blink_rate`, 12-bit ADC;
  - `3v3` → `net_3v3`;
  - `gnd` → `net_gnd`.
- [ ] The project does not contain raw MCU register numbers or backend-specific settings.

### R2 — Catalog validation

- [ ] `playground-cli lint <project>` resolves packages from the catalog roots in the order defined by [ADR-0002](../adr/0002-runtime-package-catalog.md).
- [ ] Validation rejects:
  - a duplicate package ID with a different content hash;
  - a component port that does not exist in its manifest;
  - a net with no members or an unknown member;
  - a board pin that does not exist or lacks the required capability;
  - a net with more than one active digital driver;
  - a missing or incompatible package schema version;
  - a missing firmware reference or an incompatible firmware/board combination;
  - a malformed lockfile.
- [ ] Errors are reported with the file, the object path, and a stable error code. The CLI exits non-zero on any error.
- [ ] A valid project passes lint with no warnings that change behavior.

### R3 — Native simulation

- [ ] The scheduler uses integer microseconds and orders events by `(time, insertion_sequence)`.
- [ ] The run advances until the project duration or an explicit fault, whichever comes first.
- [ ] GPIO output drives a net; the LED model reflects the resolved state with its configured series resistance.
- [ ] GPIO input with pull-up reads `high` when the button is open and `low` when it is pressed to ground.
- [ ] The ADC returns a 12-bit value derived from the potentiometer position, with no floating-point value in the trace identity.
- [ ] The button model exposes a contact state driven by the input fixture.
- [ ] The simulator enforces a virtual-time limit and an event limit; exceeding the event limit produces a typed fault.
- [ ] The simulator does not read wall-clock time, the filesystem, the network, or OS randomness.

### R4 — Firmware path

- [ ] The example firmware is a `no_std` Rust crate using `embedded-hal` 1.0 traits for digital output, digital input, and ADC.
- [ ] The firmware refers to `status_led`, `user_button`, and `blink_rate` through board/project aliases, not fixed pin numbers.
- [ ] The firmware contains no host dependencies and no simulator-specific code.
- [ ] Host-simulation adapters implement the same traits for the native backend.
- [ ] The same firmware source is intended to build for a later physical board through a board-specific adapter; this PRD does not implement that adapter.

### R5 — Headless CLI and trace

- [ ] `playground-cli lint <project>` validates the catalog and project and prints a summary.
- [ ] `playground-cli run <project> --trace <path>` loads the project, runs the simulation, writes the trace, and prints a summary.
- [ ] `playground-cli run` accepts an optional `--seed` override for experiments; without it, the project seed is used.
- [ ] The trace is canonical UTF-8 JSONL: one event per line, keys in schema order, no extra whitespace, and a trailing newline. The trace schema is versioned.
- [ ] The trace contains at least: `run_start`, `pin_change`, `adc_sample`, `input`, and `run_end`. A fault is recorded as a `fault` event before `run_end`.
- [ ] `run_start` records the project ID, board ID and version, seed, trace schema version, and input fixture hash.
- [ ] The trace hash is SHA-256 over the exact trace file bytes.
- [ ] The CLI exits `0` on success, `1` on validation error, and `2` on a runtime fault.

### R6 — Determinism and evidence

- [ ] Two consecutive runs of the example with the same seed, lockfile, and input fixture produce identical trace hashes.
- [ ] Changing the input fixture changes the trace hash.
- [ ] A golden trace is committed as a test artifact, with a documented regeneration command.
- [ ] The Linux CI lane runs lint, the example, the double-run hash check, and the golden-trace comparison.
- [ ] Results are recorded in [Evidence](../evidence.md) under E-2, E-3, E-4, and E-5 with the commit, platform, command, and artifact.
- [ ] No document claims the slice works until the evidence rows contain a pass result.

### R7 — Documentation and status

- [ ] The README quick start is updated only after the commands exist and pass.
- [ ] The roadmap marks the documentation baseline and the vertical-slice items according to actual evidence.
- [ ] Any contract change discovered during implementation is recorded in an ADR in the same change.

## Edge Cases

| Case | Expected behavior |
| --- | --- |
| A package manifest is missing or malformed. | Lint reports the file and a stable error code; the run does not start. |
| A component port referenced by a net does not exist. | Validation error naming the instance, port, and net. |
| A board pin lacks the required ADC or pull-up capability. | Validation error naming the pin, capability, and net. |
| Two digital outputs drive the same net. | Validation error; the simulator never resolves it by last-writer-wins. |
| The lockfile is missing or does not match a package hash. | Validation error with a resolution hint; no implicit re-resolution. |
| The project seed is missing. | Validation error; the example ships an explicit seed. |
| The duration is zero or negative. | Validation error; a run must have a positive virtual-time bound. |
| The event limit is reached before the duration. | Typed fault, `fault` event, then `run_end`; exit code `2`. |
| The button is pressed at the same microsecond as an LED toggle. | The stable `(time, insertion_sequence)` order decides; the golden trace records it. |
| The trace path is not writable. | Runtime error before simulation starts; no partial trace is reported as complete. |
| The firmware references an alias that the project does not define. | Validation error before the session starts. |

## Constraints

- **Offline:** lint and run must work without a network connection.
- **Single process:** the simulator runs in-process; no external emulator is required.
- **Single-threaded simulation:** the first slice does not introduce nondeterministic thread scheduling.
- **No floating point in trace identity:** analog values are quantized before they reach the trace.
- **Linux first:** the first CI evidence runs on Linux. A cross-platform claim needs a second platform in CI.
- **No physical hardware:** this slice proves host simulation only. A passing result does not prove hardware behavior.
- **No UI:** the CLI is the only interface. The desktop app starts in v0.2.
- **Versioned contracts:** project, package, protocol, and trace schemas carry explicit versions.

## Success

| Signal | Target |
| --- | --- |
| Example project validates | `playground-cli lint projects/examples/blink` exits `0` on Linux CI. |
| Example project runs | `playground-cli run projects/examples/blink` completes 2,000,000 µs of virtual time and writes a trace. |
| Button changes behavior | The scripted press at 750,000 µs stops LED changes; release at 800,000 µs resumes them. |
| Deterministic trace | Two runs produce the same SHA-256 hash on the same platform. |
| Golden trace | The committed golden trace matches the current run. |
| Evidence recorded | E-2 through E-5 contain pass results with commit, platform, command, and artifact. |
| Seam review | A maintainer can point to the project, catalog, net, firmware, and trace boundaries in the diff. |

## Open Questions

- [ ] Is `mce.reference-board` the right first board abstraction, or should the slice model a specific board such as the ESP32-C3 SuperMini?
- [ ] Is Rust with `embedded-hal` 1.0 the confirmed first firmware path? The product definition still lists this as an unvalidated assumption.
- [ ] Should the example project file use TOML or JSON? The schema is the contract; the first encoding is still open.
- [ ] Is a 1 µs tick sufficient for the first component models, or does the scheduler need sub-microsecond resolution?
- [ ] Should the button input fixture be a separate file referenced by the project, or an inline section of the project file?
- [ ] Is the trace schema stable enough for a committed golden file, or should the first golden test compare a normalized event list?
