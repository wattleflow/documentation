<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-PRF-01 — Work proportional to input

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Performance efficiency — time behaviour, resource utilisation |
| **Enforcement** | **not measured** by `wem_lint` — unit test per component; declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |
| **Raised by** | `HLRQ-19` |

> **Demarcation.** `HLRQ-18` observes load **at run
> time**; [`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) governs metrics. This entry is a
> **design property of code**: how work grows with one item's size.

## 1. Statement

The work a component does to convert, parse or format **one item** grows at most linearly with the
size of that item, unless a super-linear cost is declared with its reason and bound.

## 2. Acceptance criteria

1. **No whole-collection search inside a loop over that collection.** Looking each element up in a
   list rebuilt per iteration is the usual source of quadratic cost.
2. **Measured by work, not by clock.** The test counts a deterministic unit of work (objects built,
   elements visited) for an input of size *n*; the count stays within a constant multiple of *n*.
3. **`[M]`** Ratio of work at *k·n* to work at *n* — **diagnostic, not a gate**. Charter:
   [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md).
4. **Declared exception.** A super-linear step names why it is needed and the largest input it is
   meant for.

## 3. Verification

Unit test with a work counter; confirmed by a mutation reintroducing the super-linear step.

**Witness (D-05):** the Word reader — 40 000 paragraph objects for 200 paragraphs before
 within 2 × *n* after.

## 4. Rationale

A per-item transformation runs once per document in every pass. Quadratic cost is invisible on test
data and dominant on the long document that matters. A count of work shows it at any size; a time
ratio does not, because fixed cost dominates small inputs (c.2).

## 5. Open

1. **Category.** `PRF` is not in the vocabulary; [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume.md) and [`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) record performance
   efficiency under `OBS`. Which axis carries it is for the documentation to decide.
2. Other components are **not assessed** under this entry.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-14) — **identifier and category `PRF` are provisional**; a new quality-axis category enters with a documented change |
