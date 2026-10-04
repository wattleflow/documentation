<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-FUN-02 — Recognition accuracy is measured per engine on a declared reference set

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional suitability — functional correctness; ISO/IEC 25012 accuracy |
| **Enforcement** | **not measured** by `wem_lint`; a measurement script per engine set; declared blind spot (D-11) |
| **Raised by** | `HLRQ-23` `BR-23-07` · `FRQ-OCR-23.2` M7 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

For every recognition engine a converter offers (image to text, speech to text), accuracy is a
**measured value** against a **declared reference set** with ground truth, computed by **one
procedure identical for all engines**. No engine is described as more or less accurate than
another without a measurement of both on the same set. An unmeasured engine is stated as
unmeasured.

## 2. Acceptance criteria

1. **`[M]` Character error rate (CER) and word error rate (WER)** — scale: ratio, unit: edit
   distance over reference length; text normalised by collapsing whitespace. Published as a
   diagnostic with its blind spots, **never as a gate** ([`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md) c.1, c.2).
2. **The reference set is declared:** content, language, degradations, size, provenance. A
   **combined** degradation is a case of its own — single degradations do not predict it
   (measured: CER ≤ 0.012 alone, 0.271 combined, `analysis` §2).
3. **Same set, same procedure, per engine.** A comparison of two engines cites the same set and
   the same script. *(by review)*
4. **Per language.** A language without data is declared **unmeasured**, not extrapolated.
5. **A confidence score is not accuracy.** A score an engine reports is not compared across
   engines and is not presented as an error rate without calibration on the set.
6. **The result carries the triple** (tool, criterion, platform — D-10) and the run date.
7. **Acceptance thresholds** are *calibrated threshold (TBD)*; none is invented here.

## 3. Verification

A measurement script over the declared set, one result file per run, kept beside its analysis
(`06-ANALYSIS/`). **Blind spots:** synthetic images do not represent scanned documents; a single
font and language; engines needing provisioned models cannot be measured where those are absent.

## 4. Rationale

| principle | implication |
|---|---|
| A claim without evidence is an aspiration (D-05) | "engine A errs more than B" needs both measured |
| Diagnostic, not gate ([`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md)) | the value informs the choice; it does not decide it |
| Selection by configuration (`BR-23-02`) | the operator needs numbers to choose, per language and degradation |

## 5. Open

1. **The reference set** — who provides it, and whether it may contain real documents ([`NFRQ-SEC-08`](NFRQ-SEC-08-emitted-document-confidentiality.md)
   governs emitted content).
2. **Speech engines** use the same procedure; whether one set spans both is undecided.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-29) — **the identifier is provisional**; category `FUN` is provisional |
