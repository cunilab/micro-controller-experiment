---
id: MCE-ADR-0003
title: "ADR-0003: Logical nets and board pin binding"
status: proposed
---

# ADR-0003: Logical nets and board pin binding

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [ADR-0002](./0002-runtime-package-catalog.md), [ADR-0004](./0004-component-model-driver-split.md)

## Context

A component such as an LED has an anode and a cathode. A board has GPIO pins with different capabilities. If the project stores "LED anode on GPIO4" directly on the component, then changing the board forces every component to be edited, and the firmware's pin numbers are baked into the project.

The product needs a representation that survives a board swap and still gives the simulator and firmware an unambiguous electrical connection.

## Decision

Projects store **logical nets**, and a separate **board binding** connects those nets to board pins.

- A component instance exposes named **ports** (for example, `anode`, `cathode`, `sda`, `vcc`). Ports are declared by the component package.
- A **net** is a named connectivity group with a list of member ports. Nets are the only place where component ports are joined.
- The board binding maps a **board pin** to a net. A net has at most one board-pin binding unless the net is explicitly declared as a bus with the required roles.
- A net may be declared as a digital signal, an ADC channel, a PWM channel, or a bus (I2C, SPI, UART). The declaration determines how the simulator resolves it and which capability the board pin must provide.
- Digital nets resolve to one of `low`, `high`, `high-z`, or `conflict`. The validator rejects a statically known multi-driver net; if a package produces a conflict at runtime, the simulator reports it as a typed fault, never as a last-writer-wins result.
- The project also declares **pin aliases** (for example, `status_led`, `user_button`, `blink_rate`) that map to nets. Firmware refers to aliases, not to board pin numbers.
- The board package declares pin capabilities (GPIO, ADC channels, PWM channels, bus roles, voltage). The validator checks every binding against those capabilities and rejects a mismatch.
- A board swap keeps the logical nets and aliases. The host proposes a new binding by alias and capability, and the user confirms it. Unresolvable aliases are reported, not silently dropped.
- The project schema stores board pin identifiers as stable package-defined strings, not as raw MCU register numbers.

## Consequences

- The project has one more layer of indirection. A simple blink project needs nets and aliases even though a direct pin number would be shorter.
- Board swaps and component reuse become possible without rewriting component instances.
- Validation must understand capabilities, bus roles, and driver conflicts. This is more work than checking that a pin number exists.
- The UI needs a net-oriented view as well as a board view. Users must be able to see which net a pin belongs to.
- Firmware drivers become board-independent when they consume aliases and `embedded-hal`-style ports instead of fixed pin numbers.
- A net is a logical connectivity model, not a claim of electrical equivalence. Analog and high-frequency behavior remain out of scope.

## Alternatives considered

**Store board pin numbers directly on component instances.** Rejected: a board change rewrites every component, and the same component cannot be reused across boards without edits.

**Store firmware pin numbers in the project only.** Rejected: the simulator and the UI would have no component connectivity model, and a board swap would still require manual edits.

**Treat every connection as a bus.** Rejected: it makes simple GPIO circuits unnecessarily complex and hides driver conflicts that a net model can detect.

**Bind components directly to MCU pins and skip nets.** Rejected: it cannot express a shared net, a button to ground, or a component with multiple ports, and it couples the component package to a specific board.
