# NFRQ-OBS-01 — Audit levels: meaning, audience and placement

| | |
|---|---|
| **Status** | Accepted (`DR-WFL-018`, 2026-08-23; amended by `DR-WFL-021`, 2026-08-24; §1 `INFO` rule amended by `DR-WFL-028`, 2026-09-09 — **proposal**, marked inline) |
| **Quality (25010)** | Maintainability (analysability) · Reliability (recoverability) · Usability (for the operator as the record's reader) |
| **Decisions** | [`DR-WFL-018`](../04-DR/DR-WFL-018-audit-levels-fields-and-volume.md) · [`DR-WFL-021`](../04-DR/DR-WFL-021-audit-trace-follows-the-call-order.md) · [`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md) |
| **Enforcement** | `wem_lint` ≥ 1.13.0 — `audit_level_vs_propagation`, `audit_failure_trace`; both `severity: warning` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

The level of an audit record is determined by **its audience and the question it answers** — not
by its place in the code, nor by a subjective judgement of importance. The rule applies equally
to every class in the framework.

| The record answers | Level | Audience | When read |
|---|---|---|---|
| "Was the work done, and how much?" | `INFO` | operator | always |
| "Which layers did the item pass through?" | `INFO` | operator | always |
| "Why did this item behave differently?" | `WARNING` | operator | always |
| "The work was **not** done" | `ERROR` | operator and on-call | always |
| "The component cannot continue" | `CRITICAL` | on-call | always |
| "What exactly happened before that?" | `DEBUG` | developer | on demand |

Five normative rules follow:

* **`INFO`** carries **two** kinds of record (`DR-WFL-021`): the **work-unit boundary**, written
  by that unit's owner, and the **entry into an operation on a document**, written by a step in
  the persistence chain. Their demarcation, order and volume are governed by
  [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume-EN.md). **`DR-WFL-028` (proposal, in code
  since 2026-09-09):** a document is itself a work unit owned by the **processor**, so its closing
  record is of the first kind, not the second; and the **pipeline layer** leaves the second kind
  entirely — its record keeps its fields and position but drops to `DEBUG`.
* **`WARNING`** marks a **valid but degraded outcome** — a fallback taken, an item skipped, a
  duplicate deduplicated. **Never a failure.**
* **`ERROR`** marks work that was not done. It is written by **the layer that handles the
  failure**, and **once per cause**.
* **`CRITICAL`** is reserved for **non-continuability**: the component enters an unusable state.
  "The operation broke, we continue with the next one" is **not** `CRITICAL`.
* **`exception()`** (a record with a traceback) is used **only** where the traceback is the
  content of the record **and** the exception is not raised further. Otherwise `error()`.

**Trace and propagation.** A layer that **wraps and re-raises** an exception leaves a trace at
**`DEBUG`** level (`step=Failed`, `error=`) and carries the context **in the exception itself**
(`error=`, `exc=`, `add_context()`). It does **not** emit `ERROR`. Every class therefore still
leaves a trace of the failure, while one cause yields **one** `ERROR`, at the layer that actually
handles it. Enabling those traces is an **operational** decision, not a code change: each
component carries its own `level`, `handler` and `formatting` (`WorkflowFactory._audit`).

## 2. Acceptance criteria

1. No `except` branch both emits `error()`/`critical()`/`exception()` **and** re-raises (`raise`)
   in the same branch. The rule holds for every path ending in a raise, including a `raise` from
   the `try` body; that wider form is **not machine-decidable** and is checked by review.
2. An `except` branch that re-raises leaves **exactly one** trace at `debug()` level, with
   `step=Event.Failed.name` and the key `error=`.
3. `exception()` appears solely in a branch **without** `raise`.
4. `warning()` does not appear on a path ending in a raise or in a failure return.
5. `critical()` appears solely where the component enters an unusable state (an FSM transition to
   `Failed`/`Stopped`, a cycle abort); "the iteration broke" is `error()`.
6. Every class that catches an exception **leaves a trace**: either `error()`/`critical()` if it
   handles it, or `debug()` + `raise` if it propagates it. **Silent swallowing is a violation.**

**Three exceptions to c.6**, all machine-recognisable:

| Exception | Reason |
|---|---|
| **No `self` in scope** — a branch at module level or in a module function | there is nothing to write through; the exception message is the sole carrier of context |
| **Bare pass-through** — `except <Exception>: raise` with no modification | the layer that raised it already left a trace; a second record would be the duplicate c.1 forbids |
| **Guarded optional dependency** (`DR-WFL-003`) — an `except ImportError` raising a domain error with installation guidance | the failure is configurational, not data-borne: the exception carries the whole story, and the boundary records it once |

The list of guarded types is **criterion data** (`tools/dictionary.json → audit.import_guard`).

## 3. Verification

| Criterion | Rule | Severity |
|---|---|---|
| 1 | `audit_level_vs_propagation` | `warning` |
| 2, 6 | `audit_failure_trace` | `warning` |
| 3 | `audit_level_vs_propagation` (`exception()` ∈ escalating levels) | `warning` |
| 4, 5 | — | **not machine-decidable** — declared blind spot (D-11) |

The current state is a **worklist**; the build does not fail. Raising to `error` requires a DR.

## 4. State of enforcement

The `strategies` layer is aligned (`DR-WFL-018`); the reference pattern is
`strategies/documents/*.py` — entry `debug`, outcome `debug`, degradation `warning`, failure
`debug` + `raise`. The other packages are a worklist; **that state is not transcribed here**
(D-13) but read from the conformance snapshot in
[`../workflow/conformance/`](../workflow/conformance/).

## 5. Justification

| Principle | Implication |
|---|---|
| Signal detection theory; base rate (same anchor as [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point-EN.md)) | false alarms wreck the operating point: an `ERROR` that is not a failure spends the attention a real failure needs |
| Shannon's measure of information | a level whose record is always present conveys no information; a repeated record of the same cause has zero entropy |
| RFC 5424 §6.2.1 (Syslog severity) | the scale is **ordinal and operational**, not an expression of the author's concern |
| Fail-fast + exception context (PEP 3134 `raise … from`) | failure context belongs to the **exception** (the `__cause__` chain), not to a second log record |
| Single Point of Truth | one cause → one `ERROR`; other layers leave a trace at `DEBUG` |
| Least Astonishment | the same situation gets the same level in every class; the level is a **filter**, not a style |

## 6. Traceability

Record fields — [`NFRQ-OBS-02`](NFRQ-OBS-02-audit-fields-EN.md).
Volume and ownership — [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume-EN.md).
Content confidentiality — [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality-EN.md).
