---
id: MCE-DOCS-001
title: "Micro Controller Playground Documentation Index"
status: proposed
---

# Micro Controller Playground Documentation Index

This index is the entry point for governed documentation. Each document owns one topic; other documents link to it instead of restating it.

## Documents

| Document | ID | What it owns |
| --- | --- | --- |
| [Product](./product.md) | MCE-PRODUCT-001 | Problem, users, goals, scope, principles |
| [Roadmap](./roadmap.md) | MCE-ROADMAP-001 | Milestones, outcomes, and sequencing |
| [Architecture](./architecture.md) | MCE-ARCH-001 | System structure, boundaries, interfaces, and flows |
| [Evidence](./evidence.md) | MCE-EVID-001 | Pass/fail checks and recorded results |
| [Documentation standards](./standards.md) | MCE-STD-001 | Rules for writing and changing docs |
| [ADR index](./adr/README.md) | MCE-ADR-0000 | Decision records and their status |
| [ADR-0001](./adr/0001-device-backends-behind-one-protocol.md) | MCE-ADR-0001 | Device backend boundary |
| [ADR-0002](./adr/0002-runtime-package-catalog.md) | MCE-ADR-0002 | Built-in and project-local package catalog |
| [ADR-0003](./adr/0003-logical-nets-and-board-binding.md) | MCE-ADR-0003 | Logical nets and board pin binding |
| [ADR-0004](./adr/0004-component-model-driver-split.md) | MCE-ADR-0004 | Component model/driver split and WASM plugins |
| [ADR-0005](./adr/0005-versioned-device-protocol.md) | MCE-ADR-0005 | One versioned device protocol |
| [ADR-0006](./adr/0006-deterministic-simulation-and-evidence.md) | MCE-ADR-0006 | Deterministic simulation and evidence tiers |
| [MVP PRD](./specs/0001-mvp-blink.md) | MCE-PRD-001 | First headless vertical slice |

## How to read this set

1. Start with [Product](./product.md) for the problem and boundaries.
2. Read [Roadmap](./roadmap.md) for the order of work.
3. Read [Architecture](./architecture.md) for the intended structure.
4. Read the ADRs for the decisions that are expensive to reverse.
5. Read the [MVP PRD](./specs/0001-mvp-blink.md) before implementing the first slice.
6. Record results in [Evidence](./evidence.md); do not claim behavior without a linked result.

## Governance

Every governed document follows [Documentation standards](./standards.md): YAML header, permanent ID, one owner per topic, evidence-only pass/fail claims, and an ADR for contract changes. To add a document, copy a template, choose a new ID, and add a row above.
