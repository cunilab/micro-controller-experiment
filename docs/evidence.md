---
id: MCE-EVID-001
title: "Validation and Evidence — Documentation Baseline and v0.1"
status: proposed
---

# Validation and Evidence — Documentation Baseline and v0.1

This document owns pass/fail claims. It records what was actually observed, with the commit, platform, command, and artifact. Plans and intentions do not belong here.

No implementation evidence exists yet. Every check below is pending until the first vertical slice lands.

## Checks

| # | Check | How to verify | Result |
| --- | --- | --- | --- |
| E-1 | Every governed document has a valid header, unique ID, and resolving relative links. | Docs review plus a planned `scripts/check-docs` check in CI. | ⏳ pending |
| E-2 | Catalog validation rejects duplicate IDs, missing pins, pin capability mismatch, and invalid bus configuration. | `playground-cli lint` against valid and invalid fixtures. | ⏳ pending |
| E-3 | The example `blink` project runs headless for a bounded time and produces the expected trace. | `playground-cli run projects/examples/blink` with a fixed seed. | ⏳ pending |
| E-4 | A button press changes the trace at the expected simulation time. | Scripted input fixture + golden trace. | ⏳ pending |
| E-5 | Two runs of the same project with the same seed produce the same trace hash on one platform. | Run twice on Linux, compare hashes. | ⏳ pending |
| E-5b | The same trace hash is produced on two operating systems. | Run the same project and seed on two platforms in CI. | ⏳ pending |
| E-6 | The same project runs in the simulator and on the real board. | Run the example on both backends; compare observable behavior, not just exit code. | ⏳ pending |
| E-7 | A malformed or truncated protocol frame is rejected without dispatching side effects. | Protocol unit tests and fuzz corpus. | ⏳ pending |
| E-8 | A second board package can be added without changing `playground-core` or the UI. | Diff review + catalog/build test with a second board fixture. | ⏳ pending |
| E-9 | A second component pack can be added without changing `playground-core` or the UI. | Diff review + catalog/build test with a second component fixture. | ⏳ pending |
| E-10 | Flashing refuses a firmware image whose board identity does not match the selected board. | Negative test with a mismatched image fixture. | ⏳ pending |
| E-11 | A user-authored WASM component cannot access the filesystem or network by default. | Sandbox capability test (deferred until the WASM SDK exists). | ⏳ pending |

## Evidence

No results recorded yet.

When recording a result, include:

- **Result:** ✅ pass, ❌ fail, ⏳ pending, or ⏭️ not run.
- **How verified:** exact command, CI lane, or manual steps.
- **Observed:** what was actually seen, not what was expected.
- **Artifact:** link to a CI run, log, screenshot, or committed trace.
- **Source:** commit or tag, platform, and date.

## Deferrals and Limits

| Check | Reason | Tracking |
| --- | --- | --- |
| E-1 docs CI check | The docs baseline lands before the tooling skeleton. | Add `scripts/check-docs` with the first tooling PR. |
| E-6 physical run | No implementation or supported board yet. | v0.3 hardware path. |
| E-10 flashing guard | No flasher adapter yet. | v0.3 hardware path. |
| E-5b cross-platform hash | v0.1 evidence starts on Linux; the second platform is added in v0.4. | v0.4 buses and CI. |
| E-11 WASM sandbox | WASM SDK is planned for v1.0. | v1.0 extensibility. |

Host-simulation results never close a physical check. A passing unit test never proves hardware behavior. A passing emulator test does not prove the physical board.

## Open Questions

- Which board is used for the first physical check?
- Which platform is the reference for the first cross-platform trace hash?
- What exact observable behavior proves simulator/hardware parity without overclaiming electrical equivalence?
