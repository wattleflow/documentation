<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-04 — MomentAwareHelper

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional correctness · Maintainability — modularity, analysability · Interoperability |
| **Enforcement** | c.3, c.5–c.7 **by test** ([`test_moment.py`](../../../workflow/tests/test_moment.py)); c.1, c.2 **not measured** — [AST](../../GLOSSARY.md#abbr-ast) candidate (calls of `isoformat`, `strftime`, `fromisoformat`, `strptime` outside `wattleflow.helpers.moment`); declared [blind spot](../../GLOSSARY.md#slijepa-pjega) (D-11); c.4 **by review** |
| **Reference frame** | [ISO](../../GLOSSARY.md#abbr-iso) 8601-1:2019 · RFC 3339 (the Internet profile of ISO 8601) · `HLRQ-MMN` · [`NFRQ-DEF-03`](NFRQ-DEF-03-comparison-and-boundary-values.md) §3.1 · [`NFRQ-DEF-05`](NFRQ-DEF-05-moment-naive-helper.md) (wall time) · [`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication.md) · [`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard.md) |
| **Implementation** | [`wattleflow.helpers.moment`](../../../workflow/src/wattleflow/helpers/moment/helper.py): `MomentAwareHelper`; `MomentHelper.text` and `MomentHelper.parse_iso` for a value whose kind its source decides |
| **Raised by** | review of the converters' formatters, 2026-10-06: several written forms of one datetime across converters, metrics and mail |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

An **aware moment** — a `Moment` that names one point in time (UTC nanoseconds) with an IANA zone for
display — is the **norm** for every time the framework records, compares or exchanges: it names the same
point for every reader. Every aware moment written as ISO 8601 or RFC 3339 text shall be written by
**`MomentAwareHelper`** (or `MomentHelper.text`) in the `workflow` distribution, and such text shall be read
by the same helpers; no component writes or reads that text by its own code.

The **default form** (`MomentHelper.text`) is RFC 3339 in the moment's zone, microseconds written in
full, and UTC written `Z`: `2026-10-06T01:35:02.123456Z`. A moment made by the system (`now`) carries the
**workflow zone** (`HLRQ-MMN` `BR-MMN-12`): the zone a workflow configures (`runtime.time_zone`), else the
global setting `WATTLEFLOW_TIME_ZONE`, else the system zone; it is settled once.

A component that needs another form **declares** it in its own requirement, with the facets of
`NFRQ-DEF-03` §2, and obtains it from `MomentAwareHelper.to_iso` (`tz=`, `timespec=`, `z=`). A **wall time**
is the other model (`NFRQ-DEF-05`): `MomentAwareHelper` refuses it and never assumes a zone for it.

## 2. Acceptance criteria

1. **One writer.** Only `wattleflow.helpers.moment` writes a time value as text. No other module in `src/`
   calls `isoformat`, `strftime` or a hand-written format for a time value. *(machine-checkable — AST)*
2. **One reader per standard.** `MomentAwareHelper.from_iso` and `MomentHelper.parse_iso` read RFC 3339
   text (`Z` and `±HH:MM` alike); no other module calls `fromisoformat` or `strptime` for that profile. A
   text of another standard (RFC 5322 date headers, the PDF date string) is read by a class that names its
   standard (`NFRQ-ORG-09` c.1). *(machine-checkable — AST)*
3. **Default without parameters.** `MomentHelper.text` takes the value and nothing else; its result is the
   default form of §1. *(by test)*
4. **A deviation is declared.** Every call of `to_iso` with `tz=`, `timespec=` or `z=` outside the helpers
   is backed by a declaration in the calling component's requirement naming the deviating facet.
   *(by review)*
5. **The models are not mixed.** `MomentAwareHelper` refuses a wall time and `from_iso` refuses text
   without an offset, each with a `TypeError` naming the expected kind. *(by test)*
6. **[Round trip](../../GLOSSARY.md#round-trip) is a [fixed point](../../GLOSSARY.md#fixed-point) at the
   microsecond.** For every aware moment `m` with whole microseconds, `from_iso(text(m)) == m`;
   nanoseconds survive in serialisation (`to_bytes`, `to_dict`), not in text (`HLRQ-MMN` `BR-MMN-08`).
   *(by test — boundary values of `NFRQ-DEF-03` §4)*
7. **Width is fixed per zone.** For every moment between the years 1000 and 9999 the default form has one
   width: 27 characters in UTC, 32 in any other zone. *(by test for UTC)*

## 3. Verification

`workflow/tests/test_moment.py` (`unittest`, fixed values; the workflow zone is patched, not read from the
machine) for c.3, c.5, c.6 and c.7. c.1 and c.2: an AST scan over `src/wattleflow` of each distribution for
`isoformat`, `strftime`, `fromisoformat` and `strptime` outside `wattleflow.helpers.moment` — **not
automated**; run on demand, its result is a finding vector, never a claim of cleanliness (D-11). c.4 by
review of the component's requirement at the call site.

## 4. Rationale

| [principle](../../GLOSSARY.md#princip) | implication |
|---|---|
| A moment is unambiguous | an aware moment names one point in time for every reader; it is the only kind that can be compared, sorted and joined across sources |
| One place per rule (`NFRQ-ORG-08`) | several writers of one time produce several texts; records of the converters, the metrics and the mail archive cannot be joined by time |
| Wrap the standard's implementation (`NFRQ-ORG-09`) | `datetime.isoformat` and `fromisoformat` implement the profile; the helpers own only the defaults, the zone rule and the refusal of the other model |
| Two models, never mixed (`NFRQ-DEF-03` §3.1) | a writer that silently assumes a zone moves the assumption into every record it writes |
| The zone is configuration, not code | the zone an operator reads is a deployment fact; the code carries the key (`WATTLEFLOW_TIME_ZONE`), the environment or the workflow the value |

## 5. Open

1. **Workflow zone and text order.** Text in a zone with daylight saving carries an offset that changes at
   the boundary: text order is then not time order, and records of installations in different zones do not
   sort together. Options: set the workflow zone to `UTC` where records are sorted as text, or sort by the
   moment, never the text. Decision: the author.
2. **The metrics form** (milliseconds, workflow zone) is a declaration of `FRQ-MET-01`.
3. **`RFC` is not in the glossary.** The abbreviation enters through `wattleflow-glossary` (D-12).
4. **Identifier.** `DEF-04` is provisional; entry into the register requires a documented change (D-03).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-07 | Restated over `Moment` (`HLRQ-MMN`; author's decision: `Moment` replaces `helpers/dtime.py`): the norm is the aware moment, written and read by `MomentAwareHelper` and `MomentHelper.text`/`parse_iso`; the default zone is the workflow zone (`BR-MMN-12`); verified by `test_moment.py`. File renamed `NFRQ-DEF-04-moment-aware-helper.md`. |
| v0.0.5 | 2026-10-06 | Two models defined apart (author's decision): the zoned datetime, the norm, written and read by `ZonedDateTimeHelper` (formerly `Rfc3339`); the datetime without a zone moved to `NFRQ-DEF-05`; the term *instant* replaced. File renamed `NFRQ-DEF-04-zoned-datetime-helper.md`. |
| v0.0.5 | 2026-10-06 | Renamed: *Datetime format* (`NFRQ-DEF-04-datetime-format.md`). |
| v0.0.5 | 2026-10-06 | Title and content aligned with the code (`Rfc3339` in `wattleflow.helpers.dtime`); the default form an installation setting; the conflict between the system-zone default and text order recorded as open. |
| v0.0.5 | 2026-10-06 | First entry, raised by the review of the converters' formatters; proposal — **the identifier is provisional**. |
