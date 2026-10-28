# Micro Controller Playground

> A desktop workbench for assembling, simulating, and running microcontroller projects against virtual or real boards.

**Status:** `Design / pre-implementation` · **Version:** `v0.1 (planned)`

## Overview

Micro Controller Playground is a project-based workbench for makers and firmware developers. A project selects a board, adds components, connects them through logical nets, and runs firmware against a deterministic native simulator or a real board over USB. The host shows pin state, bus traffic, serial output, and a logic-analyzer timeline.

The central rule is that simulation and hardware are two backends behind the same device contract. The UI, project format, board packages, and component packages do not know which backend is active.

## Planned features

- **Project-based workbench** — board, components, nets, firmware, and simulation settings in one reproducible project.
- **Package catalog** — boards and components are versioned packages with manifests, models, optional firmware drivers, and UI assets.
- **Same project, two backends** — run in the native simulator first, then flash the same project to a real board.
- **Inspectable execution** — pin state, bus traffic, serial output, traces, pause/step, and snapshots.
- **Honest evidence** — simulated results and hardware results are recorded separately.

## Quick start

There is no runnable build yet. The first vertical slice is specified in [docs/specs/0001-mvp-blink.md](docs/specs/0001-mvp-blink.md).

Planned developer entry points, once implemented:

```sh
# Run the headless simulator against an example project
cargo run -p playground-cli -- run projects/examples/blink

# Validate the catalog and project files
cargo run -p playground-cli -- lint
```

## Documentation

| Document | Purpose |
| --- | --- |
| [Documentation index](docs/README.md) | Entry point to all governed docs |
| [Product](docs/product.md) | Problem, users, goals, scope, principles |
| [Roadmap](docs/roadmap.md) | Milestones and outcomes |
| [Architecture](docs/architecture.md) | System structure, boundaries, and flows |
| [Evidence](docs/evidence.md) | Pass/fail checks and recorded results |
| [Documentation standards](docs/standards.md) | Rules for changing docs |
| [ADRs](docs/adr/README.md) | Important decisions and their reasoning |
| [MVP PRD](docs/specs/0001-mvp-blink.md) | First vertical slice |

## Project status

**Current:** architecture and documentation baseline; no implementation yet.  
**Next:** headless vertical slice — one board, LED/button/potentiometer, native simulation, deterministic trace.

## Contributing

The project is pre-implementation. Open an issue or comment on the docs PR before starting a large change. Contract changes (project schema, package schema, protocol, component/board API) need an ADR in the same change.

## License

Not selected yet. Do not assume reuse rights until a license is added.
