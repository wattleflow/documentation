# NFRQ-OBS-04 — What a metric must carry to be admissible

| | |
|---|---|
| **Status** | Proposal (2026-09-07). Written **before** the first collector, not after |
| **Quality (25010)** | Performance efficiency (time behaviour, resource utilisation) · Maintainability (analysability) |
| **Enforcement** | **by review**; c.1 and c.6 are AST-checkable and are candidates for `wem_lint` |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md) — every criterion here inherits it |
| **Operating point** | [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point-EN.md) — a threshold is a point on a ROC, not a preference |
| **Category** | `OBS` was introduced for **audit-record** observability. A metric is not an audit record, so this entry **widens the category**. Provisional pending a DR (D-12) — §6 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A number is not a measurement. Shewhart's distinction between **common-cause** variation
(inherent to the process, predictable within limits) and **special-cause** variation (assignable)
is what turns a number into information, and it needs a **distribution** to be drawn at all.
Deming's corollary is the operational one: reacting to common-cause variation as though it were
special — *tampering* — **increases** variation rather than reducing it.

So this entry does not ask for more numbers. It fixes what a number must carry before anyone is
entitled to act on it.

## 2. Acceptance criteria

1. **A duration or size is published as a distribution, never as a single value.** At minimum
   `_count` and `_sum`; where a quantile is wanted, buckets. A mean is **derived** at query time
   (`rate(_sum)/rate(_count)`), never stored — a stored mean cannot be re-aggregated and hides
   the dispersion that makes it meaningful.
2. **`[M]` Every metric declares its scale, unit and subject.** Ratio scale, SI unit (`seconds`,
   `bytes`), and the subject the count is *per*. Only operations valid for the scale are
   permitted (`NFRQ-DEF-02` c.1).
3. **Labels are rational subgroups, and are justified as such.** Populations that differ in kind
   are not pooled: a 3 kB text PDF read by a local library and a 40 MB scan read by OCR are
   different processes, and limits computed over their union describe neither. A label exists to
   separate a subgroup — not to carry an identifier.
4. **Cardinality is bounded and declared.** Label values come from a closed, small set. A
   filename, a document identifier, a path or a timestamp is **never** a label. The entry states
   the expected number of series.
5. **A rate has its denominator.** A failure count without an attempt count is not a rate, and a
   throughput without a duration is not a speed. Counters are published in pairs.
6. **The operational definition states inclusive or exclusive.** Layers nest — a blackboard's
   write contains the repository's, which contains the strategy's, which contains the driver's.
   The same work is therefore reported four times, and only **exclusive** time may be summed.
   A metric that does not say which it is has no operational definition (Deming) and its values
   are not comparable between observers.
7. **No threshold without limits.** An alert derives its limits from the process's own history,
   not from a chosen round number. Until validated out-of-sample the metric is **diagnostic**;
   promotion to anything that stops work requires a DR (`NFRQ-DEF-02` c.5).
8. **The measurement reports its own health.** A collector publishes what it could not measure —
   unclosed spans, dropped records, expired pairs. A metric that cannot say how complete it is
   invites the reader to mistake a gap for a zero.

## 3. Verification

| criterion | method | status |
|---|---|---|
| 1, 5 | review of the published series: is `_count`/`_sum` present, is each counter paired? | reviewable |
| 2, 4 | the entry that introduces the metric declares scale, unit and cardinality | reviewable |
| 3 | review — a label without a stated subgrouping reason is a finding | reviewable |
| 6 | AST: a method opening a span closes it exactly once (`Completed` **or** `Failed`) | **candidate for `wem_lint`** |
| 7 | no gate exists today; the criterion is preventive | vacuously true, by design |
| 8 | the collector's own series are present in the published set | not implemented |

**Reproducibility triple (D-10):** tool — review, plus AST for c.6; criterion — §2 above;
platform — declared by the holder. **Declared blind spot:** nothing here is measured by
`wem_lint` yet; c.6 is the only rule whose shape the existing `audit_*` rules could carry.

## 4. Why this is not over-specification

Measured on one document through the real chain (2026-09-07): the same work is reported by four
layers as 26.62 / 26.51 / 26.32 / **25.79 ms**. Summing them yields 105 ms for 26 ms of work — a
fourfold overstatement, produced not by a bug but by the absence of c.6. Separately, the same
pass leaves spans open on every failing path, so a failure disappears from the counts instead of
appearing in them (c.8).

Both are cheap to prevent before the first collector exists and expensive to correct in dashboards
already trusted.

## 5. Boundary

* `NFRQ-OBS-01/02/03` govern the **audit record**: its level, fields and volume. A metric is not
  an audit record and does not inherit the INFO-placement rule; it is not addressed to a reader.
* `NFRQ-DEF-02` governs **whether a measure may be used at all**. This entry governs **what shape
  it must take** once it may.
* `NFRQ-SEC-04` governs **where the threshold sits** once one is justified.

## 6. Open

1. **The category.** `OBS` was introduced for audit-record observability (`DR-WFL-018`). A metric
   is a different subject with the same concern, so either `OBS` widens by DR, or a category for
   measurement is opened. Cost of widening: the demarcation in the `OBS` header stops being true.
2. **Bucket boundaries are not yet derivable.** Shewhart requires limits computed from the
   process, and no history exists. Until it does, publish `_count`/`_sum` with coarse buckets and
   treat every boundary as provisional — a bucket set chosen now is a guess wearing a number.
3. **Inclusive, exclusive, or both** (c.6) — undecided, and it is a definition, not a default.

## 7. Justification

| Principle | Implication |
|---|---|
| Shewhart — common vs special cause | a point carries no information about which it is; the distribution does (c.1) |
| Shewhart — rational subgrouping | limits over pooled populations describe none of them (c.3) |
| Deming — tampering (funnel experiment) | acting on common-cause variation increases it; hence c.7 |
| Deming — operational definitions | a name without a stated procedure is not measurable by two people alike (c.6) |
| Goodhart / Campbell (via `NFRQ-DEF-02`) | a measure that becomes a target stops measuring — hence diagnostic before gate |
| D-09 (vector, not scalar) | no single "throughput" figure aggregates across layers or units |
| D-11 (declared blind spots) | a gap in measurement is published, not left to look like zero (c.8) |
