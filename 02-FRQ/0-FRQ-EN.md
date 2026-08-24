# FRQ — functional requirements register

| | |
|---|---|
| **Identifier** | `FR-<CATEGORY>-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Anchor** | ISO/IEC/IEEE 29148 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/0-HLRQ-EN.md) · [NFRQ](../03-NFRQ/0-NFRQ-EN.md) |
| **Language** | Index EN; the entries are Croatian (source, `CLAUDE.md` §3.2) |

This document is an **index**, not a register: each requirement lives in its own file
(`category-number-slug.md`). The register holds **an identifier, one statement and a link**; the
detail is not restated here (D-13).

Every FR carries acceptance criteria, a verification method and traceability upwards. A
requirement is either **primary** (derived from a principle, constraining decisions) or
**derived** (arising from a decision); traceability is a **graph, not a chain**
(`METHODOLOGY.md` §6).

**Common definitions** (domain, helper, canonical subject) live in
[`NFR-DEF-01`](../03-NFRQ/NFR-DEF-01-common-definitions-EN.md) — one register, everything else
references it (D-12).

## AUD — audit record

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FR-AUD-01` | The audit record travels structured to every subscribed handler, and a handler passes through only the fields its declared redaction capability covers. | [FR-Handlers](../workflow/concrete/FR-Handlers.md) | proposed |

## CON / DRV — language-model access

Parent requirement: [`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md).

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FR-CON-13.1` | The connection to the HuggingFace sub-system (cache + Hub) exposes only `connect` / `disconnect`; `offline` is a mode of operation. | [FR-CON-13.1](FRQ-CON-13.1-huggingface-connection.md) | **implemented** |
| `FR-CON-13.2` | A network connection resolves the vendor (endpoint, authentication, dialect) and exposes the same contract as a local one. | [FR-CON-13.2](FRQ-CON-13.2-remote-model-connection.md) | proposed |
| `FR-DRV-13` | The driver over that connection performs `read` / `write` / `update` / `download` on the model, and the pipeline consumes that service. | [FR-DRV-13](FRQ-DRV-13-llm-model.md) | proposed |

## OSCAL — machine-checkable compliance

Parent requirement: [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md).
The category is **in the vocabulary** as of
[`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) (2026-08-21); its axis is
capability/distribution, not class role.

| Id | Statement | Detail | Status |
|---|---|---|---|
| `FR-OSCAL-14.1` | The catalogue deserialises an OSCAL document into an immutable structure and exposes it as an aggregate of controls (Iterator) and as a tree (Visitor); profile resolution and policy enforcement are not its job. | [14.1](FRQ-OSCAL-14.1-catalog.md) | proposed |
| `FR-OSCAL-14.2` | A control carries a mandatory non-empty `id` and `title`, is built recursively, and knows for itself whether it survives profile selection (`pruned`). | [14.2](FRQ-OSCAL-14.2-control.md) | proposed |
| `FR-OSCAL-14.3` | A group preserves the catalogue hierarchy; it is not a subject of selection and disappears when no control within it survives. | [14.3](FRQ-OSCAL-14.3-group.md) | proposed |
| `FR-OSCAL-14.4` | A profile expresses the baseline as a list of `id`s from the source catalogue; it holds no controls and does not resolve itself. | [14.4](FRQ-OSCAL-14.4-profile.md) | proposed |
| `FR-OSCAL-14.5` | Value objects (`Prop`, `Link`, `Part`, `Param`, `Metadata`, `BackMatter`) carry a control's content, without identity and without traversal. | [14.5](FRQ-OSCAL-14.5-value-objects.md) | proposed |
| `FR-OSCAL-14.6` | Three bases carry one contract each — deserialisation, identity and traversal, selection — and hold them in one place rather than per class. | [14.6](FRQ-OSCAL-14.6-bases.md) | proposed |
| `FR-OSCAL-14.7` | Loading is the only boundary towards disk; the OSCAL document kind is checked in both directions, and the error names the correct loader. | [14.7](FRQ-OSCAL-14.7-loaders.md) | proposed |
| `FR-OSCAL-14.8` | The registry gives a flat view over the controls of several catalogues; an `id` collision resolves by "last one wins". | [14.8](FRQ-OSCAL-14.8-registry.md) | proposed |
| `FR-OSCAL-14.9` | Resolution turns a profile into a catalogue: pruning belongs to the nodes, and selection, strictness and result assembly to the resolver. | [14.9](FRQ-OSCAL-14.9-resolver.md) | proposed |
| `FR-OSCAL-14.10` | The crosswalk translates `id`s into the baseline's taxonomy; an unmapped id passes through unchanged and fails at the policy rather than vanishing. | [14.10](FRQ-OSCAL-14.10-crosswalk.md) | proposed |
| `FR-OSCAL-14.11` | The gate checks `declared ⊆ baseline` and rejects a component declaring a control outside the active profile. | [14.11](FRQ-OSCAL-14.11-policy.md) | proposed |
| `FR-OSCAL-14.12` | The package ships code and vendored ASD ISM artefacts with a declared public surface and an import closure of `stdlib ∪ wattleflow`. | [14.12](FRQ-OSCAL-14.12-package-surface.md) | proposed |
| `FR-OSCAL-14.13` | Three component bases (`OSCALConnection`, `OSCALDriver`, `OSCALProcessor`) carry the decorator instead of every class; the gate is a property of the hierarchy, not of discipline. | [14.13](FRQ-OSCAL-14.13-component-bases.md) | proposed |

## ORG — organisation and structure

> **Declared blind spot (D-11): the category is empty.** The inherited register carries an
> `FR-ORG-01` heading with no content. Fill it per ISO/IEC/IEEE 29148 or withdraw it; until
> then phase 3 of `CLAUDE.md` §4 is not closed. Tracked in
> [`workflow/TODO.md`](../workflow/TODO.md).

## Open — category vocabulary

The categories **`AUD`, `CON` and `DRV` are not in the vocabulary**; introducing them requires a
DR (D-12). Until then those identifiers are **provisional**, as is the `HLRQ` class itself. The
only confirmed category outside `ORG` is `OSCAL` (`DR-WFL-013`).

Most entries carry the status **proposed**; status changes through a DR, never tacitly (D-03).
