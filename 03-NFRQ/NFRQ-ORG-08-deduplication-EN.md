# NFRQ-ORG-08 — Deduplication: one place per rule, not per shape

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-08-20) — **the identifier is provisional**; entry into the register requires a DR (D-03) |
| **Quality (25010)** | Maintainability — modularity, reusability, analysability |
| **Enforcement** | **not measured** — `wem_lint` does not cover this class; declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

> **Why 08 and not 07.** `NFRQ-ORG-07` (input surface / `ALLOWED`) is cited by
> [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality-EN.md) c.1 and **measured by the lint as an
> error** (`preset_allowed_declaration`), yet it **has no entry** in this register. That is a
> declared blind spot (D-11), not a reason for 08 to take its number.

## 1. Statement

A rule, constant or procedure that holds in several places lives in **one** place; everything
else is a reference.

The measure is **similarity of rule, not similarity of text.** Two blocks of code that look the
same but answer **two different contracts** (e.g. two sub-systems with their own dialects) are
**not duplicates** — merging them is a regression, not an improvement.

## 2. Acceptance criteria

1. A constant, threshold or message shared by two or more modules is declared in **one** place.
2. A procedure repeated **under the same contract** is extracted into a helper or mixin **of the
   domain it belongs to** ([`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality-EN.md)), not into a generic
   `Helper` ([`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature-EN.md)).
3. **`[M]`** The count of repeated blocks above a similarity threshold is a **diagnostic, not a
   gate**: the finding is a candidate for review, not an obligation to merge. Charter:
   [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md).
4. Merging is **not performed** when the repeated blocks serve different contracts; the reason is
   recorded next to the code or in the change record.

## 3. Verification

Review, recorded in the change record. **Machine measurement (c.3) is not implemented** —
declared blind spot (D-11), not coverage.

## 4. Boundary

This requirement is **diagnostic**. It is not a lever for blocking a reasonable implementation:
where deduplication and a clear sub-system contract conflict, **the contract wins**, and the
decision is recorded.

## 5. Justification

| Principle | Implication |
|---|---|
| DRY (Hunt–Thomas): "single, unambiguous, authoritative representation of knowledge" | the unit is **knowledge**, not a line |
| Coupling / cohesion (Parnas) | merging by shape introduces a dependency the domain does not have |
| MDL mirror ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface-EN.md)) | `L(code) + L(abstraction)`: an abstraction hiding differences grows faster than the saving |
| Rule of three | an abstraction is derived from three real cases, not from two similar ones |
