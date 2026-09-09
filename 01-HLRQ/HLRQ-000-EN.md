# HLRQ — high-level requirements register

| | |
|---|---|
| **Identifier** | `HLRQ-NN` |
| **Version** | Draft v0.2 (split out 2026-08-24) |
| **Class** | **in use but not in the register** — introducing it requires a DR (D-12); see §Open |
| **Parents** | [PHILOSOPHY](../PHILOSOPHY.md) · [METHODOLOGY](../METHODOLOGY.md) §4 *Requirement ontology* · [DOCTRINE](../DOCTRINE.md) |
| **Sibling registers** | [FRQ](../02-FRQ/FRQ-000-EN.md) · [NFRQ](../03-NFRQ/NFRQ-000-EN.md) |
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
| `HLRQ-13` | Translation through transformation — a persistence layer for language models | [`FRQ-CON-13.1`](../02-FRQ/FRQ-CON-13.1-huggingface.md) · [`FRQ-CON-13.2`](../02-FRQ/FRQ-CON-13.2-remote-model.md) · [`FRQ-DRV-13`](../02-FRQ/FRQ-DRV-13-llm-model.md) · `FRQ-PIP-13` (candidate) | `wattleflow-processors` | [HLRQ-13](HLRQ-13-llm-models.md) | partly implemented |
| `HLRQ-14` | Machine-checkable compliance — the OSCAL model, the baseline and the `declared ⊆ baseline` gate | [`FRQ-OSCAL-14.1`…`14.13`](../02-FRQ/FRQ-000-EN.md#oscal--machine-checkable-compliance) | `wattleflow-processors` | [HLRQ-14](HLRQ-14-oscal.md) | partly implemented |
| `HLRQ-15` | Pipelined processing through substitutable primitives — the generic layer that fulfils the `core/` contracts | [`FRQ-BBD-15.1`](../02-FRQ/FRQ-BBD-15.1-blackboard.md) · [`FRQ-PIP-15.2`](../02-FRQ/FRQ-PIP-15.2-pipeline.md) · [`FRQ-PRC-15.3`](../02-FRQ/FRQ-PRC-15.3-processor.md) · [`FRQ-STR-15.4`](../02-FRQ/FRQ-STR-15.4-strategy.md) · `15.5`…`15.21` (pending) | `wattleflow-workflow` | [HLRQ-15](HLRQ-15-generic-layer.md) | in code, being recorded |
| `HLRQ-16` | Exchange with a radio device (SDR) on USB — exclusive access, an unstoppable sample stream, a truthful account of what was lost, **one class per access mechanism rather than per vendor**, and emission as a separate contract | [`FRQ-CON-16.1`](../02-FRQ/FRQ-CON-16.1-sdr-device.md) · [`FRQ-DRV-16.2`](../02-FRQ/FRQ-DRV-16.2-iq-stream.md) · [`FRQ-PRC-16.3`](../02-FRQ/FRQ-PRC-16.3-sdr-capture.md) · `FRQ-DOC-16.4`, `FRQ-PIP-16.5` (candidates) | `blackwattle` | [HLRQ-16](HLRQ-16-sdr-capture.md) | **proposed, not implemented** |

## Traceability to the NFRs

| HLRQ | Key NFRs |
|---|---|
| `HLRQ-13` | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) (ontology: `Connection`, `Driver`, `Pipeline`) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) (distribution locality) |
| `HLRQ-15` | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) (ontology, closed list of ten) · [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius-EN.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels-EN.md)…[`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume-EN.md) |
| `HLRQ-14` | [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius-EN.md)…[`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality-EN.md) · [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) · [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers-EN.md) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication-EN.md) |
| `HLRQ-16` | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) (ontology: `Connection`, `Driver`, `Processor`) · [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius-EN.md) (access to physical hardware) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) · [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume-EN.md) (record volume per pass) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication-EN.md) (**binding** via `BR-13`: shared contract → shared code; different contracts → never merged) · [`NFRQ-SEC-07`](../03-NFRQ/NFRQ-SEC-07-emission-authorisation-EN.md) (emission authorisation) |

## Open

* **The `HLRQ` class is not in the requirements register.** `CLAUDE.md` §3.6 records it as "in
  use but not in the register"; introducing it requires a DR (D-12). Until then the identifiers
  are **provisional**.

* **Both capabilities carry their own "open" items.** `HLRQ-13`: no DR record in the series had
  been opened when the narrative was written (`DR-PRC-001` was opened afterwards). `HLRQ-14`: the
  gate is `strict=False` and **inert** until it is decided who supplies the active policy — the
  check precedes construction (`DR-WFL-014`), but does not run while no call site passes
  `oscal_policy=`.

* **`HLRQ-16` is a proposal with no code and no decision record.** It is the first entry whose
  subject is a **local device** rather than a network or filesystem sub-system, and the first to
  need an OSCAL **subset** declaration (`HLRQ-16` §6). Its open items are decisions, not detail.
  Its multi-vendor axis is proposed by [`DR-PRC-003`](../04-DR/DR-PRC-003-sdr-access-layer.md)
  (2026-09-09, **proposal**); which access mechanism carries it is still open.

* **Neither capability is fully implemented.** Status changes through a DR, never tacitly (D-03).
