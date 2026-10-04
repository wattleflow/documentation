<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-08 — Deduplication: one place per rule, not per shape

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — modularity, reusability, analysability |
| **Enforcement** | **not measured** — `wem_lint` does not cover this class; declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A rule, constant or procedure that holds in several places lives in **one** place; everything
else is a reference.

The measure is **similarity of rule, not similarity of text.** Two blocks of code that look the
same but answer **two different contracts** (e.g. two sub-systems with their own dialects) are
**not duplicates** — merging them is a regression, not an improvement.

## 2. Acceptance criteria

1. A constant, threshold or message shared by two or more modules is declared in **one** place.
2. A procedure repeated **under the same contract** is extracted into a helper or mixin **of the
   domain it belongs to** ([`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality.md)), not into a generic
   `Helper` ([`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature.md)).
3. **`[M]`** The count of repeated blocks above a similarity threshold is a **diagnostic, not a
   gate**: the finding is a candidate for review, not an obligation to merge. Charter:
   [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md).
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
| MDL mirror ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md)) | `L(code) + L(abstraction)`: an abstraction hiding differences grows faster than the saving |
| Rule of three | an abstraction is derived from three real cases, not from two similar ones |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-08-20) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
