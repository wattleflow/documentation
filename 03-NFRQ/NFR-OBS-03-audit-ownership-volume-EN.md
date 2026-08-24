# NFR-OBS-03 — Ownership, order and volume of audit `[M]`

| | |
|---|---|
| **Status** | Accepted (`DR-WFL-018`, 2026-08-23; statement and c.1–3 amended by `DR-WFL-021`, 2026-08-24) |
| **Quality (25010)** | Maintainability (analysability) · Performance efficiency (resource utilisation) |
| **Decisions** | [`DR-WFL-018`](../workflow/dr/DR-WFL-018-audit-levels-fields-and-volume.md) · [`DR-WFL-021`](../workflow/dr/DR-WFL-021-audit-trace-follows-the-call-order.md) |
| **Enforcement** | `wem_lint` — `audit_info_placement` (`warning`); c.5 carries `[M]` |
| **`[M]` charter** | [`NFR-DEF-02`](NFR-DEF-02-measurement-charter-EN.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](0-NFRQ-EN.md#language)) |

> **The linter's criterion lags this text.** The `audit_info_placement` and
> `audit_event_vocabulary` rules still measure against `DR-WFL-018` and report the `DR-WFL-021`
> shape as a finding. Until tool and criterion are aligned, those findings **are not
> violations**; declared blind spot (D-11), not coverage. No conformance snapshot exists for
> `OBS-01/02/03` until then.

## 1. Statement

**`INFO` grows with the number of work units and with the number of layers an item passes
through — never with the number of steps inside one layer.** The record serves **operational
traceability**: the operator must see whether the work was done *and* which layers the item
passed through.

Two kinds of record follow:

| Kind | Written by | When | Shape |
|---|---|---|---|
| **work-unit boundary** | the unit's owner | opening and closing | phase in `msg`, **no** `step` |
| **entry into an operation on a document** | a step in the persistence chain | **on entry**, before handing down | operation in `msg`, `step=Started` |

| Work unit | Owner (writes the boundary) |
|---|---|
| a workflow run | orchestrator / workflow |
| a cycle / batch | processor |
| a document | **no boundary** — completing an individual document is not recorded (`DR-WFL-021` §Cost) |

**The order is the call order.** The chain's entry records line up in the order the layers are
called: `Pipeline.transform → Blackboard.write → Repository.write → Strategy.write →
Driver.write`. The flow then reads like the component's activity diagram. **A completion record
cannot provide this** — completion is recorded only after the work is done, so its order is
necessarily the reverse of the call order; "one record on completion" and "order as in an
activity diagram" are mutually exclusive.

**The entry record carries identity and intent, not outcome** — the source name, the document
identifier, the target name or the resolved path. The outcome is carried by the unit boundary
(`files=`, `cycles=`, `processed=`).

**Failure is read from where the block stops**, together with the `ERROR` from
[`NFR-OBS-01`](NFR-OBS-01-audit-levels-EN.md). A block ending at `Strategy Write Started` with no
following line says it stopped in the strategy; no extra record is needed for that.

**The record belongs to the layer's generic method, not to a specialisation** — where such a
method exists. The seams are `GenericPipeline.process`, `GenericRepository.write`,
`RepositoryWithDriver.write`, `StrategyWrite.write`. Blackboard and driver have none (`write` is
abstract, or delegates), so there the record sits **per class**.

## 2. Acceptance criteria

1. The unit's owner emits **exactly two** `INFO` records per unit — opening and closing — with
   the phase in `msg` and **no** `step`.
2. A step in the persistence chain emits **exactly one** `INFO` record per document, **on
   entry**, with the operation in `msg` and `step=Event.Started.name`. The step emits **no**
   completion `INFO`; its completion stays at `DEBUG`.
3. The record sits in the **layer's generic method** if the layer has one; otherwise per class.
   The same record emitted **both** generically **and** in a specialisation is a violation — it
   yields two records for one step.
4. The owner's `INFO` record carries outcome **counters** (found, processed, written, skipped,
   duplicates), not a repetition per item. A step's entry record carries **no** outcome counters.
5. **`[M]`** *Records per work unit, per level, per layer.* Scale **ratio**, unit
   "record / work unit". Reference values (**diagnostic, not a gate**): `INFO` = 2 per unit per
   owner, 1 per document per chain layer; `DEBUG` ≤ 2 per method (entry + outcome), plus records
   that capture a **decision**. Calibrated thresholds for other levels: **TBD**.
6. A `DEBUG` record that is neither an entry, an outcome nor a decision is a **candidate for
   removal**; its presence is **justified, not assumed**.
7. Disabling `DEBUG` is an **operational** measure per component (`level`, `handler`,
   `formatting`), not a reason to omit the record from the code.
8. The rule applies to **every** workflow and processor with its persistence layer. A new
   pipeline, blackboard, repository, strategy or driver without the records from c.1–c.3 is a
   **finding**, not a documentation omission.

## 3. Verification

| Criterion | Method |
|---|---|
| 1–3 | `wem_lint` — `audit_info_placement`; the list of step classes, unit owners and exceptions is criterion data (`tools/dictionary.json → audit`). **The rule still measures against `DR-WFL-018`** — see the banner. |
| 5 | measurement over a run (records / units); **diagnostic**. Promotion to a gate requires a DR with a validation dossier ([`NFR-DEF-02`](NFR-DEF-02-measurement-charter-EN.md) item 5). |
| 6 | **not automated** — telling a "decision" from a "pass-through" requires judgement; declared blind spot |
| 4, 7, 8 | review |

## 4. Why `DEBUG` is expensive

The chain measured for a single document (2026-08-23): pipeline + repository + strategy + driver
produce on the order of **ten** `DEBUG` records per document. At 10,000 documents that is on the
order of 10⁵ records — which is why `DEBUG` is not free. Exact figures are not transcribed here
(D-13); they are re-measured.

The canonical example of c.4: `WriteAttachmentDocument.info(TaskCompleted)` fires once per
attachment, while the same information already exists aggregated in `WriteMailBundle`
(`attachments=`, `written=`, `duplicates=`).

**Current state under `DR-WFL-021`.** Generic seams cover whole layers at once:
`GenericPipeline.process` covers every pipeline, `GenericRepository` and `RepositoryWithDriver`
every repository. The strategy layer **has** a seam (`StrategyWrite.write`), but the record sits
in a concrete class — one writer is covered, not the layer. Blackboard and driver have no seam
and go class by class. The remainder of those three layers is a **worklist, not conformance**.

Introducing a template method in `GenericBlackboard.write` / `GenericDriver.write` changes the
`core` contract and requires a decision in the `DR-COR` series — until then, per-class migration
is the only path.

## 5. Boundary

The requirement governs **quantity and placement**, not the content of an individual record.
Where silence and failure traceability conflict, **traceability wins**: the record is not deleted
but moved to the level and layer it belongs to
([`NFR-OBS-01`](NFR-OBS-01-audit-levels-EN.md)).

## 6. Justification

| Principle | Implication |
|---|---|
| Signal-to-noise ratio | thousands of `INFO` records for one job mean `INFO` no longer answers "is the job done" |
| Amdahl / cost of I/O in a hot loop | a record per item scales with the **input**, not with the job |
| Aggregation at the point of ownership | counters (`written=`, `duplicates=`, `files=`) carry the same information in one record |
| Goodhart (same anchor as the `[M]` charter) | the record count is not the goal; the threshold is a diagnostic, not a quota |
