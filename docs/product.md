---
id: MCE-PRODUCT-001
title: "Product — Micro Controller Playground"
status: proposed
---

# Product — Micro Controller Playground

> A project-based workbench that lets a developer test a microcontroller project in simulation first, then run the same project on real hardware.

This is the stable product definition. Feature-level behavior belongs in [`specs/`](./specs/); implementation evidence belongs in [`evidence.md`](./evidence.md).

## Product Statement

**For** makers and firmware developers  
**who** need to try a board, component, and firmware combination without rebuilding a physical setup for every change,  
**Micro Controller Playground** is a desktop workbench  
**that** turns a project into a repeatable simulation or hardware run, with the same board and component definitions on both paths.

The important difference is not a larger parts library. It is that **simulation and hardware share one device contract**, so a project does not have to be rewritten when it moves from the simulator to a real board.

## Problem

**Current situation:** a developer wires a board, flashes firmware, opens a serial monitor, and repeats. Simulators are separate tools with their own project format, parts, and code assumptions. Changing a board or a component often means changing pin numbers, drivers, and sometimes the project itself.

**Problem:** hardware iteration is slow and hard to reproduce; simulation tools are difficult to extend with local parts; and simulator behavior can drift from the firmware used on the board.

**Impact:** more time debugging wiring and tooling, less time testing the idea; experiments are difficult to share or repeat; board and component changes create rework.

## Users

| User | Need | Current alternative |
| --- | --- | --- |
| **Primary: maker / firmware developer** | Try a board + component + firmware combination quickly and move it to hardware without a rewrite. | Breadboard, vendor IDE, serial monitor, separate simulator. |
| **Secondary: educator / reviewer** | Explain or review a microcontroller setup reproducibly. | Screenshots, videos, ad-hoc wiring instructions. |
| **Secondary: component/board author** | Add a new part or board without changing the workbench core. | Fork the simulator, add a one-off part. |

## Goals

- [ ] **One project, two backends** — the same project runs in the native simulator and on a real board.
- [ ] **Package-based change** — adding or updating a board or component does not require changes to the core or UI.
- [ ] **Inspectable execution** — pin state, bus traffic, serial output, and a trace are visible and reproducible.
- [ ] **Honest evidence** — simulated, emulated, and physical results are recorded separately and never conflated.
- [ ] **Local-first** — the core works offline; project files and packages are on disk.

## Scope

### In

- Project files with a board, component instances, logical nets, firmware reference, and simulation settings.
- A catalog of versioned board and component packages with validation.
- A native simulator with deterministic time, GPIO, buses, and a trace.
- A firmware path for a supported language/framework, starting with the project's chosen stack.
- A desktop UI for editing, running, pausing, stepping, and inspecting a project.
- A physical path for a supported board: compile, flash, connect, and observe.
- Evidence and validation separate from plans.

### Out

- Full SPICE or general analog circuit simulation.
- Every board, every MCU family, and every firmware language.
- A cloud account, collaboration service, or public marketplace.
- Arbitrary native plugins loaded at runtime.
- A custom CPU emulator written from scratch.
- Mobile applications.
- Production-grade secure boot, provisioning, or fleet management.
- Claims that a simulation proves physical behavior.

## Product Principles

- **Boundaries before catalog.** A stable device, package, and net contract matters more than the number of parts.
- **Same contract, different backend.** The UI and project format do not branch on simulator versus hardware.
- **Data describes, code behaves.** Manifests describe pins, names, parameters, and recipes. Behavior lives in a typed model or driver.
- **Capabilities, not IDs.** Features are selected from declared capabilities, never from a board or component name.
- **Deterministic by default.** Integer time, seeded randomness, stable ordering, and traces make failures reproducible.
- **Evidence has tiers.** Unit, host-simulation, emulator, and physical results are different classes of proof.
- **Offline first.** A project must open and run without a network account.
- **Small vertical slices.** Prove the seams with one board, a few parts, and one firmware path before expanding.

## Assumptions

Keep only assumptions that could change the product if proven wrong.

| Assumption | Evidence | Status |
| --- | --- | --- |
| The primary firmware path can be Rust with `embedded-hal`-style drivers. | None yet | Unvalidated |
| A native simulator is sufficient for the first vertical slice; arbitrary firmware emulation can wait. | None yet | Unvalidated |
| One board and three components are enough to prove the package and net seams. | None yet | Unvalidated |
| Users will accept project files and packages on disk rather than a hosted project service. | None yet | Unvalidated |
| A desktop app is the right host because it needs serial, filesystem, compiler, and flasher access. | None yet | Unvalidated |

## Success

| Signal | Current | Target |
| --- | --- | --- |
| A second board and a second component pack can be added without changing `playground-core` or the UI. | Not possible yet | Demonstrated in the modularity milestone |
| The same project runs in the native simulator and on a real board. | Not possible yet | Demonstrated in the hardware milestone |
| A failing simulation can be reproduced from a saved trace. | Not possible yet | Golden-trace test passes on two platforms |
| A new user can run the example project and see the expected trace. | Not possible yet | Documented in the MVP PRD and verified |

## Open Questions

- Which board is the first supported target: ESP32-C3 SuperMini, RP2040, or another board?
- Which firmware language is first: Rust, Arduino/C++, or MicroPython?
- Is arbitrary-firmware emulation a day-one requirement, or can the native simulator come first?
- Are user-authored components required in the first release, or only built-in packages?
- Which license applies to the code and to package assets?
