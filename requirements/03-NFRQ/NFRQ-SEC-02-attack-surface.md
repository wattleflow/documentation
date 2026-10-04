<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-02 — Attack-surface minimality

| | |
|---|---|
| **Version** | v0.0.1 |
| **Quality (25010)** | Security; Maintainability — modularity |
| **Enforcement** | `wem_lint` metric over the AST + boundary manifest; **diagnostic** |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A component's attack surface is measured through the **methods, channels and data items exposed
across a trust boundary** (Manadhata–Wing). The public interface must be **minimal** — every
exposed method, channel and item is a cost — subject to the same **interior optimum** as
[`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md): too little decomposition ↑ blast, too much ↑
interface count.

## 2. Acceptance criteria

1. **`[M]`** Attack surface = the weighted sum of exposed methods / channels / data items across
   the boundary (of trust or of distribution). **Interval** scale (weights); **diagnostic**.
2. **A module's public interface is declared explicitly** (`__all__`); what is not exposed stays
   private.
3. **`[M]`** Growth of the attack surface **without** a fall in `E[L]` is a regression
   (Pareto-dominated).

## 3. Verification

Criterion 2 is the only enforceable one today, and it is **enforced by policy, not by a rule**: 
§2.7 requires `__all__` in every module and `__init__.py`. `wem_lint` does **not**
measure it — declared blind spot (D-11). Criteria 1 and 3 are metrics without validation, hence
diagnostics.

> **Import-time closure is attack surface.** The deferred aggregate keeps it
> small: resolving a name imports **only** the submodule that defines it. `import *` asks for
> everything and therefore pulls the whole tier — that is an honest contract, not an oversight.

## 4. Justification

| Principle | Implication |
|---|---|
| Attack surface metric (Manadhata–Wing 2011) | exposure = f(methods, channels, data) |
| Economy of mechanism (Saltzer–Schroeder) | a smaller interface is less to attack and less to verify |
| MDL mirror (Rissanen; Cilibrasi–Vitányi) | `L(modules) + L(interfaces)`: the optimum minimises total description length |
| Information Hiding (Parnas) | internal specifics do not cross the boundary |

Scientific basis: `workflow/Analiza.md` §4.5, §5.5 — **the cited sections do not exist** in that file, which carries §1, §2 and §7 only (declared gap, D-11; checked 2026-09-19). Until they do, this criterion rests on a source the reader cannot open (D-05).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.1 | 2026-10-02 | Initial version and documentation created |

