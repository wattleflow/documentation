# HLRQ — high-level requirements register

| | |
|---|---|
| **Identifier** | `HLRQ-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Class** | **in use but not in the register** — introducing it requires a DR (D-12); see §Open |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §4 *Requirement ontology* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [FRQ](../02-FRQ/0-FRQ-EN.md) · [NFRQ](../03-NFRQ/0-NFRQ-EN.md) |
| **Language** | Index EN; the entries are Croatian (source, `CLAUDE.md` §3.2) |

This document is an **index**, not a register: each capability lives in its own file
(`category-number-slug.md`).

**What an HLRQ is.** A high-level requirement carries the **narrative and business rules**
(`BR-nn`) of one capability, and the **scope** for the group of FRs beneath it. It **does not
describe steps** — the actors and per-class steps are described by its child FRs, which cite the
same rules.

The order of answering is **Why → What → How** (`CLAUDE.md` §3.4): the HLRQ carries the *why*,
the FR the *what*, the design and code the *how*.

## Register

| Id | Capability | Children | Distribution | Detail | Status |
|---|---|---|---|---|---|
| `HLRQ-13` | Translation through transformation — a persistence layer for language models | [`FR-CON-13.1`](../02-FRQ/FRQ-CON-13.1-huggingface-connection.md) · [`FR-CON-13.2`](../02-FRQ/FRQ-CON-13.2-remote-model-connection.md) · [`FR-DRV-13`](../02-FRQ/FRQ-DRV-13-llm-model.md) · `FR-PIP-13` (candidate) | `wattleflow-processors` | [HLRQ-13](HLRQ-13-llm-models.md) | partly implemented |
| `HLRQ-14` | Machine-checkable compliance — the OSCAL model, the baseline and the `declared ⊆ baseline` gate | [`FR-OSCAL-14.1`…`14.13`](../02-FRQ/0-FRQ-EN.md#oscal--machine-checkable-compliance) | `wattleflow-processors` | [HLRQ-14](HLRQ-14-oscal.md) | partly implemented |

## Traceability to the NFRs

| HLRQ | Key NFRs |
|---|---|
| `HLRQ-13` | [`NFR-ORG-04`](../03-NFRQ/NFR-ORG-04-crosscutting-capability-EN.md) (ontology: `Connection`, `Driver`, `Pipeline`) · [`NFR-SEC-03`](../03-NFRQ/NFR-SEC-03-supply-chain-locality-EN.md) (distribution locality) |
| `HLRQ-14` | [`NFR-SEC-01`](../03-NFRQ/NFR-SEC-01-blast-radius-EN.md)…[`NFR-SEC-06`](../03-NFRQ/NFR-SEC-06-audit-confidentiality-EN.md) · [`NFR-ORG-04`](../03-NFRQ/NFR-ORG-04-crosscutting-capability-EN.md) · [`NFR-ORG-05`](../03-NFRQ/NFR-ORG-05-self-referencing-helpers-EN.md) · [`NFR-ORG-08`](../03-NFRQ/NFR-ORG-08-deduplication-EN.md) |

## Open

* **The `HLRQ` class is not in the requirements register.** `CLAUDE.md` §3.6 records it as "in
  use but not in the register"; introducing it requires a DR (D-12). Until then the identifiers
  are **provisional**.

* **Both capabilities carry their own "open" items.** `HLRQ-13`: no DR record in the series had
  been opened when the narrative was written (`DR-PRC-001` was opened afterwards). `HLRQ-14`: the
  gate is `strict=False` and **inert** until it is decided who supplies the active policy — the
  check precedes construction (`DR-WFL-014`), but does not run while no call site passes
  `oscal_policy=`.

* **Neither capability is fully implemented.** Status changes through a DR, never tacitly (D-03).
