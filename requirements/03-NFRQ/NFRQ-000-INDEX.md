<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ — non-functional requirements register

| | |
|---|---|
| **Identifier** | `NFR-<CATEGORY>-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Quality anchor** | ISO/IEC 25010 (ISO/IEC 25012 for data) |
| **Parents** | [PHILOSOPHY](../../PHILOSOPHY.md) · [METHODOLOGY](../../METHODOLOGY.md) §6 *Traceability* · [DOCTRINE](../../DOCTRINE.md) |
| **Sibling registers** | [HLRQ](../01-HLRQ/HLRQ-00-INDEX.md) · [FRQ](../02-FRQ/FRQ-000-INDEX.md) |
| **Language** | EN only — see4 *Language* below |

This document is an **index**, not a register: each requirement lives in its own file
(`category-number-slug-EN.md`). The rendering is not the source of truth (D-13) — finding
counts are not quoted here; cite the conformance snapshot in
`../workflow/conformance/` instead.

Every NFR carries a **statement**, **acceptance criteria**, a **verification method**, its
**justification**, and **traceability** upwards. Changing a criterion needs a documented change (D-03).

## Shared

| Id | Subject | Entry |
|---|---|---|
| `NFRQ-DEF-01` | Common definitions — domain, helper, canonical subject | [entry](NFRQ-DEF-01-common-definitions.md) |
| `NFRQ-DEF-02` | Charter for measurable `[M]` criteria | [entry](NFRQ-DEF-02-measurement-charter.md) |
| `NFRQ-DEF-03` | Comparison and boundary values — the facets every compared or bounded value declares *(proposal, 2026-09-13)* | [entry](NFRQ-DEF-03-comparison-and-boundary-values.md) |
| `NFRQ-APX-01` | Appendix A — facet order in class nomenclature | [entry](NFRQ-APX-01-facet-order.md) |

## ORG — organisation and structure

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-ORG-01` | Dependency locality of helper classes | `domain_acyclicity` **error** · `helper_fan_in` info | [entry](NFRQ-ORG-01-helper-locality.md) |
| `NFRQ-ORG-02` | Class nomenclature | `base_family_membership` **error** · `prohibited_standalone` **error** · `acronym_case` warning | [entry](NFRQ-ORG-02-class-nomenclature.md) |
| `NFRQ-ORG-03` | Type-variable nomenclature | `typevar_role_vocabulary` warning | [entry](NFRQ-ORG-03-typevar-nomenclature.md) |
| `NFRQ-ORG-04` | Cross-cutting capability as a helper | partly `prohibited_standalone`; rest by review | [entry](NFRQ-ORG-04-crosscutting-capability.md) |
| `NFRQ-ORG-05` | Encapsulating self-referencing helper methods | AST; not automated | [entry](NFRQ-ORG-05-self-referencing-helpers.md) |
| `NFRQ-ORG-06` | *withdrawn* — decomposed into SEC-01/02/03 (2026-07-22) | — | — |
| `NFRQ-ORG-07` | Component input surface (`ALLOWED`) | `preset_allowed_declaration` **error** | [entry](NFRQ-ORG-07-preset-allowed-declaration.md) |
| `NFRQ-ORG-08` | Deduplication: one place per rule, not per shape | not measured | [entry](NFRQ-ORG-08-deduplication.md) |
| `NFRQ-ORG-09` | A specialisation over an external standard exposes its model | not measured | [entry](NFRQ-ORG-09-external-standard.md) |
| `NFRQ-ORG-10` | A learned artefact as a decision criterion — deterministic first, manifest, declared opacity | `clean_core_imports` for c.7; rest **by review** | [entry](NFRQ-ORG-10-learned-artefact.md) |
| `NFRQ-ORG-11` | The class is the unit of code; a module function only when exported and with a declared reason (`pep562`, `decorator`, `entry-point`, `standard-signature`) | AST; **not automated** — proposal | [entry](NFRQ-ORG-11-classes-over-module-functions.md) |
| `NFRQ-ORG-12` | Constants and enumerations live in their designated modules (`constants/`, `enums/`) | AST; **not automated** — accepted | [entry](NFRQ-ORG-12-constants-and-enumerations.md) |
| `NFRQ-ORG-13` | The public surface of a contract class is its interface — everything else is `_`-prefixed *(proposal, 2026-09-29; raised by `HLRQ-24`)* | introspection; **not in lint**, baseline in `06-ANALYSIS` | [entry](NFRQ-ORG-13-public-surface-is-the-interface.md) |
| `NFRQ-ORG-14` | A consumer reaches a converter only through its interface — never its parser or an engine parser for the same format *(proposal, 2026-10-05; raised by `FRQ-CNV-24.9`; identifier provisional)* | import graph and AST; **not in lint**, baseline in `06-ANALYSIS` | [entry](NFRQ-ORG-14-consumer-reaches-the-interface.md) |

## OBS — audit-record observability

> Category introduced by (2026-08-23), amended by (2026-08-24). Every
> rule is `warning` — the current state is a worklist, not a build-breaking violation.
> Demarcation: `NFRQ-SEC-06` governs the **confidentiality** of the record, `NFRQ-OBS-*` its
> **readability and cost**.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-OBS-01` | Audit levels: meaning, audience, placement | `audit_level_vs_propagation` · `audit_failure_trace` | [entry](NFRQ-OBS-01-audit-levels.md) |
| `NFRQ-OBS-02` | Record fields: `msg`, `step`, `scope`, `component`, `target` | `audit_event_vocabulary` | [entry](NFRQ-OBS-02-audit-fields.md) |
| `NFRQ-OBS-03` | Ownership, order and volume of audit `[M]` | `audit_info_placement` | [entry](NFRQ-OBS-03-audit-ownership-volume.md) |
| `NFRQ-OBS-04` | What a metric must carry to be admissible — distribution, subgroups, operational definition | **by review**; c.6 AST-checkable | [entry](NFRQ-OBS-04-metric-admissibility.md) |

## SEC — zero-trust elaboration

> Zero-trust is the founding concept; SEC breaks it into bounded and/or measurable parts. The
> structural `ORG-*` requirements and packaging are its **mechanism**, not its requirement.
>
> `SEC-09`…`SEC-14` form the **CIA suite** for a deployed instance (`HLRQ-20`): each names the
> triad member it serves and references postulate **P-23** (`POSTULATE.md`), introduced by
>.

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-SEC-01` | Blast-radius containment (compartmentalisation) | `[M]` diagnostic | [entry](NFRQ-SEC-01-blast-radius.md) |
| `NFRQ-SEC-02` | Attack-surface minimality | `[M]` diagnostic | [entry](NFRQ-SEC-02-attack-surface.md) |
| `NFRQ-SEC-03` | Supply-chain trust and distribution locality | `clean_core_imports` **error** · `distribution_manifest` warning | [entry](NFRQ-SEC-03-supply-chain-locality.md) |
| `NFRQ-SEC-04` | Detection operating point and psychological acceptability | by review | [entry](NFRQ-SEC-04-detection-operating-point.md) |
| `NFRQ-SEC-05` | Adversary model and investment discipline | by review | [entry](NFRQ-SEC-05-adversary-model.md) |
| `NFRQ-SEC-06` | Audit-record confidentiality | review + AST for c.1; **not measured** | [entry](NFRQ-SEC-06-audit-confidentiality.md) |
| `NFRQ-SEC-07` | Emission is an authorised act, not a configuration key | by review; `[M]` c.6 is a **gate candidate**, not measured | [entry](NFRQ-SEC-07-emission-authorisation.md) |
| `NFRQ-SEC-08` | Emitted-document confidentiality: a declared marking, no residue *(proposal, 2026-09-13)* | by review; **not measured** | [entry](NFRQ-SEC-08-emitted-document-confidentiality.md) |
| `NFRQ-SEC-09` | No secret in version control — tree and history clean, ignore by pattern, placeholders closed *(proposal, 2026-09-19; **P-23**)* | on-demand scanner; **not in lint** | [entry](NFRQ-SEC-09-no-secret-in-version-control.md) |
| `NFRQ-SEC-10` | A secret enters at the process boundary and never leaves through a record — no credential default, no value in records *(proposal, 2026-09-19; **P-23**)* | AST candidate `credential_default`; **not in lint** | [entry](NFRQ-SEC-10-secrets-enter-at-the-process-boundary.md) |
| `NFRQ-SEC-11` | Integrity of a deployed instance — pinned images, read-only provisioning, repository as the only change path *(proposal, 2026-09-19; **P-23**)* | text checks; **not in lint** | [entry](NFRQ-SEC-11-instance-integrity.md) |
| `NFRQ-SEC-12` | Availability of a supporting instance — volumes, bounded retention, restart policy, workflow never depends on it *(proposal, 2026-09-19; **P-23**)* | text checks + `test_exporters.py` | [entry](NFRQ-SEC-12-instance-availability.md) |
| `NFRQ-SEC-13` | Network exposure and least privilege of an instance — loopback by default, widening defaults off, no privilege escalation *(proposal, 2026-09-19; **P-23**)* | text checks; **not in lint** | [entry](NFRQ-SEC-13-network-exposure-and-least-privilege.md) |
| `NFRQ-SEC-14` | Telemetry confidentiality at the export boundary — closed label vocabulary, no identifier leaves the process *(proposal, 2026-09-19; **P-23**)* | label listing; `path` label is a standing finding | [entry](NFRQ-SEC-14-telemetry-confidentiality.md) |
| `NFRQ-SEC-15` | Data-plane operation authorisation is declared configuration, not a call-time flag *(proposal, 2026-09-27; raised by `HLRQ-21`)* | by review; `[M]` c.6 is a **gate candidate**, not measured | [entry](NFRQ-SEC-15-operation-authorisation.md) |
| `NFRQ-SEC-16` | Nothing is installed or acquired from code at run time — a missing dependency is named, never a command; models are provisioned *(proposal, 2026-09-29; raised by `HLRQ-23`)* | AST scan; **not in lint**, baseline in `06-ANALYSIS` | [entry](NFRQ-SEC-16-no-runtime-installation.md) |
| `NFRQ-SEC-17` | Third-party code is loaded at use, never at import — in a module, in an aggregate, an absent library is named *(proposal, 2026-09-30; raised by `HLRQ-26`)* | masking test; **not in lint** | [entry](NFRQ-SEC-17-deferred-third-party-loading.md) |

## FUN — functional suitability *(provisional category)*

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-FUN-01` | Conversion fidelity: nothing lost silently, the round trip is a fixed point *(proposal, 2026-09-14)* | unit tests per component; **not measured** by lint | [entry](NFRQ-FUN-01-conversion-fidelity.md) |
| `NFRQ-FUN-02` | Recognition accuracy is measured per engine on a declared reference set — CER/WER `[M]`, no engine ranked without measurement *(proposal, 2026-09-29; raised by `HLRQ-23`)* | measurement script; **not measured** by lint | [entry](NFRQ-FUN-02-recognition-accuracy.md) |

## PRF — performance efficiency *(provisional category)*

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-PRF-01` | Work proportional to input *(proposal, 2026-09-14)* | unit test with a work counter; `[M]` diagnostic | [entry](NFRQ-PRF-01-work-proportional-to-input.md) |
| `NFRQ-PRF-02` | Measurement speed — cost of measuring bounded by the work measured *(proposal, 2026-09-24)* | benchmark (`06-ANALYSIS`) | [entry](NFRQ-PRF-02-measurement-speed.md) |

> `FUN` and `PRF` are not in the category vocabulary; a new quality-axis category enters with a documented
> change. Until then both identifiers are provisional.

## MEM — memory *(provisional category)*

| Id | Subject | Enforcement | Entry |
|---|---|---|---|
| `NFRQ-MEM-01` | Instance memory — classes that inherit `Wattleflow` declare `__slots__` *(proposal, 2026-10-01)* | review; static check proposed; `[M]` diagnostic | [entry](NFRQ-MEM-01-memory.md) |

> `MEM` is also the requirement category of the Memento role on the FRQ axis (`FRQ-MEM`); the prefix
> `FRQ-` or `NFRQ-` tells them apart. The quality-axis category is provisional until a documented change admits it.

## Verification and scope
Verification is "machine-checkable" means **lint on demand**: `python tools/wem_lint.py --snapshot`. 
The criterion is separate from the tool and versioned separately (`tools/dictionary.json`); finding 
presentation is a third, again separately versioned artefact (`tools/messages.json`, D-13).

**The register is cross-distribution.** The same NFRs govern `wattleflow`,
`wattleflow-workflow` and `blackwattle`; only the **criterion** differs — each
distribution ships its own `tools/dictionary.json` with its own `rules[]` block and severities.
No NFR today is owned by a single distribution.

## Standards
**Normative references** (policy targets called directly by the criteria): ISO/IEC 25010:2011 ·
ISO/IEC 15939:2017 · NIST OSCAL · PEP 8, 420, 484, 660, 740 · CycloneDX / SPDX · Sigstore.
