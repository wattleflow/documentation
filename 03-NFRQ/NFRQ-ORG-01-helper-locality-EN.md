# NFRQ-ORG-01 — Dependency locality of helper classes

| | |
|---|---|
| **Status** | In force |
| **Quality (25010)** | Maintainability — modularity, reusability, analysability |
| **Decisions** | [`DR-WFL-001`](../04-DR/DR-WFL-001-helper-capabilities.md) · [`DR-WFL-019`](../04-DR/DR-WFL-019-helper-fan-in-is-diagnostic.md) · [`DR-WFL-017`](../04-DR/DR-WFL-017-enums-belongs-to-the-core.md) |
| **Enforcement** | `wem_lint` — `domain_acyclicity` (**error**), `helper_fan_in` (`info`, `report` per distribution) |
| **Definitions** | [`NFRQ-DEF-01`](NFRQ-DEF-01-common-definitions-EN.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A helper class resides in the **narrowest module scope that encloses all of its consumers**.

* A helper used by one domain is co-located **inside** that domain.
* A helper used by two or more **sub-packages of one domain** is promoted to that domain's
  **domain-internal shared module** (`pipelines/convertors`).
* A helper used by a package **outside its own domain** belongs on the `helpers/` shelf: the
  shelf holds what other packages use. A consumer may live in a **sibling distribution** —
  `wattleflow.helpers` is a shared name (`DR-WFL-017`).
* Shared helpers are organised **by capability** (`io`, `text`, `routing`, `validation`), never
  by consuming layer (`helpers/pipelines`): a helper shared across layers has no single owning
  consumer.

## 2. Acceptance criteria

1. Every module on the `helpers/` shelf has **at least one consumer outside the shelf**. A module
   that imports no package outside `helpers/` belongs next to its consumer, or is not needed.
2. A domain-local helper class is not imported outside its domain except through that domain's
   **public interface** (`__all__`).
3. **No import edge exists from `helpers/` towards a domain package** (acyclicity).
4. Shared module names denote a **capability**, not a consuming layer.

## 3. Verification

Import-graph analysis (`wem_lint`, later a CI gate).

| Criterion | Rule | Severity | Note |
|---|---|---|---|
| 3 | `domain_acyclicity` | **error** | build fails |
| 1 | `helper_fan_in` | `info` | report shape comes from the distribution's `rules[].report` |
| 2, 4 | — | — | **not automated** — declared blind spot (D-11), not coverage |

## 4. Fan-in is a diagnostic, not a verdict (`DR-WFL-019`)

The consumer count is a **lower bound** for three reasons at once: imports within the shelf are
not counted, relative imports (`from .audit import …`) are not in the graph, and the sibling
distribution is not in the same `--src`. Therefore:

* the finding is **`INFO`**, and relocating a module requires checking consumers in **both**
  distributions;
* **the report shape is a property of the distribution**: where the shelf serves foreign
  consumers (`wattleflow-workflow`) the criterion is stated as **a single statistic**; where the
  shelf and its consumers ship together (`blackwattle`) an unused module is a
  **per-module finding**;
* **fan-in is not proof of belonging.** A cross-cutting capability
  ([`NFRQ-ORG-04`](NFRQ-ORG-04-crosscutting-capability-EN.md)) and a module implementing a contract
  from `core/` stay on the shelf regardless of their consumer count.

The consumer in criterion 1 is counted over **packages**, not sub-packages: a helper shared by
`pipelines/pdf` and `pipelines/png` has consumers only in `pipelines`.

## 5. Justification

| Principle | Implication for placement |
|---|---|
| Common Closure Principle | what changes together is packaged together |
| Common Reuse Principle | grouping by consuming layer violates it when a helper serves several layers |
| Stable Dependencies Principle | a shared core must not depend on volatile domain concepts |
| Acyclic Dependencies Principle | the shelf never imports from a domain package |
| Information Hiding (Parnas) | a domain-internal helper stays behind the domain boundary |
| YAGNI / Occam | `helpers/<capability>` sub-hierarchies only under demonstrated clustering |

## 6. Traceability

Extended across the distribution boundary by
[`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality-EN.md). Placement of cross-cutting capabilities:
[`NFRQ-ORG-04`](NFRQ-ORG-04-crosscutting-capability-EN.md). Module and class names:
[`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature-EN.md).
