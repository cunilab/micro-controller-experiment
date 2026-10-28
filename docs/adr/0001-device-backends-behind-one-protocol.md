---
id: MCE-ADR-0001
title: "ADR-0001: Device backends behind one protocol"
status: proposed
---

# ADR-0001: Device backends behind one protocol

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [ADR-0005](./0005-versioned-device-protocol.md), [ADR-0006](./0006-deterministic-simulation-and-evidence.md)

## Context

The product must run a project in a native simulator and, later, on a real board. It may also gain an emulator backend. If each backend exposes its own API, the UI, project format, and session lifecycle grow backend-specific branches. Simulator behavior then drifts from hardware behavior, and every new backend repeats the integration work.

The alternative is to make the backend boundary the main architectural seam. That requires a contract that covers more than pin toggles: session lifecycle, identity, capabilities, pin and bus events, serial output, reset, and faults.

## Decision

All device execution goes through one backend contract, defined in `playground-protocol` and consumed by the host session manager.

- The backend kinds are **native simulator** (first), **physical board over USB** (v0.3), and **emulator adapter** (optional, later). They are interchangeable from the host's point of view.
- The host selects a backend from the project's run target. It does not expose backend-specific operations to the UI.
- The UI consumes typed session state and trace events. It may display the backend identity and backend-specific diagnostics, but the editing and run workflow is identical.
- A backend declares **capabilities** (for example, GPIO, ADC, PWM, I2C, SPI, UART, trace, reset). The host negotiates capabilities at session start instead of assuming them from a board or backend name.
- A backend that cannot provide a requested capability fails the session with a typed error. It does not silently degrade behavior.
- The in-process simulator may dispatch typed messages directly rather than serialize them, but it implements the same message semantics and passes the same contract tests as a remote backend.
- Backend-specific state stays behind the boundary. The project file and component/board packages never record which backend is active.

The wire-level details of this contract are owned by [ADR-0005](./0005-versioned-device-protocol.md). Determinism requirements are owned by [ADR-0006](./0006-deterministic-simulation-and-evidence.md).

## Consequences

- The protocol must be designed before the first backend is useful, which front-loads work.
- The contract is the lowest common denominator unless capabilities are used carefully. A feature that only one backend can support must be capability-gated.
- The simulator pays a small abstraction cost even when running in-process.
- Backend-specific diagnostics need an explicit, typed escape hatch; otherwise they leak into the UI.
- Every backend needs the same contract tests. A passing simulator test does not prove the physical backend.
- Physical faults, disconnects, and reconnects must be represented in the shared session model rather than handled as UI special cases.

## Alternatives considered

**Separate simulator and hardware APIs.** Rejected: two integration paths, two sets of UI branches, and a high risk of behavioral drift.

**Simulator only, with flashing as an external step.** Rejected: the product's central promise is moving the same project between simulation and hardware. Leaving hardware outside the session model breaks that promise and the observation workflow.

**Run everything through an emulator.** Rejected for the first slice: an emulator does not model the board's components or the project's nets, and it adds a large external dependency before the contracts are proven. An emulator remains a later backend behind the same protocol.

**Make the UI backend-aware.** Rejected: it moves compatibility logic into the least testable layer and multiplies work for every new backend.
