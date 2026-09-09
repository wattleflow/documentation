# NFRQ — non-functional requirements register

| | |
|---|---|
| **Identifier** | `NFR-<CATEGORY>-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Quality anchor** | ISO/IEC 25010 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/HLRQ-000-EN.md) · [FRQ](../02-FRQ/FRQ-000-EN.md) |
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
| `NFRQ-DEF-01` | Common definitions — domain, helper, canonical subject | [entry](NFRQ-DEF-01-common-definitions-EN.md) |
| `NFRQ-DEF-02` | Charter for measurable `[M]` criteria | [entry](NFRQ-DEF-02-measurement-charter-EN.md) |
| `NFRQ-APX-01` | Appendix A — facet order in class nomenclature | [entry](NFRQ-APX-01-facet-order-EN.md) |

## ORG — organisation and structure

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-ORG-01` | Dependency locality of helper classes | `domain_acyclicity` **error** · `helper_fan_in` info | [entry](NFRQ-ORG-01-helper-locality-EN.md) |
| `NFRQ-ORG-02` | Class nomenclature | `base_family_membership` **error** · `prohibited_standalone` **error** · `acronym_case` warning | [entry](NFRQ-ORG-02-class-nomenclature-EN.md) |
| `NFRQ-ORG-03` | Type-variable nomenclature | `typevar_role_vocabulary` warning | [entry](NFRQ-ORG-03-typevar-nomenclature-EN.md) |
| `NFRQ-ORG-04` | Cross-cutting capability as a helper | partly `prohibited_standalone`; rest by review | [entry](NFRQ-ORG-04-crosscutting-capability-EN.md) |
| `NFRQ-ORG-05` | Encapsulating self-referencing helper methods | AST; not automated | [entry](NFRQ-ORG-05-self-referencing-helpers-EN.md) |
| `NFRQ-ORG-06` | *withdrawn* — decomposed into SEC-01/02/03 (2026-07-22) | — | — |
| `NFRQ-ORG-07` | Component input surface (`ALLOWED`) | `preset_allowed_declaration` **error** | **gap** |
| `NFRQ-ORG-08` | Deduplication: one place per rule, not per shape | not measured | [entry](NFRQ-ORG-08-deduplication-EN.md) |
| `NFRQ-ORG-09` | A specialisation over an external standard exposes its model | not measured | [entry](NFRQ-ORG-09-external-standard-EN.md) |
| `NFRQ-ORG-10` | A learned artefact as a decision criterion — deterministic first, manifest, declared opacity | `clean_core_imports` for c.7; rest **by review** | [entry](NFRQ-ORG-10-learned-artefact-EN.md) |

> **Declared blind spot (D-11): `NFRQ-ORG-07` has no entry.** Both linters run
> `preset_allowed_declaration` as an **error**, and `NFRQ-ORG-08` and `NFRQ-SEC-06` c.1 both cite
> `NFRQ-ORG-07` as their source — yet no section exists. Opening the entry requires a DR (D-03).
> Until then the rule is enforced without a written criterion.

## OBS — audit-record observability

> Category introduced by `DR-WFL-018` (2026-08-23), amended by `DR-WFL-021` (2026-08-24). Every
> rule is `warning` — the current state is a worklist, not a build-breaking violation.
> Demarcation: `NFRQ-SEC-06` governs the **confidentiality** of the record, `NFRQ-OBS-*` its
> **readability and cost**.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-OBS-01` | Audit levels: meaning, audience, placement | `audit_level_vs_propagation` · `audit_failure_trace` | [entry](NFRQ-OBS-01-audit-levels-EN.md) |
| `NFRQ-OBS-02` | Record fields: `msg`, `step`, `scope`, `component`, `target` | `audit_event_vocabulary` | [entry](NFRQ-OBS-02-audit-fields-EN.md) |
| `NFRQ-OBS-03` | Ownership, order and volume of audit `[M]` | `audit_info_placement` | [entry](NFRQ-OBS-03-audit-ownership-volume-EN.md) |
| `NFRQ-OBS-04` | What a metric must carry to be admissible — distribution, subgroups, operational definition | **by review**; c.6 AST-checkable | [entry](NFRQ-OBS-04-metric-admissibility-EN.md) |

## SEC — zero-trust elaboration

> Zero-trust is the founding concept; SEC breaks it into bounded and/or measurable parts. The
> structural `ORG-*` requirements and packaging are its **mechanism**, not its requirement.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-SEC-01` | Blast-radius containment (compartmentalisation) | `[M]` diagnostic | [entry](NFRQ-SEC-01-blast-radius-EN.md) |
| `NFRQ-SEC-02` | Attack-surface minimality | `[M]` diagnostic | [entry](NFRQ-SEC-02-attack-surface-EN.md) |
| `NFRQ-SEC-03` | Supply-chain trust and distribution locality | `clean_core_imports` **error** · `distribution_manifest` warning | [entry](NFRQ-SEC-03-supply-chain-locality-EN.md) |
| `NFRQ-SEC-04` | Detection operating point and psychological acceptability | by review | [entry](NFRQ-SEC-04-detection-operating-point-EN.md) |
| `NFRQ-SEC-05` | Adversary model and investment discipline | by review | [entry](NFRQ-SEC-05-adversary-model-EN.md) |
| `NFRQ-SEC-06` | Audit-record confidentiality | review + AST for c.1; **not measured** | [entry](NFRQ-SEC-06-audit-confidentiality-EN.md) |
| `NFRQ-SEC-07` | Emission is an authorised act, not a configuration key | by review; `[M]` c.6 is a **gate candidate**, not measured | [entry](NFRQ-SEC-07-emission-authorisation-EN.md) |

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
restating sources. The exception is [`NFRQ-APX-01`](NFRQ-APX-01-facet-order-EN.md), which treats a
separate subject and carries its own inline references.

**Normative references** (policy targets called directly by the criteria): ISO/IEC 25010:2011 ·
ISO/IEC 15939:2017 · NIST OSCAL · PEP 8, 420, 484, 660, 740 · CycloneDX / SPDX · Sigstore.
