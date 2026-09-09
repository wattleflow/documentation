# NFRQ-DEF-01 — Common definitions

| | |
|---|---|
| **Role** | Terms used by every `NFRQ-ORG-*` requirement; the definition lives here, everything else references it |
| **Register** | [`0-NFRQ`](NFRQ-000-EN.md) |
| **Discourse vocabulary** | [`dictionary.yaml`](../dictionary.yaml) — source of truth for terms |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Terms

**Domain** — a *top-level functional package*: `pipelines`, `drivers`, `processors`,
`strategies`, `connections`, `documents`, `blackboards`.

**Helper class** — a class that is not part of a domain's public contract (not `Pipeline*`,
`Driver*`, `Processor*`, a strategy…); it exists to serve other classes.

**Domain-internal shared module** — a module shared by two or more **sub-packages of one
domain** (e.g. an OCR helper used by `pipelines/pdf` and `pipelines/png`). It lives **inside
that domain** (`pipelines/convertors`), not in the global `helpers/`.

**Shared helper** — a module on the `helpers/` shelf, organised **by capability** (`io`, `text`,
`routing`, `validation`), never by consuming layer.

**Canonical subject** — the full, unabbreviated subject name, used both as the domain package
name and as the leading facet of a class name (`dataframe`, not `dframe`; `text`, not `txt`).

## 2. The `helpers/` shelf is a shared name

`wattleflow.helpers` is a PEP 420 namespace merged from two distributions (`DR-WFL-017`). The
consequence for any measurement over the shelf: **one tree cannot see all of its consumers**, so
every fan-in count is a lower bound (`DR-WFL-019`). Imports across a distribution boundary always
go through an explicit submodule, never an aggregate (`CLAUDE.md` §2.7 item 4).

## 3. The zero-trust boundary

A shared helper that depends on a third-party package (e.g. OCR → `pytesseract`) **must not**
enter the clean-core `helpers/`; it belongs to `blackwattle` and loads lazily.
Promotion to "shared" respects the zero-trust boundary — see
[`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality-EN.md) and `CLAUDE.md` §7.4.

## 4. Traceability

`METHODOLOGY.md` §6 (traceability is a graph, not a chain) · `DOCTRINE.md` D-12 (five controlled
vocabularies) · `NFRQ-ORG-01`, `NFRQ-ORG-02`, `NFRQ-ORG-04`.
