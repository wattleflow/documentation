<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-PRF-02 — Measurement speed: the cost of measuring is bounded by the work measured

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Performance efficiency — time behaviour |
| **Enforcement** | **benchmark**, rerun when a measurement mechanism changes — `2026-09-24-measurement-explicit-vs-audit` |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

> **Demarcation.** [`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) governs the **shape** of a
> metric; [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume.md) the volume of audit records;
> `HLRQ-18` BR-07 that a measurement fault never stops
> work. This entry governs the **speed** of measuring: what it may cost.

## 1. Statement

Measuring an operation slows it by at most a declared share of its own work. A measurement
mechanism states its cost per measured operation; an operation too small for that cost is measured
in aggregate, not one by one.

## 2. Definitions

- `w` — median work of one operation, in ns, without measurement.
- `c` — cost the mechanism adds per measured operation, in ns (scenario minus bare call).
- `p` — measurement budget: the permitted relative slowdown `c / w`.
- `c_off` — cost at a call site when measurement is off (capability present, object not registered).

## 3. Acceptance criteria

1. **Budget.** An operation measured one by one satisfies `c / w ≤ p`, with default `p = 1 %`
   and at most `p = 5 %` when the entry measuring it declares why.
2. **Granularity.** An operation with `w < c / p` is not measured one by one. Its enclosing
   operation, a batch or the pass is measured instead (per-row measurement is excluded, as in
   [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume.md)).
3. **Declared cost.** Every measurement mechanism declares `c` and `c_off`, measured by the method
   of §4, on a stated platform (D-10).
4. **Off is nearly free.** A capability present on every framework object costs nothing where it
   is not called, and `c_off ≤ 200 ns` where it is.
5. **Independent of the logger.** Measuring does not require an open log level. The cost of an
   emitted record does not enter `c` (measured at 12.6 µs per operation: 5.4 × the integrated
   mechanism).
6. **No growth with volume.** Memory held by measurement does not grow with the number of
   operations (aggregation on close).
7. **CPU time per pass, not per operation**, unless `p` still holds with its cost included.

## 4. Verification

The benchmark in `06-ANALYSIS`: empty operation on a real `Wattleflow` class; per scenario the
least of 7 × 200 000 calls, median of 3 processes, scenarios interleaved; `c` = scenario − bare.

Measured 2026-09-24 (CPython 3.11.15, WSL2, 4 CPU):

| mechanism | `c` | `w_min` at 1 % | `w_min` at 5 % |
|---|---:|---:|---:|
| integrated (audit → `Monitor`, `OPERATIONS`) | 2 326 ns | 233 µs | 47 µs |
| explicit (`performance(op)`, `with`) | 813 ns | 81 µs | 16 µs |
| explicit, off (`c_off`) | 162 ns | — | — |

## 5. Rationale

At small volume every mechanism is negligible: 18 measured operations per document cost 42 µs
integrated against hundreds of ms of work. At large volume the cost is linear, `ΔT = N · c`: 10⁹
operations cost 39 min integrated and 14 min explicit. At a device's rate (SDR, 213 µs per block),
the integrated mechanism takes 26 % of the budget. A budget relative to `w` covers both cases with
one rule.

## 6. Open

| item | note |
|---|---|
| default `p` | 1 % proposed; the documentation decides |
| mechanism | integrated, explicit, or both behind one interface — the analysis §3 |
| registration by weakref | framework classes with a full `__slots__` chain carry no `__weakref__` |
| real `w` per role | the budget needs measured medians per operation, not the empty operation |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-24) — category `PRF` is provisional ([`NFRQ-PRF-01`](NFRQ-PRF-01-work-proportional-to-input.md)) |
