<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-12 — Availability of a supporting instance

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration; corollary: an instance is recreatable from the repository plus one untracked file, so availability is a property of the description, not of a machine |
| **CIA** | **Availability** |
| **Quality (25010)** | Reliability — availability, recoverability, fault tolerance *(25010 places availability under Reliability; the CIA triad names it as a security goal — both are declared)* |
| **Enforcement** | text checks over `dockers/<instance>/` (c.1–c.3); test for c.4 (`test_exporters.py`); review for c.5; **not in `wem_lint`** — declared blind spot (D-11) |
| **Reference frame** | ISO/IEC 27002:2022 §8.13 *Information backup*, §8.14 *Redundancy* [61]; NIST SP 800-53 `CP-9`, `CP-10`, `SC-5` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A supporting instance survives a container restart without losing what a run pushed, is
**bounded** in what it keeps, and is **never a dependency of the workflow**: a run completes and
reports whether or not the instance is up.

## 2. Acceptance criteria

1. **State lives in named volumes** and the pushed groups are persisted (`--persistence.file`);
   a recreate of the containers keeps the last push. *(machine-checkable — text; verified by recreate)*
2. **Retention is bounded** (`--storage.tsdb.retention.time` or size); an unbounded store is a
   finding — the host disk is a resource of the run (`HLRQ-18`). *(machine-checkable — text)*
3. **Restart policy is declared** (`unless-stopped`); the instance comes back with the host.
   *(machine-checkable — text)*
4. **The workflow does not depend on the instance.** A failed push is `WARNING` and the pass
   completes; no `run()` path blocks on the sink. *(test)*
5. **Recreatable from the repository alone**: `git clone` + `cp .env.example .env` + values +
   `docker compose up` yields a working instance; anything else needed is a finding. *(by review;
   verified once per instance and recorded in its FRQ)*

## 3. Verification

c.1–c.3 by text on demand; c.4 by `blackwattle/tests/metrics/test_exporters.py` (sink failure
is a warning); c.5 by the first-run walkthrough in `FRQ-OPS-20.1` §10. Health checks
(`healthcheck:`) and resource limits (`deploy.resources`) are **candidates** (§4).

## 4. Justification

| Principle | Implication |
|---|---|
| P-10 — an indicator used for control stops measuring | a workflow that waits for its monitor makes the monitor a control; the push is fire-and-report |
| Recoverability (25010) | the description in the repository *is* the backup of the instance; only the data volume needs one |
| `HLRQ-18` BR-07 | the monitor observes and reports; its own failure never fails the workflow |
| Reference standard: ISO/IEC 25010 — Reliability (availability, recoverability) | |

## 5. Open

Health checks on each service; CPU/memory limits per container; backup of the Prometheus
volume. Each is a compose change — a decision only if it becomes a criterion.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
