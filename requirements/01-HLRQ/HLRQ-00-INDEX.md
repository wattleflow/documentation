<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# HLRQ — high-level requirements index

| | |
|---|---|
| **Identifier** | `HLRQ-<SLUG>` (the file name without `.md`) |
| **Class** | **in use but not in the register** — introducing it requires a documented change (D-12) |
| **Parents** | [PHILOSOPHY](../../PHILOSOPHY.md) · [METHODOLOGY](../../METHODOLOGY.md) §4 *Requirement ontology* · [DOCTRINE](../../DOCTRINE.md) |
| **Sibling registers** | [FRQ](../02-FRQ/FRQ-000-INDEX.md) · [NFRQ](../03-NFRQ/NFRQ-000-INDEX.md) |
| **Language** | Index EN; the entries are Croatian (source, `DOCUMENTATION.md` §3.2) |

This document is an **index**, not a register: each capability lives in its own file.

**What an HLRQ is.** A high-level requirement carries the **narrative and business rules**
(`BR-<abbreviation>-<nn>`) of one capability, and the **scope** for the group of FRs beneath it. It
**does not describe steps** — the actors and per-class steps are described by its child FRs, which
cite the same rules.

The order of answering is **Why → What → How** (`DOCUMENTATION.md` §3.4): the HLRQ carries the *why*,
the FR the *what*, the design and code the *how*.

## Register

| Id | Capability | Children | Distribution | Status |
|---|---|---|---|---|
| [`HLRQ-01-GENERIC-LAYER`](HLRQ-01-GENERIC-LAYER.md) | Pipelined processing through substitutable primitives — the generic layer that fulfils the `core/` contracts | the *FRQ* column of the table in §3 | `wattleflow-workflow` (`concrete/`) | in code |
| [`HLRQ-10-SERIALISATION`](HLRQ-10-SERIALISATION.md) | Format boundary — parser, formatter and converter of the generic layer | [`FRQ-SER-PAR`](../02-FRQ/FRQ-SER-PAR-parser.md) · [`FRQ-SER-FMT`](../02-FRQ/FRQ-SER-FMT-formatter.md) · [`FRQ-SER-CNV`](../02-FRQ/FRQ-SER-CNV-converter.md) | `wattleflow-workflow` — clean core tier | in code, recorded |

## Traceability to the NFRs

| HLRQ | Key NFRs |
|---|---|
| [`HLRQ-01-GENERIC-LAYER`](HLRQ-01-GENERIC-LAYER.md) | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) · [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) · [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) · [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md)…[`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) · [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) · [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) |
