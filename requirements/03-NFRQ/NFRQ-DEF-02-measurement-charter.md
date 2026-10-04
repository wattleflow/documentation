<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-DEF-02 — Charter for measurable `[M]` criteria

| | |
|---|---|
| **Role** | The conditions under which a criterion marked `[M]` may enter use |
| **Register** | [`0-NFRQ`](NFRQ-000-INDEX.md) |
| **Anchor** | ISO/IEC 15939 (measurement process) · `METHODOLOGY.md` §5.1 (validity protocol V1–V6) · `DOCTRINE.md` D-09 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A criterion marked **`[M]`** states a **metric**, not a boolean threshold. The catalogue stays
categorised by domain; `[M]` only marks measurement criteria so that metric methods can be
developed against them.

## 2. The charter

1. **The scale is declared.** The criterion states its scale type (nominal / ordinal / interval /
   ratio) and its unit. Only statistics valid for that scale are permitted, and **no aggregation
   across scales**.
2. **It is published as a diagnostic** — value plus blind spots — and **never as a pass/fail
   gate**, until validated **out-of-sample** against an external criterion (change cost [h],
   defect density).
3. **The graph includes every edge** — static, relative and dynamic (`ClassLoader`, YAML) — and
   declares its blind spots (as with the fan-in lower bound in [`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality.md)).
4. **Reach is computed over the transitive closure** (propagation cost, blast radius), not over
   direct edges.
5. **Promoting an `[M]` metric to a gate requires a documented change** with a validation dossier.

## 3. Why

| Principle | Implication |
|---|---|
| Representational theory of measurement (Stevens; Krantz et al.) | a claim may use only the operations its scale admits |
| Goodhart / Campbell | a measure that becomes a target stops being a measure — hence the ban on gating before validation |
| ISO/IEC 15939 | measurement is a process with defined inputs, procedure and validity status |
| D-09 (vector, not scalar) | the output is a vector across dimensions; there is no overall score |

## 4. Holders

[`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication.md) c.3 · [`NFRQ-OBS-03`](NFRQ-OBS-03-audit-ownership-volume.md) c.5 · [`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md) c.1–2 · [`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md) c.1, c.3 ·
[`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md) c.6 · [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point.md) c.2 · [`NFRQ-SEC-05`](NFRQ-SEC-05-adversary-model.md) c.2.

## 5. Status

No `[M]` measure is validated out-of-sample today, so **none is a gate**. Declared, not
suppressed (D-11).
