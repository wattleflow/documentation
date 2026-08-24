# Wattleflow Engineering Philosophy

## Towards a scientifically grounded methodology of software engineering

**Version:** Draft v0.4.2
**Status:** living document. This is the English edition; the Croatian source is
`workflow/hr/FILOZOFIJA.md`, and the two have diverged — a declared gap (D-11), not an
approved translation. `CLAUDE.md` §3.2 makes the Croatian text authoritative until v1.0.
**Source language:** Croatian

> *“Architecture is not software documentation. Architecture is the scientific foundation from which software follows.”*

---

## Foreword

Wattleflow grew out of practice, under conditions where time and immediate needs set the pace. This philosophy marks a deliberate turn: from implementation-led development towards scientifically grounded, multidisciplinary engineering.

The turn is not a change of direction but a return to the original intent — to a founding thesis from information science that remains the basis of the doctrine:

> **Information is a value bearing meaning and context, and its protection is a structural precondition of a reliable system — not an add-on.**

## Summary

If information carries meaning and context, then a system that handles it should not leave that meaning implicit. The whole doctrine follows from this: meaning is declared before it is implemented, and reliability and protection are built into the structure rather than added at the end.

The guiding hypothesis is this: **architecture does not arise as a by-product of implementation.** Architecture rests on engineering principles, and those principles follow from a grounded scientific approach and professional knowledge. The alternative is familiar and measurably expensive — architectural drift, mounting technical debt, and a system whose reasons no one can reconstruct.

Iteration between architecture and implementation is therefore not ruled out; it is expected. It takes place, however, within the governance cascade described in *Doctrine*: implementation experience revises architecture through a documented decision (a decision record — DR), not through a silent change to the code. Silent change is a problem not because of formalism, but because after it the system loses its own rationale.

The document is accordingly an **anti-manifesto**: a manifesto declares values without a burden of proof; an anti-manifesto declares the burden of proof to be the value. Two things are made explicit here and binding on all other documentation:

1. **Philosophy is the umbrella** over development — the starting point from which policies, principles and methods are derived (section *Doctrine*).
2. **Critical thinking is a foundational doctrine** that Wattleflow not only adopts but propagates — building it into how work is done rather than leaving it to chance (section *Critical thinking*).

The guiding operational principle is **measurability**: a load-bearing doctrinal claim comes in falsifiable form, or it remains unverified (section *Hypotheses*). This applies to the premises of this document as well.

Architecture and functionality are derived from requirements that live in [`02-FRQ/`](02-FRQ/0-FRQ-EN.md) (functional, with [`01-HLRQ/`](01-HLRQ/0-HLRQ-EN.md) above them) and [`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md) (non-functional); the foundations of the methodology are set out in Volume I ([`METHODOLOGY.md`](METHODOLOGY.md)).

## Contents (volumes)

| Volume | Title | Location |
|---|---|---|
| I | Foundations | [METHODOLOGY.md](METHODOLOGY.md) |
| II | Architectural principles | *(planned)* |
| III | Decision Records | `workflow/dr/` (WFL) · `workflow/core/dr/` (COR) · `04-DR/` (PRC) *(active)* |
| IV | System design | *(planned)* |
| V | Development standards and functionality | [`02-FRQ/`](02-FRQ/0-FRQ-EN.md), [`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md) *(active)* |
| VI | Quality assurance | *(planned)* |
| VII | AI-assisted software engineering | *(planned)* |
| VIII | Governance, compliance and information science | *(planned)* |

---

## Motivation

Modern software development often suffers from an inversion of priorities. A typical project runs like this:

```
Business requirement
      ↓
User story
      ↓
Implementation
      ↓
Architecture  (reconstructed afterwards, from the code)
```

In such a sequence architecture accumulates rather than emerges — a sediment of implementation decisions that no one made explicitly. The consequences are familiar: architectural drift, inconsistent design, mounting technical debt, poor maintainability, reduced explainability and limited educational value.

This inversion is not a historical necessity. Iterative and incremental development predates the agile movement by decades; the original “waterfall” model was explicitly iterative and warned that a single one-way pass “invites failure” (P-15). The feedback loop as a scientific instrument comes from statistical process control, where it turns on *theory* — hypothesis → measurement → conclusion — and not on a backlog (P-01). Experience without theory teaches nothing (P-02).

Wattleflow therefore does not discard iteration; it restores its scientific content. Scientific and engineering principles, together with business objectives, enter as **inputs**; requirements are **derived** from them; measurements close the loop:

```
Scientific principles + Engineering principles + Business objectives
      ↓
Functional and non-functional requirements  (02-FRQ, 03-NFRQ)
      ↓
Architecture  ⇄  Implementation   (iteration under the cascade; revision through DR)
      ↓
Testing + Conformance (wem_lint) + Operational metrics
      ↓
Empirical evidence → returns into the principles
```

Architecture and documentation thereby become first-class engineering artefacts and **epistemological material**: a record of knowledge, not a work diary (P-14). Business objectives are an input that shapes requirements — not an artefact slipped in underneath them afterwards. The full traceability model and the requirements ontology are set out in Volume I (§6, §4).

---

## Doctrine: philosophy as the umbrella

Philosophy stands at the head of the development cycle — the umbrella under which policies, principles and methods take shape. It is not a governing body but a body of knowledge: it brings together an ontological position and epistemological material, so that everything else has somewhere to follow from.

Its discourse is **epistemological** by nature: it concerns which knowledge we treat as grounded, and how we acquire, test and revise it.

From that discourse comes an **ontological perspective** — what *exists* in the domain and in the requirements space (domain ontology: Workflow, Document, Strategy…; requirements ontology: business requirement, rule, FR/NFR, constraint). Ontology is not a foundation *beneath* philosophy but its **product**: philosophical discourse settles which entities and relations exist, and on what criterion that knowledge is defended.

```
      Philosophy      (umbrella — epistemological discourse)
           │
           ├──────────►  Ontological perspective  (what exists: domain + requirements)
           │
           ▼
       Policies       (binding rules — CLAUDE.md, security policy)
           │
           ▼
       Principles     (engineering and scientific principles — Parnas, SOLID, DRY, …)
           │
           ▼
       Methods        (methodologies, standards, tools — PEP, ISO, SBOM, wem_lint)
```

**Direction of justification.** The cascade does not describe authority but the provenance of reasons: a method is justified by a principle, a principle by a policy, a policy by philosophy. Where something is adopted without that trail, no prohibition has been broken — a reason is simply missing, and a decision without a reason can later be neither verified nor reversed.

Departures are therefore expected, and take two legitimate forms: a derivation from a higher layer, or a **documented revision** of that layer through a decision record (DR). The difference between them is epistemic rather than formal — the first applies existing knowledge, the second changes it, and the distinction is worth keeping.

The cascade thus works in both directions: from above it supplies reasons, from below evidence corrects them. An example: instead of a bespoke digest registry, the wheel RECORD/SBOM mechanism was adopted — new understanding of an existing standard refined the method, with traceability preserved to the Occam/DRY principle and to the epistemic core of the philosophy.

**The epistemic core of trust.** At the philosophical level: *trust is not assumed but demonstrated* — a special case of the general doctrine that a claim without testimony does not bind. The named security policy that follows from it (zero-trust architecture, trust boundaries, supply-chain integrity) lives one layer down, among the policies, and is revised like any policy — through a DR. Philosophy thereby prescribes no security technology, only the epistemic standard that any security technology should meet.

## Critical thinking — a foundational doctrine

The engine of epistemological discourse is critical thinking: systematically questioning assumptions, asking for justification, and standing ready to revise in the face of evidence. The notion is well established in science; it is stated explicitly here because Wattleflow not only adopts it but **propagates** it — building it into how work is done. Critical thinking is not a stance but an **enforceable practice**:

- A decision does not appeal to authority or habit; it survives judgement — a scientific principle, a recognised standard, mathematical reasoning, or empirical evidence. **Personal preference is not a justification.**
- **The DR is the artefact of critical thinking** [44][45][46] — it records context, alternatives considered, cost and testimony, so the decision remains verifiable and reversible. A decision citing no testimony is declared an aspiration, not a proven result.
- The direction of justification (“change only through documented revision”) is critical thinking **institutionalised** — a guard against drift by inertia.
- It applies equally to people and to AI agents: where there is doubt, ask, set out cost and alternatives, and do not guess (`POLICY.md` §5; Volume I §9).
- It applies to the philosophy itself: a doctrinal claim that does not say what would refute it is leaning on authority — precisely what this doctrine rejects. Every load-bearing claim therefore comes in falsifiable form (next section).

**Knowledge grows — and that growth drives the discourse.** Theoretical knowledge is not static: through epistemological analysis and new findings it widens, re-enters the philosophical discourse, and reshapes policies, principles and methods down the cascade. This is the operational form of the feedback loop from *Motivation*: measurements and new insights are not the end of the chain but its next input. So that the growth of knowledge stays traceable rather than arbitrary, each such shift is recorded (DR or FR/NFR) with its rationale.

## Hypotheses

The doctrine of measurability applies first to the philosophy itself. Load-bearing claims take falsifiable form; each states a measure and the testimony that would refute it. Measurements live in conformance analysis (wem_lint, C-snapshots) and in the work on quantifying the architectural boundary.

**H1 — Architecture-first under governance reduces architectural drift.**
*Measure:* agreement between the declared and the detected partition of the system (normalised mutual information), tracked across versions.
*Refuted by:* a systematic fall in agreement despite the cascade being applied, or equal agreement in comparable systems without governance.

**H2 — An enforced semiotic registry reduces the semantic entropy of a system.**
A controlled vocabulary (the naming registry, ORG-02/03) acts as a semiotic stabiliser (P-08): it fixes the relation between sign and role.
*Measure:* the trend in naming violations per version; the number of signs carrying multiple semantics (baseline: a single TypeVar `T` serving four roles, recorded in the core analysis).
*Refuted by:* stagnation or growth in ambiguity despite the registry being enforced.

**H3 — A versioned criterion preserves the validity of the instrument.**
A measuring instrument corrodes, and does so quietly (P-10); versioning the criterion (registry + tool + platform) makes the divergence between proxy and construct visible.
*Measure:* detection latency for validity incidents (baseline: incident ABS-03 detected within two measurement cycles, not years).
*Refuted by:* a validity incident that the versioned system misses for longer than an unversioned benchmark, or findings from two criterion versions that cannot be reconciled by the declared notes.

**H4-DQI is a candidate, not a register entry.** The data-quality index states its own
hypothesis in [`05-METHOD/dqi.md`](05-METHOD/dqi.md) and declares there that it is not held
here; until it is admitted through a DR it binds nothing (D-05, D-11).

The list is open. A new doctrinal claim enters the document with its hypothesis — or with an explicit note that it is conceptual rather than empirical.

---

## Beyond software engineering

Large information systems live at the intersection of **computer science**, **software engineering** and **information science**. Wattleflow deliberately integrates all three, with emphasis on the third — historically pushed out of development practice, though it underpins it:

- **information theory** supplies a measure of uncertainty and the limits of transmission (P-06),
- **cybernetics** supplies the feedback loop, regulation and the law of requisite variety (P-05),
- **semiotics** supplies the relation of sign and meaning in an organisational context (P-08),
- **the infological tradition** supplies the distinction between data and information — information is a function of data, prior knowledge and time of interpretation (P-07).

Each discipline and its contribution are set out in Volume I (§3).

```
              Computer science
                      ▲
                      │
 Information  ◄───────┼───────►  Software
   science            │        engineering
                      ▼
                 Wattleflow
```

## Ontology before implementation

Implementation follows ontology, not the other way round. Wattleflow defines an explicit **domain ontology** (Workflow, Document, Strategy, Driver, Repository, Connection, Processor, Pipeline, Blackboard, Memento) and a separate **requirements ontology**. Both are set out in Volume I (§4), and both are, under the Doctrine, products of epistemological discourse: they change when knowledge changes, through a DR.

## Scientific foundations

Every significant architectural decision is traceable to grounded knowledge:

- **design theory** — Parnas [1], SOLID, GRASP, GoF, Dijkstra, Law of Demeter, Stable Dependencies/Abstractions,
- **engineering principles** — KISS, DRY, YAGNI, Open–Closed, Liskov, Principle of Least Astonishment, Occam’s razor,
- **systems theory** — Simon [2], Conway, Gall, Lehman, Brooks [29],
- **measurement theory** — the representation condition and scale types (P-09).

The catalogue and its application are in Volume I (§5); load-bearing claims, with their provenance and keys, are held in the postulate registry (`POSTULATE.md`).

## Architecture as an educational artefact

The purpose of architecture is not only to produce software but to convey engineering knowledge: why components exist, why responsibilities are separated, why dependencies flow one way, why naming conventions matter, and why these particular patterns were chosen.

Architecture is therefore also an educational resource. Documentation, meanwhile, is a system with several audiences — architect, implementer, tester, security analyst, integrator, business user, auditor — each of which needs its own artefact. A single format for all audiences violates the law of requisite variety (P-05).

More deeply: architecture is the externalised part of the theory a team holds about the system, so its educational value lies precisely in transmitting that theory (P-16).

## Architectural self-description

Architecture is intelligible without reading the implementation: a class name, its inheritance, its place in the package and its dependencies communicate layer, domain, responsibility and life cycle.

Names are accordingly **domain-qualified and role-explicit**. Bare generic nouns (`Manager`, `Helper`, `Parser`, `Piece`, `Sheet`) do not pass — qualified forms (`DriverS3UriParser`, `SheetNestingPlacement`) do. This is a semiotic policy (P-08): a name is a sign whose relation to its role the registry fixes and the linter enforces. The enforceable grammar is [`NFR-ORG-02`](03-NFRQ/NFR-ORG-02-class-nomenclature-EN.md); the rationale is in Volume I §8.

## AI-assisted software engineering

Large language models introduce a new consumer of architecture. Historically architecture served people; today it also supports machine reasoning — generating documentation, reverse engineering, dependency analysis, validation, refactoring and code generation.

Semantic consistency serves both, and the doctrine of critical thinking holds for both: where there is doubt, ask, set out cost and alternatives — do not guess. Elaborated in Volume I §9.

## Decision Records (DR)

The practice of decision records (legacy name: ADR) has a clear genealogy: the decision as a first-class object of architecture and the “evaporation of knowledge” when decisions go unrecorded [45]; a rich record template [44]; and the minimal format that made the practice sustainable [46].

Wattleflow adopts an extended template — **Status / Context / Decision / Contract / Cost / Testimony / Registry** — the richness of the Tyree–Akerman line with a discipline of cost: fields exist because a tool and a process consume them, not for completeness. The *Testimony* field separates the proven from the aspirational; the *Registry* field ties the decision to a machine-checkable criterion, at which point the record stops being merely a record and becomes a source of conformance rules.

Within Wattleflow, DRs serve three complementary aims:

- **Business alignment** — decisions remain traceable to business objectives.
- **Architectural governance** — consistency across long-term evolution.
- **Scientific justification** — a substantial decision rests on recognised principles, theories, mathematical reasoning or standards.

**Numbering:** a per-project series with a prefix — `DR-COR` (core), `DR-WFL` (workflow), `DR-PRC` (processors), `DR-CAD` (cad). The identifier is thereby globally unique without a central counter, and a decision belongs to the series of whichever project’s artefact it changes.

The record format itself is a replaceable sub-method of the doctrinal function of recording decisions (Volume I, §7.1): the function is obligatory, the form is revised by evidence. Elaborated in Volume III.

## Functional requirements

ISO/IEC/IEEE 29148 defines functional requirements as those describing system behaviour, the services it provides, its inputs and outputs, and its responses to external events. In Wattleflow that idea lives in [`02-FRQ/`](02-FRQ/0-FRQ-EN.md), where functional requirements are given unique identifiers, acceptance criteria, and traceability to business objectives and architecture.

The main disciplines from 29148 that this document follows:

- **Stakeholder needs and expectations** — functional requirements start from stakeholder interests and business objectives, not from technology.
- **Elicitation and analysis** — requirements are gathered, analysed, clarified and prioritised ahead of architectural decisions.
- **Documentation** — each FR carries a unique ID, description, rationale, acceptance criteria and traceability references, consistent with the index in `02-FRQ/`.
- **Verification and validation** — requirements are verified as correct, complete and verifiable, and validated against real user needs.
- **Traceability and requirements management** — changes are tracked and mapped across the full requirements life cycle.

This approach bears out the document’s decision: functionality is held in a separate register, and architecture follows it as derived — not the reverse.

## Non-functional requirements

Architectural decisions generate **measurable** non-functional requirements: each anchored to ISO/IEC 25010, identified as `NFR-<CATEGORY>-NN`, and enforced through coding standards, architectural reviews, static analysis and CI/CD controls.

The conformance measure is vectorial — by dimension, with no scalar overall score. The reason is metric rather than stylistic: a weighted sum across nominal and ordinal scales is not a defined operation (P-09), so a single “system score” would be a number without meaning. The register is held in [`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md).

## Standards compliance

Wattleflow does not replace standards but integrates them: PEP, ISO/IEC/IEEE 42010, ISO/IEC 25010/25012/12207, OMG SBVR and OCL, NIST OSCAL, W3C RDF/OWL/PROV-O, and (planned) SPDX/CycloneDX. A standard enters through epistemic judgement, not by inertia; what each one covers is set out in Volume I (§7).

## Vision

The long-term aim of Wattleflow is not another workflow framework, but to show that modern information systems can be built with a methodology that joins scientific reasoning, software engineering, information science, internationally recognised standards and practical implementation.

The framework that comes out of this should be technically rigorous, instructive, explainable, maintainable, compliant, AI-compatible and architecturally self-describing.

In this philosophy, architecture is not software documentation.

**Architecture is the scientific foundation from which software follows.**

---

## References

This document keeps no bibliography of its own (P-20: a rendering is not the source of truth). Keys [n] point to the consolidated table in `documentation/LITERATURE.md`; P-NN markers point to the postulate registry `documentation/POSTULATE.md` (claim with provenance, application and key).

Citation standard: numeric keys (IEEE style), unique across all doctrinal documents and the paper, under an **append-only** rule — a new key goes only at the end of the table, since insertion would renumber citations across every document. The evolutionary path towards stable identifiers (`literatura.yaml`, with numbers as a generated rendering) is recorded as a DR candidate.