---
id: MCE-ADR-0000
title: "Architecture Decision Records"
status: proposed
---

# Architecture Decision Records

An ADR records a decision that is expensive, important, or likely to be questioned later. It explains the forces and the choice, not every detail of the resulting code.

## Rules

- One decision per record.
- Numbers are sequential and never reused. A replaced decision is marked `superseded` and linked to its replacement.
- `status` describes the text: `draft`, `proposed`, `accepted`, or `superseded`.
- A contract change lands with its ADR in the same pull request.
- Accepted ADRs are not edited to match new behavior. Add a new ADR and link the old one.

## Index

| ADR | Title | Status |
| --- | --- | --- |
| [0001](./0001-device-backends-behind-one-protocol.md) | Device backends behind one protocol | proposed |
| [0002](./0002-runtime-package-catalog.md) | Built-in and project-local package catalog | proposed |
| [0003](./0003-logical-nets-and-board-binding.md) | Logical nets and board pin binding | proposed |
| [0004](./0004-component-model-driver-split.md) | Component model/driver split and WASM plugins | proposed |
| [0005](./0005-versioned-device-protocol.md) | One versioned device protocol | proposed |
| [0006](./0006-deterministic-simulation-and-evidence.md) | Deterministic simulation and evidence tiers | proposed |

## Planned but not written

Write these only when the decision is actually needed:

- Project schema versioning and migration policy.
- Firmware language and toolchain boundary.
- Artifact and package signing.
- Host/device transport (USB serial, WebSocket bridge, or another).
