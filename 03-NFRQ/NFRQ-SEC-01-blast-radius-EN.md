# NFRQ-SEC-01 — Blast-radius containment

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-07-22) |
| **Quality (25010)** | Security — confidentiality, integrity |
| **Enforcement** | `wem_lint` metric — **diagnostic**; a gate only through a DR |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

The expected loss from compromise is

> `E[L] = Σ P(compromise_i) · Blast(i) · V`

where `Blast(i)` is **reachability from `i`** in the dependency (and privilege) graph. The
architecture must **bound the blast radius per component**: decomposition and least privilege
reduce `Blast`, but **every new interface is an entry point** — the optimum is **interior, not at
the edge** (the MDL mirror; see [`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface-EN.md)).

## 2. Acceptance criteria

1. **`[M]`** `Blast(i)` = the share of the system reachable from `i` over the **transitive
   closure** of the dependency graph, including shared dependencies. **Ratio** scale;
   **diagnostic**.
2. **`[M]`** A hub with a `Blast` above a **calibrated** threshold (against an external
   criterion) is a candidate for compartmentalisation; **the threshold is not a gate until
   validated** (`[M]` charter, item 2).
3. No clean-core hub has **third-party code in its transitive `Blast`** — see
   [`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality-EN.md).
4. `Blast` includes **dynamic edges** (`ClassLoader`, YAML), not only static imports.

## 3. Verification

`wem_lint` metric (diagnostic); `WARNING` when a hub crosses the calibrated threshold. **A gate
only through a DR.** Criterion 3 is measured indirectly today, through `clean_core_imports`
(SEC-03) — a `Blast` measurement of its own is not implemented; declared blind spot (D-11).

## 4. Justification

| Principle | Implication |
|---|---|
| Least privilege / compartmentalisation (Saltzer–Schroeder) | a smaller `Blast` per compromise |
| Design rules / Net Option Value (Baldwin–Clark) | modularity is a **multiplier in the risk calculation**, not an aesthetic |
| Correlated breaches (log4j) | `Blast` over **transitive** dependencies, including shared third-party ones |
| Interior optimum (MDL mirror / Manadhata–Wing) | excessive decomposition ↑ interface count ↑ attack surface |

Scientific basis: `tools/Analiza.md` §5.5 (structure as a risk multiplier).
