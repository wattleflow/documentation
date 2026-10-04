<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-14 — Telemetry confidentiality at the export boundary

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration; corollary: what a run exports is read by a system the run does not control, so the export carries measures and identities, never values |
| **CIA** | **Confidentiality** |
| **Quality (25010)** | Security — confidentiality |
| **Enforcement** | review + text over `Monitor.samples()` labels; AST candidate for label names; **not in `wem_lint`** — declared blind spot (D-11) |
| **Reference frame** | *Privacy Act 1988* (Cth), APP 3, APP 11; ISO/IEC 27002:2022 §8.11 *Data masking* [61]; extends [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md) and [`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) c.4 to the monitoring plane |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

> **Demarcation.** [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md) governs the audit
> record; [`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) governs what a metric must carry
> to be admissible. This entry governs what a metric **must not** carry once it leaves the process.

## 1. Statement

A sample that leaves the process — a Prometheus exposition, a Grafana annotation, any exporter —
carries **measures** (counts, bytes, seconds) and **declared identities** (`workflow`, `app`,
component family, operation name). It never carries a filename, a path, a document identifier, a
user name, a credential or document content, in a label **or** in a value.

## 2. Acceptance criteria

1. **Label vocabulary is closed**: `workflow`, `app`, `component`, `operation`, `outcome`,
   `scope`, `mode`, `stat`, `job`, `instance`. A new label enters with a documented change (D-12).
   *(machine-checkable — set comparison over `samples()`)*
2. **No label is an identifier of a document, file, path or person** ([`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) c.4). The
   `path` label on `wf_storage_free_bytes` / `wf_storage_total_bytes` is a **standing finding**
   (worklist): replace by the driver name. *(machine-checkable — label name list)*
3. **Values are numbers**; a string never travels as a metric value or annotation text except the
   workflow name. *(machine-checkable)*
4. **The export destination is declared** in `monitoring: exporters:`; a run with no exporter
   exports nothing. *(test — `test_exporters.py`)*
5. **Cross-boundary export is de-identified by the (sample, exporter) pair**, as `SEC-06` c.3 for
   handlers: an exporter to a shared gateway may drop labels a local one keeps. *(by review;
   no such exporter exists today)*

## 3. Verification

c.1–c.3 by a label listing over `Monitor.samples()` on demand (the metric names are enumerated in
`FRQ-MET-01` §8); c.4 by test. The `path` label is the one known violation of c.2 on 2026-09-19.

## 4. Justification

| Principle | Implication |
|---|---|
| Minimisation (APP 3) | telemetry is collected for cost analysis; nothing beyond the measure is needed for that purpose |
| P-23 | the gateway is another trust domain; a label is a value that crosses it |
| Cardinality ([`NFRQ-OBS-04`](NFRQ-OBS-04-metric-admissibility.md) c.4) | identifiers as labels are a confidentiality defect *and* an availability defect (series explosion) — one criterion serves both |
| Reference standard: ISO/IEC 25010 — Security (confidentiality) | |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
