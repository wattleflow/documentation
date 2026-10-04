<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-FUN-01 — Conversion fidelity: nothing lost silently, the round trip is a fixed point

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional suitability — functional correctness, functional completeness; ISO/IEC 25012 completeness, accuracy |
| **Enforcement** | **not measured** by `wem_lint` — unit tests per component; declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |
| **Raised by** | `HLRQ-19` `BR-05`, `BR-08` |

> **Demarcation.** [`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard.md) governs **whose model** a
> format component exposes. This entry governs **what survives** the crossing.

## 1. Statement

A component that converts content between two formats keeps every piece of source content in the
target, or **declares** what it drops. Over the subset it declares for both directions, reading
back what it wrote is a **fixed point**. A conversion does not depend on earlier conversions by the
same instance.

## 2. Acceptance criteria

1. **Declared subset.** The component's requirement lists the constructs it recognises in each
   direction, naming the external clause where one exists ([`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard.md) c.1).
2. **No silent loss.** Content outside the subset is carried as text. Content that cannot be carried
   is a **declared loss** in the component's requirement; an undeclared loss is a defect.
3. **Literal markup survives.** Text that reads as markup in the target format is escaped, so it
   comes back as text, not as structure.
4. **Fixed point.** For a sample covering the subset, `read(write(read(write(x))))` equals
   `read(write(x))`.
5. **Independence.** Repeated calls on one instance give what calls on fresh instances give.
6. **Deviation is declared.** Where c.2 overrides the external standard (e.g. keeping table cells
   the standard would drop), the deviation is stated at the point in code and in the requirement.

## 3. Verification

Unit test per criterion, each confirmed by a mutation that the test must fail.

**Witness (D-05):** the Word converter — the defects and their correction are recorded in
 declared losses in
`FRQ-DRV-19.2` §7.

Other converters (PDF, XML, RSS) are **not assessed** under this entry.

## 4. Rationale

A conversion is where content changes hands between formats that do not share a model; a loss there
is invisible downstream, because the consumer only ever sees the target. The fixed point is the
cheapest mechanical witness that the two directions agree — it needs no oracle, only the component
itself.

## 5. Open

1. **Category.** `FUN` is not in the vocabulary; [`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard.md) records the same quality under `ORG`.
   Which axis carries functional correctness is for the documentation to decide.
2. **Scope of c.4** for one-way components (a parser with no formatter) — criterion does not apply;
   c.2 still does.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-14) — **identifier and category `FUN` are provisional**; a new quality-axis category enters with a documented change |
