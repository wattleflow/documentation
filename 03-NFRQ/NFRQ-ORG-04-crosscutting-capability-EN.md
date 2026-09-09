# NFRQ-ORG-04 — Cross-cutting capability as a helper

| | |
|---|---|
| **Status** | Proposal (Draft) |
| **Quality (25010)** | Maintainability — modularity, reusability, analysability |
| **Decision** | [`DR-WFL-001`](../04-DR/DR-WFL-001-helper-capabilities.md) |
| **Enforcement** | `prohibited_standalone` (c.3, shared with ORG-02); c.1 and c.4 by review |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A cross-cutting responsibility that serves several consumers and belongs to no domain primitive —
e.g. **routing** an artefact to a destination (sub-directory / topic / index / table) — is
modelled as a **callable helper capability**, **not** as a specialisation of a domain primitive
(`Strategy`, `Pipeline`, `Driver`, `Processor`). It is placed and named **by capability**
([`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality-EN.md) c.4); class names follow
[`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature-EN.md).

* A capability is **callable from any consumer** (strategy, processor, driver). A `Strategy` is
  not: it is invoked solely by its context (Repository / Blackboard), so a neighbouring
  `Strategy` cannot use it.
* Per-transport variants are polymorphic implementations behind a **stable abstraction** (DIP),
  not GoF Strategy participants.
* The capability's neutral datum (e.g. a `route` label) is transport-agnostic and **does not
  borrow a transport-specific term** for a generic role (`partition` is a Kafka/Spark physical
  partition).

## 2. Acceptance criteria

1. A class or function implementing a cross-cutting capability **does not inherit** a domain
   primitive (`Strategy`, `Pipeline`, `Driver`, `Processor`, `Blackboard`, `Repository`); the
   exposed interface is **callable** (a method/function), not a context delegate.
2. The capability module is named **by capability** (`routing`), not by consuming layer
   (inherited from ORG-01 c.4).
3. Class names respect ORG-02: generic role nouns (`Router`, `Resolver`, `Generator`, `Scanner`)
   are permitted **only when qualified**.
4. A transport-neutral datum does not use a transport-specific term for a generic role.

## 3. Verification

Criterion 3 — `wem_lint` (`prohibited_standalone`); the build fails.
Criteria 1 and 4 — **by review** (inheritance base, and term vocabulary respectively); full
automation is planned, and until then this is a declared blind spot (D-11).
Criterion 2 inherits its enforcement from ORG-01 (acyclicity for placement).

## 4. Holders

`helpers/routing.py` — `RoutingRule`, `PatternSpec`, `DestinationRouter`,
`LocalStorageDestinationRouter`, `route_label` / `route_target`;
`helpers/files.py` — `FileSourceScanner`.
The register gains a `capabilities:` list (currently `routing`) and an extended
`prohibited_standalone`.

## 5. Justification

| Principle | Implication |
|---|---|
| Ontology (`PHILOSOPHY.md`) | `Strategy`/`Pipeline`/… are reserved primitives; a capability is not disguised as one |
| GoF Strategy | invoked by its context, not composable into neighbouring strategies → a capability must be callable |
| GRASP — Pure Fabrication, High Cohesion | a cross-cutting responsibility with no home domain → a fabricated cohesive helper |
| SRP / Separation of Concerns (Parnas, Dijkstra) | "where it is addressed" separated from "how it is persisted" |
| DIP / Stable Abstractions | an abstract contract plus per-transport concretes |
| Open–Closed | a new transport is a new implementation, with no change to consumers |
| Information Hiding (Parnas) | the neutral datum hides transport specifics |
