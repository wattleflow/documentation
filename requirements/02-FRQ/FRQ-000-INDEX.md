<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ — functional requirements register

| | |
|---|---|
| **Identifier** | `FRQ-<CATEGORY>-NN` |
| **Version** | Draft v0.2 |
| **Anchor** | ISO/IEC/IEEE 29148 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../../PHILOSOPHY.md) · [METHODOLOGY](../../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/HLRQ-00-INDEX.md) · [NFRQ](../03-NFRQ/NFRQ-000-INDEX.md) |
| **Language** | Index EN; the entries are Croatian (source, `CLAUDE.md` §3.2) |

This document is an **index**, not a register: each requirement lives in its own file
(`category-number-slug.md`). The register holds **an identifier, one statement and a link**; the
detail is not restated here (D-13).

Every FR carries acceptance criteria, a verification method and traceability upwards. A
requirement is either **primary** (derived from a principle, constraining decisions) or
**derived** (arising from a decision); traceability is a **graph, not a chain**
(`METHODOLOGY.md` §6).

**The register runs in both directions.** Some entries **document code that already exists**,
recovered by reverse engineering (`CLAUDE.md` §4 t.1); others, like the `HLRQ-16` group, **precede
the code** so that design and architecture can be reviewed, amended and approved before anything is
written. The entry shape is identical either way — what differs is **Status** and **§Verification**,
which is where it becomes visible whether a claim was measured or merely stated (D-05). A record is
a specification of knowledge, not a work log (**P-14**, Parnas & Clements 1986).

**Entries name roles, not classes.** Prose speaks of roles and families; the identifiers live in the
vocabulary and in the diagrams, which use technical (UML) notation for exactly that purpose
(`CLAUDE.md` §3.3, **P-08**; a diagram is a view, never a source of truth — **P-20**, D-13).
Consequence: renaming a class changes one vocabulary entry, not N places in the texts — and a new
name enters through a decision, not by appearing in a sentence.

The audiences are business stakeholders, end users, developers and testers; no single artefact
serves all of them, and an audience left without one is declared as a gap rather than passed over
in silence (D-11).

**Common definitions** (domain, helper, canonical subject) live in
[`NFRQ-DEF-01`](../03-NFRQ/NFRQ-DEF-01-common-definitions.md) — one register, everything else
references it (D-12).

## AUD — audit record

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-AUD-01` | The record is assembled from named fields, serialised into one message and handed to the per-class logger; the structured multi-sink path with per-field redaction is the target, not the code. | [AUD-01](FRQ-AUD-01-audit-record-path.md) | in code, recorded |

> `AUD` stays **provisional**: audit is a cross-cutting capability, not an ontology primitive,
> so it belongs on the capability axis and still lacks a parent HLRQ.


## HLRQ-01 — the generic layer (`concrete/`)

Parent requirement: [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md).
The role axis is **in the vocabulary** as of
 (2026-08-27); its axis is the
domain ontology (`CLAUDE.md` §1), a closed list of thirteen roles plus `PTN` for the pattern
infrastructure that is not a primitive.

| Id | Statement |   | Status |
|---|---|---|---|
| `FRQ-WFL` | Generic workflow and the factory that resolves every primitive by name from configuration. | [WFL](FRQ-WFL-workflow.md) | in code, recorded |
| `FRQ-DRV` | Generic driver and its lazy proxy — operations against an external system. | [DRV](FRQ-DRV-driver.md) | in code, recorded |
| `FRQ-CON` | Generic connection — access to an external system, with the state machine owned by the generic class. | [CON](FRQ-CON-connection.md) | in code, recorded |
| `FRQ-PRC` | The processor owns the pass: it drives the generator, runs every item through every pipeline, decides the flush boundary and is the only primitive able to resume from a memento. | [PRC](FRQ-PRC-processor.md) | in code, recorded |
| `FRQ-PIP` | The pipeline is one transformation over one item; `process` wraps `transform` with input checks, the opening audit record and a single failure class. | [PIP](FRQ-PIP-pipeline.md) | in code, recorded |
| [`FRQ-BBD`](FRQ-BBD-blackboard.md) | The blackboard is the canvas between the pipeline's per-item rate and the repository's batch rate; it owns the lifecycle and exposes the canvas read-only, while policy stays with the specialisation. | | in code, recorded |
| `FRQ-STR` | The strategy family carries a vocabulary, not behaviour: four families differ only by method name, so the call site's type states which operation is meaningful. | [STR](FRQ-STR-strategy.md) | in code, recorded |
| `FRQ-REP` | Generic repository and the driver-backed variant — persistent storage of items; the driver variant only adds the driver to the strategy context. | [REP](FRQ-REP-repository.md) | in code, recorded |
| `FRQ-DOC` | Document, adapter and facade — the unit of data that travels the flow. | [DOC](FRQ-DOC-document.md) | in code, recorded |
| `FRQ-MEM` | Memento — the snapshot a processor resumes from. | [MEM](FRQ-MEM-memento.md) | in code, recorded |
| `FRQ-PTN` | Root base of every framework object. | [PTN](FRQ-PTN-root-base.md) | in code, recorded |
| `FRQ-ORC` | Orchestrator — runs a set of processors sequentially or in parallel and tells listeners; two collaborators are held but unused. | [ORC](FRQ-ORC-orchestrator.md) | in code, recorded; defects listed in the entry |
| `FRQ-SCH` | Scheduler — event source that sets up, starts and stops one orchestrator; a skeleton until a specialisation supplies it. | [SCH](FRQ-SCH-scheduler.md) | in code, recorded; defects listed in the entry |
| `FRQ-MGR` | Managers of connections, drivers and processors — a name-keyed registry with an operation and teardown at destruction. | [MGR](FRQ-MGR-managers.md) | in code, recorded; defects listed in the entry |
| `FRQ-OBS` | Thread-safe observable — a list of observers notified outside the lock; one failing observer is logged, not propagated. | [OBS](FRQ-OBS-observable.md) | in code, recorded; defects listed in the entry |
| `FRQ-SMC` | State machine and its one-shot guard wrapper — transitions only from a table. | [SMC](FRQ-SMC-state-machine.md) | in code, recorded; defects listed in the entry |
| `FRQ-ITR` | Lazy sync and async iterators — the source is built at the first fetch. | [ITR](FRQ-ITR-lazy-iterators.md) | in code, recorded; defects listed in the entry |
| `FRQ-SGT` | Singleton — one instance per concrete subclass, `__init__` runs once. | [SGT](FRQ-SGT-singleton.md) | in code, recorded; defects listed in the entry |
| `FRQ-HLP` | `Attribute` and `NameHelper` — checks of configuration keys and names for records. | [HLP](FRQ-HLP-helpers.md) | in code, recorded; defects listed in the entry |
| `FRQ-PTN` (exceptions) | Exceptions of the generic layer (`exception.py`). | — | **not yet written** |
| `FRQ-PRC-01.22` | Two flows the primitives collaborate in — creation (processor → blackboard → create strategy) and persistence (pipeline → blackboard → repository → write strategy → driver); N repositories mean N write strategies over one document. | [PRC](FRQ-PRC-01.22-document-flow.md) | proposed |
| `FRQ-SER-CNV` | Generic converter — context of a conversion strategy; holds one strategy and runs it, wraps failures in one error class. | [SER](FRQ-SER-CNV-converter.md) | in code, recorded |
| `FRQ-SER-PAR` | Generic parser — resolves exactly one source (stream, path, payload) into a binary reader for the specialisation; category `PAR`. | [SER](FRQ-SER-PAR-parser.md) | in code, recorded |
| `FRQ-SER-FMT` | Generic formatter — checks mandatory content and optional type, returns the payload and writes nothing; category `FMT`. | [SER](FRQ-SER-FMT-formatter.md) | in code, recorded |

## ORG — organisation and structure

> **Declared blind spot (D-11): the category is empty.** The inherited register carries an
> `FRQ-ORG-01` heading with no content. Fill it per ISO/IEC/IEEE 29148 or withdraw it; until
> then phase 3 of `CLAUDE.md` §4 is not closed. Tracked in [`TODO`].

## Open — category vocabulary

The role axis is closed and in the vocabulary as of
: ten primitives (`WFL`, `PRC`,
`PIP`, `DRV`, `REP`, `BBD`, `STR`, `CON`, `DOC`, `MEM`) plus `CNV`, `PAR`, `FMT` and `PTN`. The capability axis holds
`OSCAL`. `ORG` predates both.

**`AUD` and `MAIL` remain provisional** — neither is a primitive nor `concrete/` infrastructure,
and neither has a parent HLRQ yet. `MAIL` was entered without a recorded decision (2026-08-28): the register is
held to be sufficient justification, exceptions are decided by the documentation. So does the `HLRQ` class itself (`CLAUDE.md` §3.6). Neither the `FR`→`FRQ`
nor the `NFR`→`NFRQ` rename of 2026-08-27 has a recorded decision; both are tracked in
[`TODO`](../../workflow/TODO.md).
