# Wattleflow documentation
![WattleFlow Logo](wattleflow.png)

[![PyPI version](https://img.shields.io/pypi/v/wattleflow-workflow.svg)](https://pypi.org/project/wattleflow-workflow/)
[![Python versions](https://img.shields.io/pypi/pyversions/wattleflow-workflow.svg)](https://pypi.org/project/wattleflow-workflow/)
[![License](https://img.shields.io/pypi/l/wattleflow-workflow.svg)](https://github.com/wattleflow-workflow/core/blob/default/LICENSE)

---
**WattleFlow** — graceful flow,
modular, scaled with purpose,
patterns guide the stream,
extensible, clear design,
built to last and grow.
---

| Characteristic           | Value                                                                   |
| ------------------------ | ----------------------------------------------------------------------- |
| **Version**              | v0.1 |
| **License**              | © 2022–2026 WattleFlow. All rights reserved. |
| **Documentation**        | [**Wattleflow Documentation**](https://github.com/wattleflow/documentation.git) |


## Contents (volumes)

| Volume | Title | Location |
|---|---|---|
| I | Foundations | [PHILOSOPHY](PHILOSOPHY.md) · [DOCTRINE](DOCTRINE.md) · [METHODOLOGY](METHODOLOGY.md) · [POSTULATE](POSTULATE.md) · [LITERATURE](LITERATURE.md) |
| II | Architectural Principles | *(planned)* |
| III | Decisions | carried by the requirement documentation (`requirements/01-HLRQ/`–`03-NFRQ/`) |
| IV | System Design | *(planned)* |
| V | Development Standards | [`requirements/01-HLRQ/`](requirements/01-HLRQ/HLRQ-00-INDEX.md) · [`requirements/02-FRQ/`](requirements/02-FRQ/FRQ-000-INDEX.md) · [`requirements/03-NFRQ/`](requirements/03-NFRQ/NFRQ-000-INDEX.md) *(in progress)* |
| VI | Quality Assurance | *(planned)* |
| VII | AI-Assisted Software Engineering | *(planned)* |
| VIII | Governance, Compliance and Information Science | *(planned)* |

## Where things live

| Layer | Artefact |
|---|---|
| Philosophy (umbrella) | [PHILOSOPHY.md](PHILOSOPHY.md) |
| Norms (registry, 9 articles) | [DOCTRINE.md](DOCTRINE.md) |
| Claims (registry `P-01…P-21`) | [POSTULATE.md](POSTULATE.md) |
| References (append-only, keys 1–64) | [LITERATURE.md](LITERATURE.md) |
| Method (Volume I) | [METHODOLOGY.md](METHODOLOGY.md) · [`requirements/05-METHODS/`](requirements/05-METHODS/dqi.md) |
| Policy | [CLAUDE.md](CLAUDE.md) (= `POLICY.md`) |
| Discourse vocabulary | [dictionary.yaml](dictionary.yaml) → [DICTIONARY.md](DICTIONARY.md) |
| Requirements | [`requirements/01-HLRQ/`](requirements/01-HLRQ/HLRQ-00-INDEX.md) · [`requirements/02-FRQ/`](requirements/02-FRQ/FRQ-000-INDEX.md) · [`requirements/03-NFRQ/`](requirements/03-NFRQ/NFRQ-000-INDEX.md) |
| Analyses · change records | [`requirements/06-ANALYSIS/`](requirements/06-ANALYSIS/) · [`requirements/07-CHANGES/`](requirements/07-CHANGES/) |
| Diagram register (drawn · reserved) | [`requirements/DIAGRAMS.md`](requirements/DIAGRAMS.md) |
| Per-distribution documentation | [`core/`](core/README.md) · [`workflow/`](workflow/README.md) · [`blackwattle/`](blackwattle/README.md) |
| Conformance snapshots | [`workflow/conformance/`](workflow/conformance/) |
| Worklist (state of work, not norm) | [`workflow/TODO.md`](workflow/TODO.md) · [`workflow/DONE.md`](workflow/DONE.md) |

**Language.** The source language is Croatian until v1.0 (`CLAUDE.md` §3.2); English editions
that already exist before that point are a declared divergence, not an approved translation —
see the note in `CLAUDE.md` §3.2.

**What this published edition contains — and what it does not.** From 2026-09-09 the repository
is published, superseding the earlier local-only lock (`CLAUDE.md` §8, still to be reconciled). The published set is the **doctrinal layer and the requirement
registers** only. Deliberately not published: the dated analyses
(`requirements/06-ANALYSIS/`), the alignment records (`requirements/07-CHANGES/`), the method documents (`requirements/05-METHODS/`) and
the worklist.

**Declared consequence (D-11).** Links in the published texts that point into those
unpublished trees do not resolve here. This is stated rather than hidden: an unresolvable
reference is a known cost of the publication scope, not an error in the text, and the traceability
it records still holds in the full tree.
