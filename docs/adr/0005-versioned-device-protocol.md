---
id: MCE-ADR-0005
title: "ADR-0005: One versioned device protocol"
status: proposed
---

# ADR-0005: One versioned device protocol

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [ADR-0001](./0001-device-backends-behind-one-protocol.md), [ADR-0006](./0006-deterministic-simulation-and-evidence.md)

## Context

The host talks to a simulator in the same process and, later, to a physical board over a serial link. Those paths have different transports and failure modes. If their message semantics diverge, the simulator stops being a useful preview of hardware, and the UI needs two behavior models.

The protocol also has to survive version skew: an older desktop app may meet newer firmware, or a newer app may meet an older board. Both sides need to fail clearly instead of misinterpreting messages.

## Decision

`playground-protocol` defines one versioned, schema-first device protocol. Every backend implements it.

- The protocol is `no_std`-compatible and has no host, UI, serial, or filesystem dependencies.
- Messages are grouped into session/control, pin, bus, serial, trace, fault, and heartbeat families.
- The wire frame is a fixed header followed by a payload and a checksum. The header carries a magic value, protocol major and minor versions, message kind, flags, sequence number, and payload length. The exact byte layout, checksum algorithm, and payload codec are defined by the crate and locked with golden vectors before the physical backend ships.
- A session starts with a handshake:
  - host `Hello` includes the protocol major/minor range, a host identifier, and a session nonce;
  - device `Welcome` includes the selected protocol major/minor, backend kind, board ID, firmware ID, capability set, and maximum frame size.
- A **major** mismatch is rejected with a typed error and no session. A **minor** mismatch negotiates down to the lower minor. Optional behavior is gated by capability flags, never inferred from a version number alone.
- Every frame belongs to a session. Frames with an unknown session, an invalid checksum, or an invalid length are dropped and counted; they must not dispatch side effects.
- Sequence numbers are tracked per direction. A gap is reported as a fault; the receiver does not silently fill it.
- `Reset` is explicit and idempotent. Reconnect starts a new session; the first release does not resume state across a disconnect.
- Heartbeats carry a session identity and a monotonically increasing counter. A missed heartbeat budget produces a typed timeout fault.
- The in-process simulator may bypass byte serialization, but it performs the same handshake, capability negotiation, session checks, and message semantics, and it passes the same contract tests.
- The protocol version is independent of the application, project schema, package schema, component API, and firmware versions.

## Consequences

- The protocol must be designed and tested before a physical backend exists. That is deliberate: the simulator is the first implementation of the contract.
- Golden vectors and a compatibility matrix become part of the test suite. Changing a message requires a versioning decision.
- Capability negotiation adds a step to every session and a failure mode that must be explained to users.
- The in-process path can be faster, but it must not become a semantic shortcut. Shared contract tests are the guard.
- A physical link adds framing, checksums, timeouts, and reconnection behavior that the simulator does not exercise. The physical backend needs its own integration tests.
- Some device-specific diagnostics will not fit the shared vocabulary. They must use the fault message or a capability-gated extension, not an out-of-band channel.

## Alternatives considered

**Let each backend define its own messages.** Rejected: it recreates the drift problem at the protocol layer and forces the host to maintain multiple parsers.

**Use an existing generic RPC protocol.** Rejected as the primary contract: generic RPC does not define pin, bus, trace, or capability semantics, and it brings a large dependency into `no_std` firmware. A transport may still use a standard framing layer internally.

**Version only the transport, not the messages.** Rejected: message shape changes are the compatibility risk. The schema and its semantics must be versioned together.

**Negotiate by trying messages until one works.** Rejected: it hides incompatibility and makes failures nondeterministic. Explicit version and capability negotiation fails fast.
