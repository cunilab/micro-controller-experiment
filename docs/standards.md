---
id: MCE-STD-001
title: "Micro Controller Playground Documentation Standards"
status: proposed
---

# Micro Controller Playground Documentation Standards

Applies to everything under `docs/`. The root `README.md` is the short entry point and has no header.

The goal is documentation that stays correct with little effort. Git already records who changed what and when; the docs do not duplicate that history.

## Header

Every governed document starts with this YAML header and nothing before it:

```yaml
---
id: MCE-<AREA>-NNN
title: "Matches the H1 heading"
status: proposed
---
```

| Field | Rule |
| --- | --- |
| `id` | Permanent and unique, `MCE-<AREA>-NNN`. ADRs use `MCE-ADR-NNNN`. Never reused, even if the file moves. |
| `title` | Matches the H1 heading. Keep it quoted; an unquoted `:` breaks the header. |
| `status` | `draft`, `proposed`, `accepted`, or `superseded`. No other fields. |

Status describes the text, not the software. Whether something is implemented is recorded only in [Evidence](./evidence.md). A document never claims a feature works.

Lifecycle: `draft` → `proposed` (open for review) → `accepted` (maintainer approved) → `superseded`. A substantive edit to an accepted document returns it to `proposed` until re-approved.

## One source of truth per topic

Each rule lives in exactly one document; others link to it instead of restating it.

| Topic | Owner |
| --- | --- |
| Product purpose, users, goals, scope, principles | [`product.md`](./product.md) |
| Milestone order and outcomes | [`roadmap.md`](./roadmap.md) |
| System structure, boundaries, interfaces, flows | [`architecture.md`](./architecture.md) |
| Feature behavior and boundaries | [`specs/`](./specs/) |
| Pass/fail checks and recorded results | [`evidence.md`](./evidence.md) |
| Rationale for an expensive or hard-to-reverse choice | [`adr/`](./adr/) |
| Documentation rules | this file |

If the same rule appears in two places, delete one copy and link to the owner.

## Changing documents

- New documents: copy the relevant template from the organization template repository, choose a new ID, and add a row to the [index](./README.md).
- A contract change needs an ADR or an ADR update in the same PR. Contracts include:
  - the project file schema;
  - the board and component package schemas;
  - the host/device protocol;
  - the component and board API boundaries;
  - the evidence vocabulary.
- An ADR number is never reused. If a decision is replaced, keep the old file and mark it `superseded`.
- Prefer plain English, full sentences, and defined terms. Do not use unexplained abbreviations.
- Delete stale documentation or update it. A stale document is worse than no document.

## Review checklist

- [ ] Header has `id`, `title`, and `status`; the ID is unique.
- [ ] `title` matches the H1 heading.
- [ ] Relative links resolve.
- [ ] No rule is duplicated from its owning document.
- [ ] No claim that something is implemented or passing without a linked result in [Evidence](./evidence.md).
- [ ] A changed contract is covered by an ADR.
- [ ] Plans are written as plans, not as claims about current behavior.
