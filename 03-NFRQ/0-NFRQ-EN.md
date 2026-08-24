# NFRQ — non-functional requirements register

| | |
|---|---|
| **Identifier** | `NFR-<CATEGORY>-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Quality anchor** | ISO/IEC 25010 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/0-HLRQ-EN.md) · [FRQ](../02-FRQ/0-FRQ-EN.md) |
| **Language** | EN only — see *Language* below |

This document is an **index**, not a register: each requirement lives in its own file
(`category-number-slug-EN.md`). The rendering is not the source of truth (D-13) — finding
counts are not quoted here; cite the conformance snapshot in
[`../workflow/conformance/`](../workflow/conformance/) instead.

Every NFR carries a **statement**, **acceptance criteria**, a **verification method**, its
**justification**, and **traceability** upwards. Changing a criterion goes through a DR (D-03).

## Shared

| Id | Subject | Entry |
|---|---|---|
| `NFR-DEF-01` | Common definitions — domain, helper, canonical subject | [entry](NFR-DEF-01-common-definitions-EN.md) |
| `NFR-DEF-02` | Charter for measurable `[M]` criteria | [entry](NFR-DEF-02-measurement-charter-EN.md) |
| `NFR-APX-01` | Appendix A — facet order in class nomenclature | [entry](NFR-APX-01-facet-order-EN.md) |

## ORG — organisation and structure

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFR-ORG-01` | Dependency locality of helper classes | `domain_acyclicity` **error** · `helper_fan_in` info | [entry](NFR-ORG-01-helper-locality-EN.md) |
| `NFR-ORG-02` | Class nomenclature | `base_family_membership` **error** · `prohibited_standalone` **error** · `acronym_case` warning | [entry](NFR-ORG-02-class-nomenclature-EN.md) |
| `NFR-ORG-03` | Type-variable nomenclature | `typevar_role_vocabulary` warning | [entry](NFR-ORG-03-typevar-nomenclature-EN.md) |
| `NFR-ORG-04` | Cross-cutting capability as a helper | partly `prohibited_standalone`; rest by review | [entry](NFR-ORG-04-crosscutting-capability-EN.md) |
| `NFR-ORG-05` | Encapsulating self-referencing helper methods | AST; not automated | [entry](NFR-ORG-05-self-referencing-helpers-EN.md) |
| `NFR-ORG-06` | *withdrawn* — decomposed into SEC-01/02/03 (2026-07-22) | — | — |
| `NFR-ORG-07` | Component input surface (`ALLOWED`) | `preset_allowed_declaration` **error** | **gap** |
| `NFR-ORG-08` | Deduplication: one place per rule, not per shape | not measured | [entry](NFR-ORG-08-deduplication-EN.md) |

> **Declared blind spot (D-11): `NFR-ORG-07` has no entry.** Both linters run
> `preset_allowed_declaration` as an **error**, and `NFR-ORG-08` and `NFR-SEC-06` c.1 both cite
> `NFR-ORG-07` as their source — yet no section exists. Opening the entry requires a DR (D-03).
> Until then the rule is enforced without a written criterion.

## OBS — audit-record observability

> Category introduced by `DR-WFL-018` (2026-08-23), amended by `DR-WFL-021` (2026-08-24). Every
> rule is `warning` — the current state is a worklist, not a build-breaking violation.
> Demarcation: `NFR-SEC-06` governs the **confidentiality** of the record, `NFR-OBS-*` its
> **readability and cost**.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFR-OBS-01` | Audit levels: meaning, audience, placement | `audit_level_vs_propagation` · `audit_failure_trace` | [entry](NFR-OBS-01-audit-levels-EN.md) |
| `NFR-OBS-02` | Record fields: `msg`, `step`, `scope`, `component`, `target` | `audit_event_vocabulary` | [entry](NFR-OBS-02-audit-fields-EN.md) |
| `NFR-OBS-03` | Ownership, order and volume of audit `[M]` | `audit_info_placement` | [entry](NFR-OBS-03-audit-ownership-volume-EN.md) |

## SEC — zero-trust elaboration

> Zero-trust is the founding concept; SEC breaks it into bounded and/or measurable parts. The
> structural `ORG-*` requirements and packaging are its **mechanism**, not its requirement.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFR-SEC-01` | Blast-radius containment (compartmentalisation) | `[M]` diagnostic | [entry](NFR-SEC-01-blast-radius-EN.md) |
| `NFR-SEC-02` | Attack-surface minimality | `[M]` diagnostic | [entry](NFR-SEC-02-attack-surface-EN.md) |
| `NFR-SEC-03` | Supply-chain trust and distribution locality | `clean_core_imports` **error** · `distribution_manifest` warning | [entry](NFR-SEC-03-supply-chain-locality-EN.md) |
| `NFR-SEC-04` | Detection operating point and psychological acceptability | by review | [entry](NFR-SEC-04-detection-operating-point-EN.md) |
| `NFR-SEC-05` | Adversary model and investment discipline | by review | [entry](NFR-SEC-05-adversary-model-EN.md) |
| `NFR-SEC-06` | Audit-record confidentiality | review + AST for c.1; **not measured** | [entry](NFR-SEC-06-audit-confidentiality-EN.md) |

## Language

`CLAUDE.md` §3.2 makes the Croatian text authoritative until v1.0, yet **this register exists
only in English**: the split of 2026-08-24 produced no HR edition, so there is nothing for the
EN text to be a translation *of*. Two consequences, both declared rather than resolved (D-11):
the policy and the tree disagree (a finding under D-02), and no HR/EN drift check is needed
because there is no pair to compare. Resolution — extend §3.2 or restore the HR edition —
requires a DR (D-03); tracked in [`workflow/TODO.md`](../workflow/TODO.md).

## Verification and scope

No test framework or CI has been chosen (`CLAUDE.md` §4). Until then "machine-checkable" means
**lint on demand**: `python tools/wem_lint.py --snapshot`. The criterion is separate from the
tool and versioned separately (`tools/dictionary.json`); finding presentation is a third,
again separately versioned artefact (`tools/messages.json`, D-13).

**The register is cross-distribution.** The same NFRs govern `wattleflow`,
`wattleflow-workflow` and `wattleflow-processors`; only the **criterion** differs — each
distribution ships its own `tools/dictionary.json` with its own `rules[]` block and severities.
No NFR today is owned by a single distribution.

## Scientific basis

This register is **policy derived from research** and carries no bibliography of its own. The
basis for the measurement (`[M]`) and security (SEC) requirements is
[`workflow/Analiza.md`](../workflow/Analiza.md); NFR criteria cite its **conclusions** rather than
restating sources. The exception is [`NFR-APX-01`](NFR-APX-01-facet-order-EN.md), which treats a
separate subject and carries its own inline references.

**Normative references** (policy targets called directly by the criteria): ISO/IEC 25010:2011 ·
ISO/IEC 15939:2017 · NIST OSCAL · PEP 8, 420, 484, 660, 740 · CycloneDX / SPDX · Sigstore.
