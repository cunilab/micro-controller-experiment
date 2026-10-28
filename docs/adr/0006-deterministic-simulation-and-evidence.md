---
id: MCE-ADR-0006
title: "ADR-0006: Deterministic simulation and evidence tiers"
status: proposed
---

# ADR-0006: Deterministic simulation and evidence tiers

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [Evidence](../evidence.md), [ADR-0001](./0001-device-backends-behind-one-protocol.md)

## Context

A simulator is only useful for debugging if a failure can be reproduced. Wall-clock time, unordered iteration, floating-point drift, and thread scheduling make runs differ for reasons unrelated to the project. The same problem appears in evidence: a host-simulation pass can be mistaken for proof that hardware works.

The product needs a definition of determinism that component authors can follow and a vocabulary that keeps different kinds of proof separate.

## Decision

### Deterministic simulation

- The simulation clock is integer **microseconds**. Scheduling, timeouts, and trace timestamps use this clock. The core does not read wall-clock time.
- Events are ordered by `(time, insertion_sequence)`. The insertion sequence is stable and part of the simulation contract.
- Randomness comes from an explicit, seeded, documented generator. The seed is stored in the project and recorded in the trace. The core does not use thread-local or OS randomness.
- Iteration order is stable: ordered maps or sorted vectors, never hash-map order, in any path that affects state or trace output.
- Analog values are quantized at the boundary (for example, ADC counts or integer microvolts). Floating-point math is not used for scheduling, ordering, or trace identity.
- Inputs (button presses, bus responses, external stimuli) come from a scripted fixture with timestamps. Interactive input is a separate feature and is recorded as an input event before it affects the run.
- A run has a virtual-time limit and an event limit. Exceeding either produces a typed fault, not an unbounded loop.
- The trace is a versioned, canonical sequence of events. Its identity is a SHA-256 hash of the canonical bytes. Human-readable rendering is a view; it does not change the identity.
- Trace identity covers the app build, project schema version, package versions and content hashes, firmware ID, protocol version, trace schema version, seed, and input fixture hash. A change to any of them is expected to change the hash.
- Determinism is claimed for the same app build, lockfile, seed, input fixture, and platform. A cross-platform claim requires the same hash on at least two platforms in CI.

### Evidence tiers

Every check in [`evidence.md`](../evidence.md) names exactly one tier:

| Tier | What it proves | What it does not prove |
| --- | --- | --- |
| **Unit** | A function or package behaves as specified in isolation. | Integration, timing, or hardware behavior. |
| **Host simulation** | The project runs in the native simulator with the recorded trace. | Emulator behavior or physical behavior. |
| **Emulator** | The firmware runs under an emulator backend and matches the expected observations. | Physical electrical behavior or component models outside the emulator. |
| **Physical** | A real board runs the firmware and the observed behavior matches the recorded expectation. | Electrical equivalence beyond the observed signals, or other boards. |

A lower tier never closes a higher-tier check. A physical check is behavioral evidence, not a claim that simulation is electrically accurate.

## Consequences

- Component authors give up convenience features that break determinism: wall-clock delays, unordered maps in state paths, and unseeded randomness.
- Floating-point sensor models must quantize at the boundary and document the quantization. This can lose precision.
- Traces grow with event count. The event limit and a bounded ring buffer keep memory use predictable; a full trace may be written to disk.
- Trace hashes are sensitive to schema and package versions. That is intentional, but it means golden traces must be regenerated when a versioned input changes.
- Evidence records need more metadata (tier, commit, platform, command, artifact). The discipline is the point.
- Cross-platform determinism is a testable claim, not an assumption. It starts as a Linux-only check and becomes a two-platform CI check in v0.4.
- The simulator cannot model every physical effect. The documentation must keep saying so.

## Alternatives considered

**Use wall-clock time and accept nondeterminism.** Rejected: it makes failures unreproducible and golden traces impossible.

**Use floating-point time and values throughout.** Rejected: rounding and platform differences break trace identity and event ordering.

**Hash the rendered text trace.** Rejected: formatting changes would invalidate traces for no behavioral reason. The hash is over canonical bytes.

**Treat any passing test as evidence for all tiers.** Rejected: it hides the difference between a host test and a physical run, which is exactly the overclaim the product must avoid.

**Record evidence in commit messages or PR descriptions only.** Rejected: it is not discoverable and disappears from the documentation set. Evidence has one owner document.
