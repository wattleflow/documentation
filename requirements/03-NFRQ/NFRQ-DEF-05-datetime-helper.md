<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-05 — DateTimeHelper

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional correctness · Maintainability — modularity, analysability · Interoperability |
| **Enforcement** | c.2–c.5 **by test** ([`test_datetime_helpers.py`](../../../workflow/tests/test_datetime_helpers.py)); c.1 **by review** |
| **Reference frame** | [ISO](../../GLOSSARY.md#abbr-iso) 8601-1:2019 (local time, no UTC offset) · [`NFRQ-DEF-03`](NFRQ-DEF-03-comparison-and-boundary-values.md) §3.1 · [`NFRQ-DEF-04`](NFRQ-DEF-04-zoned-datetime-helper.md) (zoned datetime, the norm) |
| **Implementation** | [`wattleflow.helpers.dtime`](../../../workflow/src/wattleflow/helpers/dtime.py): `DateTimeHelper`; `DateTimeKind` routes a value whose kind is known only at run time |
| **Raised by** | author's decision 2026-10-06: a datetime without a zone is not the norm and is defined apart from it |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A **datetime without a zone** — a calendar date and clock time with no UTC offset — is **not the norm**:
it does not name one point in time, because each reader may place it in a different zone. It is
permitted only where its source gives no zone (a date bound in a configuration, a date printed in a
document, a value of a format without a zone) and only through **`DateTimeHelper`** on the `helpers/`
shelf of the `workflow` distribution.

`DateTimeHelper` writes such a datetime as ISO 8601 text without an offset (`write`, microseconds by
default, `timespec=` when declared), reads such text back (`read`), and is the **only crossing** into the
norm: `zoned(value, zone=…)` gives the zoned datetime of `NFRQ-DEF-04` in the zone the caller names
(an IANA name, `UTC`, or the installation's zone passed explicitly). It never assumes a zone and never
compares, sorts or orders a datetime without a zone with a zoned one; a component that must compare
first makes the value zoned. A value whose model is decided by its source goes through `DateTimeKind`.

## 2. Acceptance criteria

1. **Declared use.** Every component that holds a datetime without a zone declares in its requirement
   why its source gives no zone and, when it compares, the zone it makes it zoned in. *(by review)*
2. **The models are not mixed.** `write` refuses a zoned datetime and `read` refuses text with an offset,
   each with an error naming `ZonedDateTimeHelper` as the remedy. *(by test)*
3. **One crossing, zone named.** `zoned` requires the zone as an argument; there is no default zone and
   no implicit conversion anywhere else. *(by test)*
4. **Round trip.** For every datetime without a zone `t`, `read(write(t)) == t`. *(by test)*
5. **Routing keeps the model.** `DateTimeKind.text` and `DateTimeKind.parse` give a datetime without a
   zone to `DateTimeHelper` and a zoned one to `ZonedDateTimeHelper`, unchanged. *(by test)*

## 3. Verification

`workflow/tests/test_datetime_helpers.py` (`unittest`, fixed values) for c.2–c.5. c.1 by review of the
component's requirement; the components that hold a datetime without a zone today are listed in §5.

## 4. Rationale

| [principle](../../GLOSSARY.md#princip) | implication |
|---|---|
| A datetime without a zone is ambiguous | two readers in two zones place it at two points in time; it cannot be compared with the norm until a zone is named |
| Two models, never mixed (`NFRQ-DEF-03` §3.1) | a separate helper makes the exception visible at every call; nothing slides from one model into the other without `zoned` |
| One crossing | the zone is decided once, where the value is made zoned, and is written in the caller's code, not assumed by a helper |

## 5. Open

1. **Components that hold a datetime without a zone today:** the date window of `CreatedWithin`
   (date bounds from the configuration, made zoned in the installation's zone before comparison),
   the PDF date without a zone (`PdfParser.pdf_date`), mail headers without a zone (outside RFC 5322) and printed dates (a `-0000`, military or unknown zone name is UTC by RFC 5322 §3.3 and §4.3, so zoned),
   RSS and W3C-DTF dates without a time, and metadata values of documents. Each needs the declaration
   of c.1 in its requirement.
2. **Identifier.** `DEF-05` is provisional; entry into the register requires a documented change (D-03).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-06 | §5 t.1: mail headers with `-0000`, military or unknown zone names are zoned (UTC), not without a zone (`FRQ-MAIL-01-message-parser-ANL`). |
| v0.0.5 | 2026-10-06 | First entry: the datetime without a zone defined apart from the zoned norm (`NFRQ-DEF-04`), written and read by `DateTimeHelper`, with `zoned` as the only crossing (author's decision). |
