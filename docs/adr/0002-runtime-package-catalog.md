---
id: MCE-ADR-0002
title: "ADR-0002: Built-in and project-local package catalog"
status: proposed
---

# ADR-0002: Built-in and project-local package catalog

- **Date:** 2026-10-28
- **Milestone:** v0.1
- **Related:** [Architecture](../architecture.md), [ADR-0004](./0004-component-model-driver-split.md)

## Context

Boards and components will change frequently, and the project must not require a core change for each one. They also need to be reproducible: opening an old project should not silently pick up a newer part definition.

A package catalog is the boundary between "data on disk" and "typed objects the host can run". It must answer four questions: what packages exist, which version a project uses, whether a package is valid, and where its code and assets live.

## Decision

Boards and components are **packages** discovered from one or more catalog roots at runtime. The host does not hardcode a list of supported parts.

- A package is a directory with a manifest, optional model code, optional firmware driver, and optional UI assets.
- The catalog roots are:
  1. **Project-local** — `packages/` inside the project directory. Used for development and for a project that needs a pinned private part.
  2. **User** — an optional per-user data directory. Used for parts shared across projects.
  3. **Bundled** — packages shipped read-only with the application. The first-party catalog lives here.
- Resolution order is project-local, then user, then bundled. A project lockfile pins the exact package version and content hash, so resolution is reproducible even if a root changes later.
- Two packages with the same ID and version are allowed across roots only when their content hashes match. A conflict is a validation error; the host does not guess.
- Manifests declare a package schema version and an API version. The catalog rejects a package built for an incompatible API.
- The catalog validates manifests before registration: unique IDs, valid pin and bus references, required assets, and well-formed parameters.
- Built-in packages are compiled into the application. Project-local packages may contain data and, for trusted packages, Rust model/driver crates compiled into the application build. Arbitrary native libraries are never loaded at runtime.
- User-authored executable behavior is limited to sandboxed WASM simulation models, as decided in [ADR-0004](./0004-component-model-driver-split.md).
- The host exposes catalog listings and validation results to the UI as typed data. The UI does not read package directories itself.

## Consequences

- Projects remain reproducible without a network service, at the cost of a lockfile that must be maintained.
- Catalog scans must be deterministic: roots are searched in a fixed order, results are sorted by ID and version, and duplicate IDs produce stable errors.
- Users can develop a package inside a project without publishing it. They can also accidentally shadow a bundled package; the lockfile and conflict rules make that visible.
- The bundled catalog and the application are versioned together. A bundled package update can change simulation behavior; evidence must record the package version.
- Package validation becomes a core product surface, not an afterthought. `playground-cli lint` is the first user of it.
- A future registry can be added as another catalog root or resolver without changing the project format.

## Alternatives considered

**Hardcode supported boards and components in the application.** Rejected: every new part becomes a core change, which defeats the product's modularity goal.

**Only a bundled catalog.** Rejected: it blocks local part development and makes the first vertical slice depend on a release cycle for every fix.

**Only project-local packages.** Rejected: every new project would duplicate the standard catalog, and shared fixes would be difficult to distribute.

**Resolve the newest compatible version at open time.** Rejected: it makes old projects non-reproducible and turns a package update into an implicit project change.

**Load native shared libraries.** Rejected: unsafe, platform-specific, and incompatible with the sandbox and review goals.
