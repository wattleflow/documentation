<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-04 — ZonedDateTimeHelper

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Functional correctness · Maintainability — modularity, analysability · Interoperability |
| **Enforcement** | c.3–c.7 **by test** ([`test_datetime_helpers.py`](../../../workflow/tests/test_datetime_helpers.py)); c.1, c.2 **not measured** — [AST](../../GLOSSARY.md#abbr-ast) candidate (calls of `isoformat`, `strftime`, `fromisoformat`, `strptime` outside `wattleflow.helpers.dtime`); declared [blind spot](../../GLOSSARY.md#slijepa-pjega) (D-11) |
| **Reference frame** | [ISO](../../GLOSSARY.md#abbr-iso) 8601-1:2019 · RFC 3339 (the Internet profile of ISO 8601) · [`NFRQ-DEF-03`](NFRQ-DEF-03-comparison-and-boundary-values.md) §3.1 · [`NFRQ-DEF-05`](NFRQ-DEF-05-datetime-helper.md) (datetime without a zone) · [`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication.md) · [`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard.md) |
| **Implementation** | [`wattleflow.helpers.dtime`](../../../workflow/src/wattleflow/helpers/dtime.py): `ZonedDateTimeHelper`; `DateTimeKind` routes a value whose kind is known only at run time |
| **Raised by** | review of the converters' formatters, 2026-10-06: several written forms of one datetime across converters, metrics and mail |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A **zoned datetime** — a datetime that carries its UTC offset or zone — is the **norm** for every time
the framework records, compares or exchanges: it names one point in time, whatever reads it. Every
zoned datetime written as text shall be written by **`ZonedDateTimeHelper`** on the `helpers/` shelf of
the `workflow` distribution, and RFC 3339 text shall be read into a zoned datetime by the same class;
no component writes or reads that text by its own code.

The **default form** (`ZonedDateTimeHelper.write`) is RFC 3339 with three settings of the installation,
read once at import and never changed at run time:

| setting | default | source |
|---|---|---|
| zone | the system zone | `WATTLEFLOW_TIME_ZONE` (an IANA name or `UTC`) when set |
| precision | microseconds, written in full | `DEFAULT_TIMESPEC` |
| UTC offset | written `Z` | `DEFAULT_UTC_MARKER` (`None` writes `+00:00`) |

With the zone set to `UTC` the default form is `2026-10-06T01:35:02.123456Z`.

A component that needs another form **declares** it in its own requirement, with the facets of
`NFRQ-DEF-03` §2, and obtains it from `ZonedDateTimeHelper.convert` (`zone=`, `timespec=`, `utc_marker=`;
`zone=value.tzinfo` keeps the offset the value carries). A **datetime without a zone** is the other
model (`NFRQ-DEF-05`): `ZonedDateTimeHelper` refuses it and never assumes a zone for it. A value whose
model is decided by its source (metadata read from a document, a mail header) goes through
`DateTimeKind`, which hands it to the helper of its model and to nothing else.

## 2. Acceptance criteria

1. **One writer.** Only `wattleflow.helpers.dtime` writes a datetime as text. No other module in `src/`
   calls `isoformat`, `strftime` or a hand-written format for a time value. *(machine-checkable — AST)*
2. **One reader per standard.** `ZonedDateTimeHelper.read` reads RFC 3339 text (`Z` and `±HH:MM` alike);
   no other module calls `fromisoformat` or `strptime` for that profile. A text of another standard
   (RFC 5322 date headers, the PDF date string) is read by a class that names its standard
   (`NFRQ-ORG-09` c.1). *(machine-checkable — AST)*
3. **Default without parameters.** `write` takes the datetime and nothing else; its result is the
   default form of §1. *(by test)*
4. **A deviation is declared and routed.** Every call of `convert` outside the helper is backed by a
   declaration in the calling component's requirement naming the deviating facet. *(by review)*
5. **The models are not mixed.** `write` and `convert` refuse a datetime without a zone, and `read`
   refuses text without an offset, each with an error naming `DateTimeHelper` as the remedy. *(by test)*
6. **[Round trip](../../GLOSSARY.md#round-trip) is a [fixed point](../../GLOSSARY.md#fixed-point).** For every zoned datetime `t`, `read(write(t)) == t`; for every text
   `s` in the accepted profile, `write(read(s))` is the default form (`NFRQ-FUN-01`). *(by test —
   boundary values of `NFRQ-DEF-03` §4)*
7. **Width is fixed per installation.** For every datetime between the years 1000 and 9999 the default
   form has one width: 27 characters with the zone `UTC`, 32 with any other zone. *(by test for `UTC`)*

## 3. Verification

`workflow/tests/test_datetime_helpers.py` (`unittest`, fixed values, no clock) for c.3, c.5, c.6 and c.7;
the default settings are patched in the test, not read from the machine. c.1 and c.2: an AST scan over
`src/wattleflow` of each distribution for `isoformat`, `strftime`, `fromisoformat` and `strptime` outside
`wattleflow.helpers.dtime` — **not automated**; run on demand, its result is a finding vector, never a
claim of cleanliness (D-11). c.4 by review of the component's requirement at the call site.

## 4. Rationale

| [principle](../../GLOSSARY.md#princip) | implication |
|---|---|
| A zone makes a datetime unambiguous | a zoned datetime names one point in time for every reader; it is the only kind that can be compared, sorted and joined across sources |
| One place per rule (`NFRQ-ORG-08`) | several writers of one datetime produce several texts; records of the converters, the metrics and the mail archive cannot be joined by time |
| Wrap the standard's implementation (`NFRQ-ORG-09`) | `datetime.isoformat` and `fromisoformat` implement the profile; the helper owns only the defaults and the refusal of the other model |
| Two models, never mixed (`NFRQ-DEF-03` §3.1) | a writer that silently assumes a zone moves the assumption into every record it writes |
| Settings of the installation, not of the code | the zone an operator reads is a deployment fact; the code carries the keys, the environment the value (`wattleflow-code` §4) |

## 5. Open

1. **Default zone and text order.** With the system zone as default, the text carries the zone's offset,
   which changes at a daylight-saving boundary: text order is then not time order, and records written
   in different installations do not sort together. Options: default to `UTC` and let a component
   declare a local zone, or keep the system zone. Decision: the author.
2. **The metrics exception** (milliseconds, machine zone) is a decision of `FRQ-MET-01`.
3. **`RFC` is not in the glossary.** The abbreviation enters through `wattleflow-glossary` (D-12).
4. **Identifier.** `DEF-04` is provisional; entry into the register requires a documented change (D-03).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-06 | Two models defined apart (author's decision): this entry is the zoned datetime, the norm, written and read by `ZonedDateTimeHelper` (formerly `Rfc3339`); the datetime without a zone moved to `NFRQ-DEF-05` (`DateTimeHelper`); a value whose model is known only at run time goes through `DateTimeKind`; the term *instant* replaced by *zoned datetime*. File renamed `NFRQ-DEF-04-zoned-datetime-helper.md`. |
| v0.0.5 | 2026-10-06 | Renamed: *Datetime format* (`NFRQ-DEF-04-datetime-format.md`). |
| v0.0.5 | 2026-10-06 | `as_given` and `parse` added for values of run-time kind; every writer and reader in `blackwattle` moved to `Rfc3339`, the converters' tests restated in the default form (microseconds in full, `Z`); §5 t.2 restated as the list of deviations awaiting declaration. |
| v0.0.5 | 2026-10-06 | Title and content aligned with the code: the writer and reader is `Rfc3339` in `wattleflow.helpers.dtime` (with `Zone`, `Now`, `Stamp`), not a helper still to be made; the default form is an installation setting (zone from `WATTLEFLOW_TIME_ZONE`, else the system zone), not a fixed UTC form; c.3–c.7 verified by `test_rfc3339.py`; fixed width restated per installation; the conflict between the system-zone default and text order, and the components that bypass the helper, recorded as open. |
| v0.0.5 | 2026-10-06 | First entry, raised by the review of the converters' formatters; proposal — **the identifier is provisional**. |
