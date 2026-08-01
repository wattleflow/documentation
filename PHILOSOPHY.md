# The Wattleflow Engineering Philosophy

## Towards a Scientifically Grounded Software Engineering Methodology

**Version:** Draft v0.4.1 (EN)
**Status:** living document; translation of the Croatian source (FILOZOFIJA.md v0.4.1). Until v1.0, the Croatian text remains authoritative and this English edition is a maintained view of it (D-13); discrepancies are resolved in favour of the source.
**Source language:** Croatian

> *"Architecture is not the documentation of software. Architecture is the scientific foundation from which software arises."*

---

## Foreword

Wattleflow began in practice, as the work of a single author, where shortage
of time and practical needs dictated both pace and approach. This philosophy
marks a deliberate turn: from implementation-driven development towards
scientifically grounded, multidisciplinary engineering. This is not an
ambition added after the fact but the original intent, as witnessed by the
author's earlier work — in particular the research in the information
sciences (RESEARCH.md): the thesis that information is value with meaning
and context, and that its protection is a structural precondition of a
trustworthy system, not an add-on.

## Summary

This document establishes the engineering philosophy on which Wattleflow
rests. The aim is not to document a series of decisions, but to set out the
rigour and methodology for designing information systems, grounded in
scientific principles, internationally recognised standards, and the
practical needs of engineering modern, AI-assisted systems.

The guiding hypothesis is: **architecture must not arise as a by-product of
implementation.** Architecture stands on engineering principles, and those
principles derive from a well-founded scientific approach and professional
knowledge. Iteration between architecture and implementation is permitted —
indeed expected — but only within the governance cascade described in the
Doctrine: implementation experience may revise architecture solely through a
documented decision (a decision record, DR), never through the silent
accretion of code.

This document is therefore an **anti-manifesto**: a manifesto declares
values without the burden of proof; an anti-manifesto declares the burden of
proof as a value. Two things this document makes explicit and binding on all
other documentation:

1. **The philosophy is the umbrella** over development — the highest
   instance from which policies, principles and methods derive (section
   *Doctrine*).
2. **Critical thinking is a foundational doctrine** which Wattleflow not
   only adopts but propagates — building it into the way of working rather
   than leaving it to chance (section *Critical Thinking*).

The guiding operational principle is **measurability**: every doctrinal
claim must have a falsifiable form (section *Hypotheses*). Architecture and
functionality are derived from requirements that live in FR.md (functional)
and NFR.md (non-functional); the foundations of the methodology are set out
in Volume I (METHODOLOGY.md).

## Contents (volumes)

| Volume | Title | Location |
|---|---|---|
| I | Foundations | METHODOLOGY.md |
| II | Architectural Principles | *(planned)* |
| III | Decision Records | docs/adr/ *(active; name migration in progress, DR-016)* |
| IV | System Design | *(planned)* |
| V | Development Standards | NFR.md *(in progress)* |
| VI | Quality Assurance | *(planned)* |
| VII | AI-Assisted Software Engineering | *(planned)* |
| VIII | Governance, Compliance and Information Science | *(planned)* |

---

## Motivation

Modern software development frequently suffers from an inversion of
priorities. A typical project proceeds in this order:

```
Business requirement
      ↓
User story
      ↓
Implementation
      ↓
Architecture  (reconstructed afterwards, from the code)
```

In this sequence, architecture arises by accretion — as the sediment of
implementation decisions that nobody took explicitly. The consequences are
familiar: architectural drift, inconsistent design, growing technical debt,
poor maintainability, reduced explainability, and limited educational value.

This inversion is not a historical necessity. Iterative and incremental
development predates the agile movement by decades; the original "waterfall"
model was explicitly iterative and warned that a single-pass sequence
"invites failure" (P-15); and the feedback loop as a scientific instrument
originates in statistical process control — where it turns over *theory*
(hypothesis → measurement → conclusion), not over a backlog (P-01).
Experience without theory teaches nothing (P-02). Wattleflow therefore does
not reject iteration; it restores its scientific content.

Wattleflow adopts scientific and engineering principles and business goals
as **inputs**; requirements are **derived** from them; measurements close
the loop:

```
Scientific principles + Engineering principles + Business goals
      ↓
Functional and non-functional requirements  (FR.md, NFR.md)
      ↓
Architecture  ⇄  Implementation   (iteration under the cascade; revision only through a DR)
      ↓
Testing + Conformance (wem_lint) + Operational metrics
      ↓
Empirical evidence → feeds back into the principles
```

Architecture and documentation thereby become first-class engineering
artefacts and **epistemological material**: a record of knowledge, not a
diary of work (P-14). Business goals are an input that shapes requirements —
not an artefact slipped in beneath them after the fact. The full
traceability model and the requirements ontology are set out in Volume I
(§4.1, §9).

---

## Doctrine: the philosophy as umbrella

The philosophy is the highest instance of the development cycle — the
umbrella from which policies, principles and methods derive. Its discourse
is by nature **epistemological**: it concerns which knowledge we hold to be
well-founded, and how we acquire, test and revise it.

From the epistemological discourse follows the **ontological perspective** —
what *exists* in the domain and in the requirements space (the domain
ontology: Workflow, Document, Strategy, …; the requirements ontology:
business requirement, business rule, FR/NFR, constraint). Ontology is not a
foundation *beneath* the philosophy but its **product**: the philosophical
discourse decides which entities and relations exist and by what criterion
that knowledge is defended.

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

**Rule of subordination:** a lower layer never overrides a higher one. A
policy is not introduced past the philosophy, a principle is not chosen past
the policies, a method is not adopted past the principles. Every deviation
is either a derivation from a higher layer or a **documented revision** of
that layer — and the only lawful form of revision is a decision record
(DR). The cascade operates in both directions: it prescribes from above and
is revised from below by evidence (example: adopting the wheel RECORD/SBOM
mechanism instead of a home-grown digest registry — new knowledge of an
existing standard refined the method, while preserving traceability to the
Occam/DRY principles and to the epistemic core of the philosophy).

**The epistemic core of trust.** At the level of the philosophy the rule
is: *trust is not presumed but proven* — a special case of the general
doctrine that a claim without evidence does not bind. The named security
policy that follows from it (zero-trust architecture, trust boundaries,
supply-chain integrity) lives one layer below, among the policies, and is
revised like any policy — through a DR. The philosophy thus prescribes not
a security technology but the epistemic standard every security technology
must satisfy.

## Critical thinking — a foundational doctrine

The engine of the epistemological discourse is critical thinking: the
systematic questioning of assumptions, the demand for justification, and
the readiness to revise in the light of evidence. The concept is well known
in science; it is stated explicitly here because Wattleflow not only adopts
it but **propagates** it — builds it into the way of working. Critical
thinking is not an attitude but an **enforceable practice**:

* No decision appeals to authority or habit — it must survive judgement:
  a scientific principle, a recognised standard, mathematical reasoning, or
  empirical evidence. **Personal preference is not a justification.**
* **The DR is an artefact of critical thinking** [44][45][46] — it records
  context, alternatives considered, cost and evidence, making the decision
  verifiable and revocable. A decision that cites no evidence is declared
  an aspiration, not proven.
* The rule of subordination ("change only through documented revision") is
  critical thinking **institutionalised** — it prevents drift by inertia.
* It applies equally to humans and to AI agents: in doubt, ask, expose the
  cost and the alternatives — do not guess (POLICY.md §5; Volume I §8).
* It applies to the philosophy itself: a doctrinal claim that does not
  state what would refute it is relying on authority — which this doctrine
  forbids. Every load-bearing claim therefore has a falsifiable form (next
  section).

**Knowledge grows — and that growth drives the discourse.** Theoretical
knowledge is not static: through epistemological analysis and new findings
it expands, re-enters the philosophical discourse, and cascades into
reshaped policies, principles and methods. This is the operational form of
the feedback loop in the Motivation: measurements and new insights are not
the end of the chain but its new input. The doctrine requires every such
shift to be recorded (DR/NFR) with justification — so that the growth of
knowledge remains traceable, not arbitrary.

## Hypotheses

The doctrine of measurability is applied first to the philosophy itself.
Load-bearing claims have a falsifiable form; each states its measure and
the evidence that would refute it. The measurements live in the conformance
analysis (wem_lint, C-snapshots) and in the paper on quantifying the
architectural boundary.

**H1 — Architecture-first under governance reduces architectural drift.**
*Measure:* agreement between the declared and the detected partition of the
system (normalised mutual information) tracked across versions.
*Refuted by:* a systematic fall in agreement despite enforcement of the
cascade, or equal agreement in comparable ungoverned systems.

**H2 — An enforced semiotic registry reduces the semantic entropy of the
system.** A controlled vocabulary (the naming registry, ORG-02/03) is a
semiotic stabiliser (P-08): it fixes the relation between sign and role.
*Measure:* the trend of naming violations per version; the number of signs
with multiple semantics (baseline: a single TypeVar `T` bearing four roles,
recorded in the core analysis).
*Refuted by:* stagnation or growth of ambiguity under an enforced registry.

**H3 — A versioned criterion preserves the validity of the instrument.** A
measuring instrument corrodes silently (P-10); versioning the criterion
(registry + tool + platform) makes the divergence of proxy and construct
visible.
*Measure:* the latency of detecting validity incidents (baseline: the
ABS-03 incident, detected within two measurement cycles rather than years).
*Refuted by:* a validity incident that the versioned system misses for
longer than an unversioned benchmark, or findings of two criterion versions
that cannot be reconciled by declared notes.

The list is not closed: a new doctrinal claim enters the document only with
its accompanying hypothesis, or with an explicit declaration that it is
conceptual rather than empirical.

---

## Beyond software engineering

Large information systems exist at the intersection of **computer
science**, **software engineering** and **information science**. Wattleflow
deliberately integrates all three — with particular emphasis on the third,
historically pushed out of development practice although it is foundational
to it: information theory provides the measure of uncertainty and the
limits of transmission (P-06); cybernetics provides the feedback loop and
regulation, and the law of requisite variety (P-05); semiotics provides the
relation of sign and meaning in an organisational context (P-08); and the
infological tradition provides the distinction between data and
information — information is a function of the data, the recipient's prior
knowledge, and the time of interpretation (P-07). Each discipline and its
contribution are elaborated in Volume I (§3).

```
                Computer science
                      ▲
                      │
Information  ◄────────┼────────►  Software
  science             │          engineering
                      ▼
                 Wattleflow
```

## Ontology before implementation

Implementation follows ontology, not the other way round. Wattleflow
defines an explicit **domain ontology** (Workflow, Document, Strategy,
Driver, Repository, Connection, Processor, Pipeline, Blackboard, Memento)
and a separate **requirements ontology**. Both are elaborated in Volume I
(§4, §4.1) — and both are, by the Doctrine, products of the epistemological
discourse: they change when knowledge changes, through a DR.

## Scientific foundations

Every significant architectural decision is traceable to well-founded
knowledge: design theory (Parnas [1], SOLID, GRASP, GoF, Dijkstra, the Law
of Demeter, Stable Dependencies/Abstractions), engineering principles
(KISS, DRY, YAGNI, Open–Closed, Liskov, the Principle of Least
Astonishment, Occam's razor), systems theory (Simon [2], Conway, Gall,
Lehman, Brooks [29]) and measurement theory (the representational condition
and scale types, P-09). The catalogue and its application are in Volume I
(§5); the load-bearing claims, with their origins and reference keys, are
kept in the register of postulates (documents/POSTULATI.md).

## Architecture as an educational artefact

The purpose of architecture is not only to produce software but to transmit
engineering knowledge: why components exist, why responsibilities are
separated, why dependencies flow in one direction, why naming conventions
matter, and why particular patterns were chosen. Architecture is therefore
an educational resource (Volume I §6) — and documentation is a system with
multiple audiences (architect, implementer, tester, security analyst,
integrator, business user, auditor), each of which needs its own artefact:
one format for all audiences violates the law of requisite variety (P-05).
Deeper still: architecture is the externalised part of the theory a team
holds about its system, so the educational value of architecture is
precisely the transmission of that theory (P-16).

## Architectural self-description

Architecture must be intelligible without reading the implementation: a
class's name, its inheritance, its place in the package and its
dependencies communicate layer, domain, responsibility and life cycle.
Names must be **domain-qualified and role-explicit** — bare generic nouns
(`Manager`, `Helper`, `Parser`, `Piece`, `Sheet`) are forbidden; qualified
forms (`DriverS3UriParser`, `SheetNestingPlacement`) are correct. This is a
semiotic policy (P-08): a name is a sign whose relation to its role the
registry fixes and the lint enforces. The enforceable grammar is NFR-ORG-02
(NFR.md); the rationale is in Volume I §7.

## AI-assisted software engineering

Large language models introduce a new consumer of architecture.
Historically, architecture served humans; today it must also support
machine reasoning — documentation generation, reverse engineering,
dependency analysis, validation, refactoring and code generation. Semantic
consistency serves both humans and machines; the doctrine of critical
thinking applies to both (in doubt: ask, expose the cost and the
alternatives — do not guess). Elaborated in Volume I §8.

## Decision Records (DR)

The practice of recording decisions (legacy name: ADR) has a clear
genealogy: the decision as a first-class object of architecture and the
"vaporisation of knowledge" when decisions go unrecorded [45]; the rich
record template [44]; and the minimal format that made the practice
sustainable [46]. Wattleflow adopts an extended template (Status / Context
/ Decision / Contract / Cost / Evidence / Registry) — the richness of the
Tyree–Akerman line with a discipline of cost: fields exist because the tool
and the process consume them, not for completeness. The *Evidence* field
separates the proven from the aspirational; the *Registry* field binds the
decision to a machine-checkable criterion, whereby the decision record
ceases to be a mere note and becomes a source of conformance rules.

Within Wattleflow, DRs serve four complementary goals:

* **Business alignment** — decisions remain traceable to business goals.
* **Reverse engineering** — the architecture can be reconstructed from the
  record.
* **Architectural governance** — consistency through long-term evolution.
* **Scientific justification** — every major decision is justified by
  recognised principles, theories, mathematical reasoning or standards.

Numbering: sequential (DR-NNN), with a category tag in the title where
useful; the legacy package ADR-001…014 is normative and is kept as
DR-001…014 upon completion of the migration (DR-016). The record format is
a replaceable sub-method of the doctrinal function of decision evidence
(Volume I, §8.1) — the function is mandatory; the form is revised by
evidence. Elaborated in Volume III.

## Non-functional requirements

Architectural decisions generate **measurable** derived non-functional
requirements, each anchored to ISO/IEC 25010, identified as
`NFR-<CATEGORY>-NN`, and enforced through coding standards, architectural
reviews, static analysis and CI/CD controls. The conformance measure is a
vector — per dimension, with no scalar overall score, because a weighted
sum across nominal and ordinal scales is not a defined operation (P-09).
The register is kept in NFR.md.

## Standards compliance

Rather than replacing standards, Wattleflow integrates them: PEP,
ISO/IEC/IEEE 42010, ISO/IEC 25010/25012/12207, OMG SBVR and OCL, NIST
OSCAL, W3C RDF/OWL/PROV-O and (future) SPDX/CycloneDX. A standard enters
through epistemic judgement, not by inertia; what each covers is listed in
Volume I (§10).

## Vision

The long-term goal of Wattleflow is not to provide yet another workflow
framework, but to demonstrate that modern information systems can be
developed with a methodology that unites scientific reasoning, software
engineering, information science, internationally recognised standards and
practical implementation.

The resulting framework should be technically rigorous, educational,
explainable, maintainable, compliant, AI-compatible and architecturally
self-describing.

In this philosophy, architecture is not the documentation of software.

**Architecture is the scientific foundation from which software arises.**

---

## References

This document does not maintain its own literature table (P-20: a view is
not a source of truth). Keys [n] refer to the consolidated table
`documents/LITERATURA.md`; P-NN codes refer to the register of postulates
`documents/POSTULATI.md` (the claim, with its origin, application and key).

Citation standard: numeric keys (IEEE style), unique across all doctrinal
documents and the paper, under an **append-only** rule (a new key goes
strictly to the end of the table — insertion would renumber citations in
every document). The evolutionary path towards stable identifiers
(literatura.yaml, with numbers as a generated view) is recorded as a DR
candidate.

---

## Synthesis note (v0.4 → v0.4.1)

1. **Terminology consistently DR**: the ADR → DR rename (the author's
   change in the section heading) carried through the entire body; the
   legacy package ADR-001…014 is kept as DR-001…014 after migration
   (DR-016); a link added to METHODOLOGY §8.1 (the record format is a
   replaceable sub-method).
2. **A References section replaces the literature table**: removed the
   local table, a stray paste fragment (including an editorial instruction
   that had ended up in the text) and duplicated Notes (they live with the
   source of truth in documents/literatura.md).
3. **A layer of P-codes introduced**: where a claim exists as a postulate,
   the text cites P-NN (the register of postulates) instead of a direct
   literature key — the referencing architecture is philosophy → postulates
   → literature. Direct keys [n] remain where no postulate exists (the
   genealogy of the DR practice [44][45][46]).
4. **Citation standard decided**: numeric keys (IEEE style) with the
   append-only rule; Harvard rejected on grounds of migration cost and the
   risk of transcription loss; stable identifiers (literatura.yaml) as the
   evolutionary path (a DR candidate).
5. Historical accuracy of the v0.3→v0.4 note restored (ADR-ORG-06).

## Synthesis note (v0.3 → v0.4)

1. **New section: Hypotheses** (H1–H3, falsifiable forms) — applies the
   doctrine of measurability to the philosophy itself; without it, the
   doctrine of critical thinking would be vulnerable to its own test.
2. **"or the Reverse" resolved**: iteration architecture ⇄ implementation
   is permitted exclusively under the cascade, with revision only through a
   DR; the hedged sentence about not imposing an ontological discourse was
   also removed.
3. **Zero-trust separated by layer**: the epistemic core ("trust is
   proven") at the level of the philosophy; the named security policy one
   layer below.
4. **DR numbering fixed**: sequential; the existing package normative
   (v0.3 implied a categorical scheme via ADR-ORG-06).
5. References added (keys into the consolidated table) and the genealogy
   of the DR practice; the Motivation section given its historical
   grounding.
6. The section on documentation audiences (the Ashby argument) entered
   *Architecture as an educational artefact*.
7. Language revision (typos, duplicated headings, consolidation of
   repeated paragraphs); nothing was removed as content, only merged.
