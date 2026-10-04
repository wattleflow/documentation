<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-03 — Comparison and boundary-value definitions

| | |
|---|---|
| **Version** | 0.1 (2026-09-13) — status changes with a documented change (D-03); §6 lists what needs a documented decision |
| **Role** | What every compared or bounded value must declare before it reaches code; the definitions live here, requirements reference them |
| **Parent rule** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) `BR-PTN-06` |
| **Register** | [`0-NFRQ`](NFRQ-000-INDEX.md) |
| **Anchor** | ISO/IEC/IEEE 29119-4 (boundary value analysis) · ISO/IEC/IEEE 29148 (verifiable acceptance criteria) · ISO 8601 · IEEE 754 · The Unicode Standard (UTF-8, UAX #29) |

## 1. Statement

A comparison or a limit is **defined in the documentation before it is implemented**. For every
value that is compared, bounded, truncated, rounded or allocated for, the requirement that uses
it declares the facets in §2 — once, here when shared, in the component's FR/NFR when specific.
A facet left undeclared is a **documentation defect**, not an implementation choice: two
implementations of the same undeclared bound are both correct and disagree.

Python tolerates much of this silently (unbounded integers, one text type, dynamic comparison
errors at run time). C++ does not: a width is chosen at compile time, a buffer is allocated to a
size, text has a byte length and a character length, and the choice leaks into every consumer.
The port to C++ is what makes an undeclared bound visible; it is not what makes it wrong.

## 2. The declaration — mandatory facets

| Facet | The question it answers | Example of the failure when undeclared |
|---|---|---|
| **Kind** | What is compared with what — only values of the same kind are comparable | A time with an offset compared with one without (§5, case 1) |
| **Unit** | Seconds or milliseconds; bytes or characters; KiB or KB | A text limit of 255 met in Python and exceeded in C++ (§3.2) |
| **Precision** | The smallest distinguishable step, and what happens to finer input | A nanosecond file time compared with a microsecond bound |
| **Inclusivity** | Is the bound itself inside: `[a, b]`, `[a, b)`, `(a, b]` | A file at exactly the end of the window |
| **Reference** | The zone of a wall time, the encoding of a text, the base of a number | A naive date read as UTC by one side and local by the other |
| **Out of range** | Reject, clamp, truncate (with a marker), or count and continue | A value silently wrapped by an unsigned integer |
| **Absence** | Absent ≠ None ≠ empty ≠ zero — each with its meaning | An omitted `level` read as `NOTSET` resets another component's level |

## 3. Proposed framework defaults

Each default applies where the requirement does not declare otherwise. A default marked
**OPEN** is proposed, not settled (§6).

### 3.1 Time

1. **Two kinds, never mixed.** An *instant* (a point on the UTC line, "aware") and a *wall time*
   (a clock reading without a zone, "naive") are different kinds. A value of the other kind is
   **rejected where it enters**, not at the comparison that would fail later.
2. **A wall time names its zone.** The component declares the zone a wall time is read in; the
   system zone is a declared choice, not a fallback.
3. **Precision is the microsecond.** Finer input (a C++ file time in nanoseconds) is truncated
   toward the past before it is compared.
4. **Intervals are half-open, `[start, end)`.** **OPEN** — the code today uses a closed interval
   whose date-only end is 23:59:59.999999. Half-open is proposed because it does not depend on the
   precision (the closed end changes meaning when the precision changes) and adjacent windows
   neither overlap nor leave a gap.
5. **A date-only bound** starts at 00:00:00 of that date; its end is 00:00:00 of the next date
   (half-open) or 23:59:59.999999 of the same date (closed), per item 4.
6. **Daylight saving.** A wall time that occurs twice resolves to the earlier instant. A wall time
   that does not occur is **OPEN** (§6).
7. **Leap seconds are not represented** (POSIX time) — declared, not handled.
8. **Accepted text** is the ISO 8601 subset the component lists; anything else is rejected, never
   guessed.

### 3.2 Text

1. **Encoding is UTF-8.** Invalid sequences: **OPEN**.
2. **A length names its unit** — bytes (UTF-8 code units), code points, or grapheme clusters
   (UAX #29). **OPEN** default; proposed: limits that allocate (buffers, columns, fields on the
   wire) in **bytes**, limits shown to a person in **code points**, grapheme clusters only when
   declared. The same text has different lengths: `Đurđevac` is 8 code points and 10 bytes — Python
   `len()` counts the first, C++ `size()` the second.
3. **Truncation never splits a code point**, and a truncation marker counts inside the limit.
4. **Comparison is by code point** — no locale collation, no case folding, no Unicode
   normalisation unless declared.

### 3.3 Numbers

1. **An integer declares width and signedness.** Overflow is an error, never a wrap: Python
   integers do not overflow, a C++ unsigned wraps and a signed overflow is undefined behaviour.
2. **A quantity with fixed decimal places** (money, a measured value with a stated resolution) is
   not held in binary floating point. The places and the **rounding mode** are declared. The
   rounding default is **OPEN**: Python `round()` rounds half to even (`round(2.5) == 2`), C++
   `std::round` half away from zero (`std::round(2.5) == 3.0`) — the same rule written twice gives
   two answers.
3. **Floating-point values are not compared for equality**; the tolerance and its kind (absolute or
   relative) are declared. NaN is rejected where it enters unless declared; so are infinities.
4. **The unit is part of the type or the name** — a C++ duration type, or `timeout_seconds`.

### 3.4 Collections and allocated sizes

1. **Cardinality** — minimum, maximum, and whether empty is valid — is declared.
2. **Whether order is significant** is declared. (The keyword channel keeps insertion order, as
   Python does — PEP 468.)
3. **A size that allocates** (a buffer, a queue) is declared with what happens when it is reached:
   block, drop and count, or error. A dropped item is reported, never silent (cf. `HLRQ-16` `BR-07`).

### 3.5 Absence

1. **Absent, None, empty and zero are four values.** Each has a declared meaning. The framework
   already depends on the difference: an omitted `level` must not reset a level another component
   set, which a reading of absence as `NOTSET` would do.

## 4. Verification

Boundary value analysis (ISO/IEC/IEEE 29119-4). For every declared bound `B` with smallest step
`u`, the tests cover **`B − u`, `B`, `B + u`**, and, where the facets allow them: empty, absent,
the maximum, and the first invalid form.

| Level | What it proves |
|---|---|
| **Unit** | The component applies the declared facets — in isolation, with fixed clocks and zones |
| **System** | The facets survive the real environment: the machine zone, the filesystem's time resolution, the encoding on the wire |
| **UAT** | The acceptance criterion is stated with **explicit values** — "a file created at the last representable moment of the end date is inside the window" — so a user can tell a pass from a fail without reading code |

An FR acceptance criterion (29148) that names a bound without its facets fails review.

## 5. Evidence — why this entry exists

1. **Kind mixed at a comparison.** `CreatedWithin("2026-01-01T00:00:00+10:00", None)` constructs
   and then raises `TypeError: can't compare offset-naive and offset-aware datetimes` on the first
   `check()` (Python 3.11.15, WSL2, measured 2026-09-13). Nothing declared that a bound must be the
   same kind as the file time. `workflow/TODO.md`, urgent after the C++ port.
2. **Order undeclared.** The C++ keyword channel was first built unordered, on the claim that
   Python "never promised" an order; PEP 468 does. The audit record printed fields in a different
   order from Python until corrected (2026-09-13).
3. **Length unit undeclared — in the C++ port itself.** The audit record truncates a value to 100
   (Python `safe_repr`, which counts characters); the C++ `Repr::safe` cuts by bytes, keeping code
   points whole. For multi-byte text the two keep a different amount. Not changed: the correct unit
   is §6 O2.
4. **Rounding undeclared.** No component rounds today; §3.3 item 2 records the trap before one does.

## 6. Open — needs a documented decision

| # | Question | Where it bites first |
|---|---|---|
| O1 | Interval closure default: half-open `[start, end)` or closed as the code is today | `CreatedWithin`; any window, schedule or retention period |
| O2 | Default unit of a text length | Audit truncation (§5 case 3); any field with a maximum |
| O3 | Default rounding mode | The first component with decimal places |
| O4 | Invalid UTF-8: reject or replace with U+FFFD | Parsers, documents from external sources |
| O5 | A wall time that does not occur (spring forward): reject, or move to the transition | Naive bounds in a zone with daylight saving |
| O6 | Scope: this entry is shared, the rule sits in [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) where the defect was found — does a framework-wide rule need its own HLRQ | `blackwattle` specialisations |

## 7. Traceability

[`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) `BR-PTN-06` · [`NFRQ-DEF-01`](NFRQ-DEF-01-common-definitions.md) (common definitions) ·
[`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md) (a bound that is measured is an `[M]`
criterion) · ISO/IEC 25010 functional correctness · `METHODOLOGY.md` §6 (traceability is a graph).
