<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-05 — MomentNaiveHelper

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional correctness · Maintainability — modularity, analysability · Interoperability |
| **Enforcement** | c.2–c.6 **by test** ([`test_moment.py`](../../../workflow/tests/test_moment.py)); c.1 **by review** |
| **Reference frame** | [ISO](../../GLOSSARY.md#abbr-iso) 8601-1:2019 (local time, no UTC offset) · [`HLRQ-MMN`](../01-HLRQ/HLRQ-MMN-moment.md) `BR-MMN-13`, `BR-MMN-14` · [`NFRQ-DEF-03`](NFRQ-DEF-03-comparison-and-boundary-values.md) §3.1 · [`NFRQ-DEF-04`](NFRQ-DEF-04-moment-aware-helper.md) (aware moment, the norm) |
| **Implementation** | [`wattleflow.helpers.moment`](../../../workflow/src/wattleflow/helpers/moment/helper.py): `MomentNaiveHelper`; `MomentHelper.text` and `MomentHelper.parse_iso` for a value whose kind its source decides |
| **Raised by** | author's decision 2026-10-06: a datetime without a zone is not the norm and is defined apart from it |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A **wall time** — a naive `Moment`: calendar date and clock time with no zone — is **not the norm**: it
does not name one point in time, because each reader may place it in a different zone. It is permitted
only where an operator writes it (a date bound in a configuration) or a source gives no zone (a date
printed in a document, a value of a format without a zone), and only through **`MomentNaiveHelper`** in the
`workflow` distribution.

`MomentNaiveHelper` writes a wall time as ISO 8601 text without an offset (`to_iso`; the default form of
`MomentHelper.text` writes microseconds in full) and reads such text back (`from_iso`). The **only
crossing** into the norm is `localize(m, zone)`, and the zone is always named by the caller: the
workflow zone for a value an operator wrote (`BR-MMN-13`), the zone a source declares for its values
(`BR-MMN-14`). Nothing assumes a zone, and a wall time is never compared, sorted or ordered with an aware
moment; a component that must compare first localizes the value in a named zone. A wall time has no
RFC 5322 form: its `-0000` means UTC (`HLRQ-MMN` `BR-MMN-09`).

## 2. Acceptance criteria

1. **Declared use.** Every component that holds a wall time declares in its requirement whether an
   operator or a source wrote it and, when it compares, the zone it localizes it in. *(by review)*
2. **The models are not mixed.** `MomentNaiveHelper` refuses an aware moment and `from_iso` refuses text
   with an offset, each with a `TypeError` naming the expected kind; comparing the two kinds is a
   `TypeError`. *(by test)*
3. **One crossing, zone named.** `localize` requires the zone as an argument; there is no default zone and
   no implicit conversion anywhere else; nanoseconds survive it. *(by test)*
4. **Round trip.** For every wall time `t` with whole microseconds, `from_iso(to_iso(t)) == t`. *(by test)*
5. **Routing keeps the kind.** `MomentHelper.parse_iso` gives a wall time for text without an offset and an
   aware moment for text with one; `MomentHelper.text` writes each in its own form. *(by test)*
6. **No RFC 5322 form.** `MomentNaiveHelper` has no `to_rfc2822`. *(by test)*

## 3. Verification

`workflow/tests/test_moment.py` (`unittest`, fixed values) for c.2–c.6. c.1 by review of the component's
requirement; the components that hold a wall time today are listed in §5.

## 4. Rationale

| [principle](../../GLOSSARY.md#princip) | implication |
|---|---|
| A wall time is ambiguous | two readers in two zones place it at two points in time; it cannot be compared with the norm until a zone is named |
| Two models, never mixed (`NFRQ-DEF-03` §3.1) | a separate helper makes the exception visible at every call; nothing slides from one model into the other without `localize` |
| One crossing, zone named by whoever knows it | the operator's zone is the workflow zone; a source's zone is the source's declaration; the helper assumes neither |

## 5. Open

1. **Components that hold a wall time today**, each needing the declaration of c.1:
   - the date bounds of `CreatedWindow` (operator; localized in the workflow zone);
   - the PDF date without a zone (`PdfParser`, its PDF date);
   - mail headers without a zone (outside RFC 5322) and printed dates (source; `source_zone`);
   - RSS dates without a zone (kept as text when written);
   - record times in the data quality index (source; `timestamp_zone`).
2. **Identifier.** `DEF-05` is provisional; entry into the register requires a documented change (D-03).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-07 | Restated over `Moment` (`HLRQ-MMN`; author's decision: `Moment` replaces `helpers/dtime.py`): the wall time is a naive `Moment` handled by `MomentNaiveHelper`; the one crossing is `localize` with the workflow zone (operator) or the source zone (source); no RFC 5322 form; §5 updated. File renamed `NFRQ-DEF-05-moment-naive-helper.md`. |
| v0.0.5 | 2026-10-06 | §5 t.1: mail headers with `-0000`, military or unknown zone names are zoned (UTC), not without a zone (`FRQ-MAIL-01-message-parser-ANL`). |
| v0.0.5 | 2026-10-06 | First entry: the datetime without a zone defined apart from the zoned norm (`NFRQ-DEF-04`), written and read by `DateTimeHelper`, with `zoned` as the only crossing (author's decision). |
