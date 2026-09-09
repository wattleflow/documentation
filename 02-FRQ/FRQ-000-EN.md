# FRQ — functional requirements register

| | |
|---|---|
| **Identifier** | `FRQ-<CATEGORY>-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Anchor** | ISO/IEC/IEEE 29148 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/HLRQ-000-EN.md) · [NFRQ](../03-NFRQ/NFRQ-000-EN.md) |
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
[`NFRQ-DEF-01`](../03-NFRQ/NFRQ-DEF-01-common-definitions-EN.md) — one register, everything else
references it (D-12).

## AUD — audit record

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-AUD-01` | The record is assembled from named fields, serialised into one message and handed to the per-class logger; the structured multi-sink path with per-field redaction is the target, not the code. | [AUD-01](FRQ-AUD-01-audit-record-path.md) | in code, recorded |

> `AUD` stays **provisional**: audit is a cross-cutting capability, not an ontology primitive,
> so it belongs on the capability axis and still lacks a parent HLRQ
> ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) t.4).


## MAIL — reading a mail message

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-MAIL-01` | Parties, times, text, attachments and identity of a message are available to the process in the same shape whatever container it arrived in, and no consumer's need decides in advance which of them is read. | [MAIL-01](FRQ-MAIL-01-message-parser.md) | in code, recorded |
| `FRQ-MAIL-02` | Where OCR damage breaks the header labels a rule depends on, a trained character n-gram classifier decides whether the extracted text is correspondence — and only there, never where the rule already decides. | [MAIL-02](FRQ-MAIL-02-degraded-text-classifier.md) | proposed, **not implemented** |

> `MAIL` is **provisional**, like `AUD`: a capability, not an ontology primitive. Its number does
> **not** reference a parent, because the entry describes a **reusable element** rather than the
> decomposition of one process — two workflows (`06_fetch_emails`, `04_pii_reduction`) reach the
> same component for different ends, so traceability here is a graph, not a chain
> ([`METHODOLOGY`](../METHODOLOGY.md) §6). Options and their cost: entry §11.
>
> First entry to carry a binding **Norms** row — the applicable NFRQs are declared by the
> requirement, not judged case by case at implementation time.


## MET — metric collection

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-MET-01` | Duration, quantity and outcome are derived from the `Started`/`Completed` pairs the code already emits, corrected in one place, and routed to zero or more endpoints — no component knows a monitor exists. | [MET-01](FRQ-MET-01-metric-collection.md) | proposed, **not implemented** |

> `MET` is **provisional**, like `AUD` and `MAIL`: a capability, not an ontology primitive, and
> without a parent HLRQ. Entry into the vocabulary requires a DR (D-12).


## HLRQ-15 — the generic layer (`concrete/`)

Parent requirement: [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md).
The role axis is **in the vocabulary** as of
[`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) (2026-08-27); its axis is the
domain ontology (`CLAUDE.md` §1), a closed list of ten primitives plus `PTN` for the pattern
infrastructure that is not a primitive.

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-BBD-15.1` | The blackboard is the canvas between the pipeline's per-item rate and the repository's batch rate; it owns the lifecycle and exposes the canvas read-only, while policy stays with the specialisation. | [15.1](FRQ-BBD-15.1-blackboard.md) | in code, recorded |
| `FRQ-PIP-15.2` | The pipeline is one transformation over one item; `process` wraps `transform` with input checks, the opening audit record and a single failure class. | [15.2](FRQ-PIP-15.2-pipeline.md) | in code, recorded |
| `FRQ-PRC-15.3` | The processor owns the pass: it drives the generator, runs every item through every pipeline, decides the flush boundary and is the only primitive able to resume from a memento. | [15.3](FRQ-PRC-15.3-processor.md) | in code, recorded |
| `FRQ-STR-15.4` | The strategy family carries a vocabulary, not behaviour: four families differ only by method name, so the call site's type states which operation is meaningful. | [15.4](FRQ-STR-15.4-strategy.md) | in code, recorded |
| `FRQ-CON-15.5` | Generic connection — access to an external system, with the state machine owned by the generic class. | — | **not yet written** |
| `FRQ-DRV-15.6` | Generic driver and its lazy proxy — operations against an external system. | — | **not yet written** |
| `FRQ-REP-15.7` | Generic repository and the driver-backed variant — persistent storage of items. | — | **not yet written** |
| `FRQ-DOC-15.8` | Document, adapter and facade — the unit of data that travels the flow. | — | **not yet written** |
| `FRQ-WFL-15.9` | Generic workflow and the factory that resolves every primitive by name from configuration. | — | **not yet written** |
| `FRQ-MEM-15.10` | Memento — the snapshot a processor resumes from. | — | **not yet written** |
| `FRQ-PTN-15.11`…`15.21` | Pattern infrastructure that is not a domain primitive: root base, orchestrator, scheduler, managers, observable, state machine, iterator, serialisation, singleton, exceptions, `Attribute`/`NameHelper`. | — | **not yet written** |
| `FRQ-PRC-15.22` | Two flows the primitives collaborate in — creation (processor → blackboard → create strategy) and persistence (pipeline → blackboard → repository → write strategy → driver); N repositories mean N write strategies over one document. | [15.22](FRQ-PRC-15.22-document-flow.md) | proposed |

## CON / DRV — connection driver access

Parent requirement: [`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md).
The categories left provisional status with
[`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) (2026-08-27).

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-CON-13.1` | The connection to the HuggingFace sub-system (cache + Hub) exposes only `connect` / `disconnect`; `offline` is a mode of operation. | [FRQ-CON-13.1](FRQ-CON-13.1-huggingface.md) | **implemented** |
| `FRQ-CON-13.2` | A network connection resolves the vendor (endpoint, authentication, dialect) and exposes the same contract as a local one. | [FRQ-CON-13.2](FRQ-CON-13.2-remote-model.md) | proposed |
| `FRQ-DRV-13` | The driver over that connection performs `read` / `write` / `update` / `download` on the model, and the pipeline consumes that service. | [FRQ-DRV-13](FRQ-DRV-13-llm-model.md) | proposed |

## HLRQ-16 — exchange with an SDR device

Parent requirement: [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md).
The role axis categories `CON`, `DRV` and `PRC` are in the vocabulary as of
[`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md); no new category is introduced.

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-CON-16.1` | The connection holds one physical receiver exclusively, addressed by a **selector that must resolve to exactly one device**, probes what the device can do rather than carrying a per-model table, and reads back the *effective* parameters rather than assuming the requested ones. | [16.1](FRQ-CON-16.1-sdr-device.md) | proposed, **not implemented** |
| `FRQ-DRV-16.2` | One driver, `read` and `write`, two protocols beneath (the precedent is the existing message-broker driver): the read path is **a single implementation for every device**, with vendor difference entering only as probed values and the parser the factory returns; `write` exists only where the device reports it. | [16.2](FRQ-DRV-16.2-iq-stream.md) | proposed, **not implemented** |
| `FRQ-PRC-16.3` | The processor owns the capture pass over a source that cannot be paused: it ends on a declared boundary and marks a partial pass as partial. | [16.3](FRQ-PRC-16.3-sdr-capture.md) | proposed, **not implemented** |
| `FRQ-DOC-16.4` | A document carrying a block of samples and its sampling description — **candidate**; no existing document class carries a raw binary buffer alongside one. | — | **not yet written** |
| `FRQ-PIP-16.5` | Pipeline as consumer — **candidate**. Demodulation is a transformation, so it is pipeline work, and its steps are built as **helpers**; which ones and how they compose follows the workflow design and is written with it. | — | **not yet written** |
| `FRQ-PRC-16.6` | The transmit pass — **candidate**; `FRQ-PRC-16.3` covers capture only, and emission is a different pass with a different boundary and its own authorisation (`NFRQ-SEC-07`). | — | **not yet written** |

> The group is written from the device's properties, **not** from existing code: a check on
> 2026-09-09 found no SDR, USB or device component in `blackwattle/src/`. Every entry is a
> proposal, and its acceptance criteria are an input to implementation, not a report (D-05).
>
> **Diagrams carry the structure, prose carries the obligation.** The decomposition, the shared
> read path and the device's direction state are shown as PlantUML views
> ([decomposition](../01-HLRQ/HLRQ-16-sdr-decomposition.puml),
> [read sequence](FRQ-DRV-16.2-read-sequence.puml),
> [device state](FRQ-CON-16.1-device-state.puml)) rather than restated in text — a view, never a
> source of truth (D-13).
>
> **Multi-vendor support is an axis, not a class count.** A class is written when the *contract* or
> the *access mechanism* changes; a difference exhausted by a name, range, unit or count is
> configuration, and the device is **asked** for its capabilities rather than described by a table
> in the repository. Proposed by [`DR-PRC-003`](../04-DR/DR-PRC-003-sdr-access-layer.md), grounded
> in [the vendor-variability analysis](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md).


## OSCAL — machine-checkable compliance

Parent requirement: [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md).
The category is **in the vocabulary** as of
[`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) (2026-08-21); its axis is
capability/distribution, not class role.

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FRQ-OSCAL-14.1` | The catalogue deserialises an OSCAL document into an immutable structure and exposes it as an aggregate of controls (Iterator) and as a tree (Visitor); profile resolution and policy enforcement are not its job. | [14.1](FRQ-OSCAL-14.1-catalog.md) | proposed |
| `FRQ-OSCAL-14.2` | A control carries a mandatory non-empty `id` and `title`, is built recursively, and knows for itself whether it survives profile selection (`pruned`). | [14.2](FRQ-OSCAL-14.2-control.md) | proposed |
| `FRQ-OSCAL-14.3` | A group preserves the catalogue hierarchy; it is not a subject of selection and disappears when no control within it survives. | [14.3](FRQ-OSCAL-14.3-group.md) | proposed |
| `FRQ-OSCAL-14.4` | A profile expresses the baseline as a list of `id`s from the source catalogue; it holds no controls and does not resolve itself. | [14.4](FRQ-OSCAL-14.4-profile.md) | proposed |
| `FRQ-OSCAL-14.5` | Value objects (`Prop`, `Link`, `Part`, `Param`, `Metadata`, `BackMatter`) carry a control's content, without identity and without traversal. | [14.5](FRQ-OSCAL-14.5-value-objects.md) | proposed |
| `FRQ-OSCAL-14.6` | Three bases carry one contract each — deserialisation, identity and traversal, selection — and hold them in one place rather than per class. | [14.6](FRQ-OSCAL-14.6-bases.md) | proposed |
| `FRQ-OSCAL-14.7` | Loading is the only boundary towards disk; the OSCAL document kind is checked in both directions, and the error names the correct loader. | [14.7](FRQ-OSCAL-14.7-loaders.md) | proposed |
| `FRQ-OSCAL-14.8` | The registry gives a flat view over the controls of several catalogues; an `id` collision resolves by "last one wins". | [14.8](FRQ-OSCAL-14.8-registry.md) | proposed |
| `FRQ-OSCAL-14.9` | Resolution turns a profile into a catalogue: pruning belongs to the nodes, and selection, strictness and result assembly to the resolver. | [14.9](FRQ-OSCAL-14.9-resolver.md) | proposed |
| `FRQ-OSCAL-14.10` | The crosswalk translates `id`s into the baseline's taxonomy; an unmapped id passes through unchanged and fails at the policy rather than vanishing. | [14.10](FRQ-OSCAL-14.10-crosswalk.md) | proposed |
| `FRQ-OSCAL-14.11` | The gate checks `declared ⊆ baseline` and rejects a component declaring a control outside the active profile. | [14.11](FRQ-OSCAL-14.11-policy.md) | proposed |
| `FRQ-OSCAL-14.12` | The package ships code and vendored ASD ISM artefacts with a declared public surface and an import closure of `stdlib ∪ wattleflow`. | [14.12](FRQ-OSCAL-14.12-package-surface.md) | proposed |
| `FRQ-OSCAL-14.13` | Three component bases (`OSCALConnection`, `OSCALDriver`, `OSCALProcessor`) carry the decorator instead of every class; the gate is a property of the hierarchy, not of discipline. | [14.13](FRQ-OSCAL-14.13-component-bases.md) | proposed |

## ORG — organisation and structure

> **Declared blind spot (D-11): the category is empty.** The inherited register carries an
> `FRQ-ORG-01` heading with no content. Fill it per ISO/IEC/IEEE 29148 or withdraw it; until
> then phase 3 of `CLAUDE.md` §4 is not closed. Tracked in [`TODO`].

## Open — category vocabulary

The role axis is closed and in the vocabulary as of
[`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md): ten primitives (`WFL`, `PRC`,
`PIP`, `DRV`, `REP`, `BBD`, `STR`, `CON`, `DOC`, `MEM`) plus `PTN`. The capability axis holds
`OSCAL` (`DR-WFL-013`). `ORG` predates both.

**`AUD` and `MAIL` remain provisional** — neither is a primitive nor `concrete/` infrastructure,
and neither has a parent HLRQ yet. `MAIL` was entered without a DR (2026-08-28): the register is
held to be sufficient justification, the `DR` series reserved for exceptions. So does the `HLRQ` class itself (`CLAUDE.md` §3.6). Neither the `FR`→`FRQ`
nor the `NFR`→`NFRQ` rename of 2026-08-27 has a decision record; both are tracked in
[`TODO`](../workflow/TODO.md).
