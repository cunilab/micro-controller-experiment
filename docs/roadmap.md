---
id: MCE-ROADMAP-001
title: "Roadmap — Micro Controller Playground"
status: proposed
---

# Roadmap — Micro Controller Playground

The roadmap optimizes for proving the seams early: one board, a few components, one firmware path, and an honest simulation boundary. Catalog breadth comes after the contracts are stable.

**Current focus:** documentation baseline and the first headless vertical slice.

Status vocabulary: ✅ shipped, 🚧 active, no icon = planned.

### 🚧 v0.1 — Headless foundation

> **Outcome:** a project can be loaded, validated, simulated, and inspected from a CLI without any UI.

- [x] **Documentation baseline** — product, architecture, ADRs, MVP spec, evidence skeleton (this PR).
- [ ] **Project and package schemas**
  - [ ] Project file with board reference, parts, logical nets, firmware reference, and simulation settings.
  - [ ] Board and component manifests with versions and lockfile.
  - [ ] Validation for duplicate IDs, missing pins, pin capability mismatch, and invalid bus configuration.
- [ ] **Native simulation core**
  - [ ] Event-driven scheduler with integer time and seeded randomness.
  - [ ] GPIO input/output, ADC, and PWM primitives.
  - [ ] LED, push-button, and potentiometer models.
- [ ] **Firmware path**
  - [ ] `no_std` firmware core with `embedded-hal`-style ports.
  - [ ] Host-sim adapters for the first board.
  - [ ] One example firmware that blinks an LED and reads a button.
- [ ] **Headless CLI**
  - [ ] `playground-cli run <project>` runs for a bounded time and prints a trace.
  - [ ] `playground-cli lint` validates the catalog and project.
  - [ ] A golden trace test proves deterministic output.

**Exit:** `blink` runs headless, the button changes the trace, and two runs produce the same hash.

### v0.2 — Workbench UI

> **Outcome:** a person can create, open, edit, run, pause, and step a project in the desktop app.

- [ ] Tauri shell with typed IPC and a React UI.
- [ ] Project tree, board/component palette, canvas, inspector, and validation panel.
- [ ] Run, pause, step, reset, speed control, and serial monitor.
- [ ] Save/load, autosave, undo/redo, and project-relative paths.
- [ ] Manifest-driven part graphics with safe SVG pin anchors.

**Exit:** the example project can be created from the UI and run without the CLI.

### v0.3 — Hardware path

> **Outcome:** the same project that runs in the simulator can be built, flashed, and observed on a real board.

- [ ] Compiler adapter for the chosen firmware stack.
- [ ] Flasher adapter with tool detection and pinned sidecar support.
- [ ] Physical backend behind the same protocol.
- [ ] Board and firmware image identity guard before flashing.
- [ ] Connection status, reconnect, heartbeat, and safe failure messages.

**Exit:** the example project runs on the real board and produces a comparable trace.

### v0.4 — Buses and a useful catalog

> **Outcome:** a project can use common I2C/SPI/UART parts and inspect their traffic.

- [ ] I2C, SPI, and UART bus models with conflict validation.
- [ ] Five to six built-in components, including a display or sensor.
- [ ] Logic-analyzer view fed by trace events.
- [ ] Component contract tests: model behavior, driver transactions, and end-to-end.
- [ ] Golden traces in CI on two operating systems.

**Exit:** an I2C display or sensor project works in simulation and on hardware.

### v0.5 — Modularity proof

> **Outcome:** the package boundary is proven with a second board and a second component pack.

- [ ] A second board package with a different pin map and MCU family or variant.
- [ ] A second component pack with at least two parts not present in the first pack.
- [ ] No changes to `playground-core` or the UI required to register either.
- [ ] Board swap/remap flow with validation and suggested compatible pins.
- [ ] Catalog and project migration tests.

**Exit:** adding a board or part is a package-only change, demonstrated in CI.

### v1.0 — Extensibility

> **Outcome:** external authors can add safe components, and an optional emulator backend can run unmodified firmware.

- [ ] WASM component SDK with a stable ABI and sandbox limits.
- [ ] Optional Renode or QEMU backend behind `DeviceBackend`.
- [ ] Signed package format and compatibility metadata.
- [ ] Import/export for a common diagram format.
- [ ] Packaging and update path for the desktop app.

**Exit:** a user-authored component runs without rebuilding the app, and the same project can use an emulator backend.

## Later

- A public package registry or marketplace.
- Multi-board projects and board-to-board buses.
- Cloud sync or collaboration.
- Additional firmware languages.
- Automated hardware-in-the-loop test rigs.
- PCB/netlist export.

## Not Planned

- Full SPICE or general-purpose analog simulation.
- A custom CPU emulator.
- Arbitrary native plugins.
- Every board and every language in the first release.
- Claims that simulation replaces physical validation.

## Sequencing Rule

Do not start the next milestone while the previous milestone's exit criterion is unproven. Hardware rows and host-sim rows are separate evidence; a passing host test does not close a hardware check.
