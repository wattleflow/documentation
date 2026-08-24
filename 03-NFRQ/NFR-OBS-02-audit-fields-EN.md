# NFR-OBS-02 — Audit-record fields

| | |
|---|---|
| **Status** | Accepted (`DR-WFL-018`, 2026-08-23; c.2 amended by `DR-WFL-021`, 2026-08-24) |
| **Quality (25010)** | Maintainability (analysability); ISO/IEC 25012 (consistency, understandability) for the record itself |
| **Decisions** | [`DR-WFL-018`](../workflow/dr/DR-WFL-018-audit-levels-fields-and-volume.md) · [`DR-WFL-021`](../workflow/dr/DR-WFL-021-audit-trace-follows-the-call-order.md) |
| **Enforcement** | `wem_lint` ≥ 1.13.0 — `audit_event_vocabulary` (`warning`) |
| **Vocabulary** | `wattleflow.enums.event` · `tools/dictionary.json → audit.fields` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](0-NFRQ-EN.md#language)) |

## 1. Statement

An audit record is a **pair (event, named fields)**, not a sentence. Every field name has
**exactly one** meaning across the framework and answers one question:

| Key | Answers | Value | Required |
|---|---|---|---|
| `msg` | **what** is being done — the operation/event | `Event.<Member>.name` | **always** |
| `step` | **where in time** within that operation | `Event.Started` / `Completed` / `Failed` / `Validating` / `Configuring`… `.name` | when the record marks a transition |
| `scope` | **which part** of the same operation | a short literal (`"insert"`, `"preflight"`) or `Event.<Member>.name` | when the operation has distinguishable sub-operations |
| `component` | **what it uses** — a sub-system or external participant | a literal (`"ssl"`, `"tika"`, `"proxy"`) | when the operation touches an external sub-system |
| `target` | **on which kind of object** it operates | a literal (`"driver"`, `"processor"`) | when the same operation works on several kinds |
| `reason` | **why** the outcome is what it is — a human sentence | a literal | `WARNING`; with `ERROR` where a sentence explains |
| `error` | **what broke** — the exception text | `str(e)` | `ERROR`, `CRITICAL`, failure traces |

The axes are **orthogonal**: one record may carry them all, but **no key takes over another's
role**. A sub-system name is not `msg`; a phase is not `scope`; a sentence is not `msg`.

## 2. Acceptance criteria

1. `msg` is **always** `Event.<Member>.name`. A literal, an f-string, a local variable and
   `.value` are violations.
2. The entry and outcome of the same operation share a `msg` and differ **only** in `step`.
   **Exception — the work-unit boundary** (`DR-WFL-021`): a record that opens or closes a unit
   carries the phase **in `msg`** (`Execute`/`Start` versus `Completed`) and carries **no**
   `step`. The reason is the audience — two records with the same name, told apart only by a
   field, force the operator to read the field to know which of the two they are looking at. The
   exception applies **solely to the unit's owner**
   ([`NFR-OBS-03`](NFR-OBS-03-audit-ownership-volume-EN.md)), never to a step in the chain.
3. The value of `step` is an `Event` member from the **phase set** — `Started`, `Starting`,
   `Validating`, `Configuring`, `Check`, `Completing`, `Completed`, `Failed`. A free literal, a
   non-phase member (`Configuration`, `Iterating`…) and a `.value` record are violations.
4. `component`, `target` and `scope` **carry no message text** and no method name.
5. No audit call unpacks the caller's dictionary (`**kwargs`) into its arguments — a caller's key
   (`msg`, `exc_info`, `extra`, `stacklevel`) would otherwise become a control signal or break
   the call. The criterion is **shared** with
   [`NFR-SEC-06`](NFR-SEC-06-audit-confidentiality-EN.md) c.1 and is **cited here, not
   duplicated**.
6. Keys come from a controlled vocabulary (`tools/dictionary.json → audit.fields`, which records
   the question each one answers); **a new key enters through a DR**.

## 3. Verification

| Criterion | Method |
|---|---|
| 1, 3, 5 | `wem_lint` — `audit_event_vocabulary`, `severity: warning` |
| 2, 4 | **by review** — they require judgement |
| 6 | depends on keys being entered in the dictionary; **not automated** |

## 4. Declared exceptions and current state

Two exceptions to c.1, both in the foundation layer:

* `helpers/audit.py` — the `IObserver` sink; the message is necessarily dynamic.
* `helpers/exception.py` — the stdlib logger inside `AuditException`; calling through `Audit`
  would recurse.

The `step` axis is **not consolidated**: alongside phase members, `.value` forms and free
literals also occur (`step="completed"`, `step="jar-resolved"`). That is a **worklist, not a
state**; the measured state is read from the conformance snapshot (D-13), not from this text.

`Event` is a large register; a minority of members have a `value` differing from their `name`,
and those are **multi-word labels, not errors** — `.name` is the stable identifier, `.value` is
the **rendering** and may change (D-13).

## 5. Justification

| Principle | Implication |
|---|---|
| Faceted classification (Ranganathan; same anchor as [`NFR-APX-01`](NFR-APX-01-facet-order-EN.md)) | the record is searched **by facet** (operation × phase × sub-system), not by full text |
| Structured logging (Chen–Jiang) | a machine-readable field survives as far as the SIEM; a sentence does not |
| Controlled vocabulary (D-12) | `Event` is the event register; a literal in `msg` bypasses it |
| Identifier stability (D-13) | `.name` is the member name; `.value` is the rendering |
| Least Astonishment | the same key name means the same thing in every class — otherwise a `component=` filter returns an incomplete result |
