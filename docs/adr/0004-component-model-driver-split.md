---
id: MCE-ADR-0004
title: "ADR-0004: Component model/driver split and WASM plugins"
status: proposed
---

# ADR-0004: Component model/driver split and WASM plugins

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [ADR-0002](./0002-runtime-package-catalog.md), [ADR-0003](./0003-logical-nets-and-board-binding.md)

## Context

A component has two kinds of behavior. In simulation, it must react to pin and bus changes with timing. In firmware, it must issue the transactions that a real device expects. Those behaviors run in different environments: the simulator is a host process, while firmware is `no_std` and may run on a microcontroller.

Putting both behaviors in one source file leads to conditional compilation, host dependencies leaking into firmware, and simulation-only shortcuts that do not match the driver. Putting behavior only in the manifest is not enough for timing, state, and bus protocols.

The product also needs a path for user-authored components without loading arbitrary native code into the desktop process.

## Decision

Every component package separates **data**, **simulation behavior**, and **firmware behavior**.

- **Manifest** — identity, version, ports, parameters, UI assets, and compatibility. The manifest is the single source of truth for names and connectivity. It contains no executable behavior.
- **Model** — the simulation-side implementation. It observes and drives ports, advances with simulation time, and emits trace-relevant state. Models are deterministic and must not read wall-clock time, the filesystem, or the network.
- **Driver** — the firmware-side implementation. It is an optional Rust crate that uses `embedded-hal` 1.0 traits and is wired to the component's logical ports by the board/project binding. It contains no host dependencies.
- **Contract tests** — every package ships tests for manifest validity, model behavior, driver transactions, and at least one end-to-end project using both.
- Model and driver may share pure data types and constants, but neither may depend on the other's runtime.
- First-party and project-local trusted packages are compiled into the application and firmware build. The manifest and API versions are checked at catalog load time.
- **User-authored executable behavior is WASM simulation models only.** WASM components run in a sandbox with no filesystem, network, or ambient host access by default. They cannot provide firmware drivers in the first release, because a driver must be compiled into the firmware image and is not sandboxed.
- A package declares which API version it implements. The host rejects a package whose API major version does not match.

## Consequences

- A component author writes and tests two implementations. This is real duplication, mitigated by a shared manifest, shared test fixtures, and contract tests.
- Model and driver versions can drift. The package version and lockfile make the combination explicit, and contract tests must cover the pair.
- WASM components are portable and safer, but slower and limited to the capabilities the sandbox exposes. A model that needs a host feature must request it explicitly.
- Firmware drivers remain native code, so installing an untrusted driver still requires a trusted build. The first release does not promise safe execution of arbitrary drivers.
- The manifest becomes a reviewed contract. Changing a port or parameter is a package version change, not a silent edit.
- Component packages cannot import core internals or each other. Shared behavior belongs in a small, versioned support crate.

## Alternatives considered

**One implementation used for simulation and firmware.** Rejected: the environments differ enough that conditional compilation spreads through the code, and simulation-only dependencies leak into firmware.

**Manifest-only components with a scripting language.** Rejected: it cannot express timing, state machines, or bus transactions well, and it creates a new language runtime inside the simulator.

**Native dynamic libraries for user components.** Rejected: unsafe, platform-specific, and impossible to sandbox consistently.

**WASM for both models and firmware drivers.** Rejected for the first release: firmware drivers must run on the target MCU and cannot depend on a WASM runtime. This can be revisited if a target runtime makes it practical.

**Let component packages depend on core internals.** Rejected: it turns every internal refactor into a package break and destroys the package boundary.
