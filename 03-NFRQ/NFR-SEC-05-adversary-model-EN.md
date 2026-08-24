# NFR-SEC-05 — Adversary model and investment discipline

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-07-22, sketch) — a theoretical frame; it guides decisions and is **not enforced in CI** |
| **Quality (25010)** | Security |
| **Enforcement** | review of architectural decisions (threat model) |
| **`[M]` charter** | [`NFR-DEF-02`](NFR-DEF-02-measurement-charter-EN.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](0-NFRQ-EN.md#language)) |

## 1. Statement

Security risk **is not exogenous variance**, as option-pricing models assume: the adversary
observes the architecture and chooses the point of attack **after** the decision.

The correct formalism is **Stackelberg** (the defender leads, the adversary best-responds) → the
criterion is **minimax over the best response**, not expected value.

> **Attack surface does not suffer attacks — it attracts them.**

## 2. Acceptance criteria *(theoretical / by review)*

1. Security decisions are assessed by **minimax** over the adversary's best response.
2. **`[M]`** Investment does not exceed the **Gordon–Loeb** ceiling (~`1/e · E[L]`). **Note:**
   `1/e` holds for **independent, non-adaptive** threats — against a strategic adversary it is
   **not invariant**.
3. Every estimate of `P(compromise)` carries an **expiry** (non-stationarity) **shorter than the
   architectural decision it justifies**; it is renewed.
4. The cost of a security failure must fall on **the decision-maker** (incentives); otherwise no
   index fixes a misallocation.

## 3. Verification

Review of architectural decisions (threat model) plus a Gordon–Loeb check of the budget. **Not a
CI gate.** No estimate of `P(compromise)` is recorded with an expiry today (c.3) — declared blind
spot (D-11).

## 4. Tension with [`NFR-SEC-01`](NFR-SEC-01-blast-radius-EN.md)

SEC-01 computes `E[L]` as an **expected value**; this requirement holds that expected value
underestimates risk against a strategic adversary. That is **not a contradiction but an order of
precedence**: `E[L]` is a diagnostic input, minimax is the decision criterion. Where they
diverge, the decision is recorded.

## 5. Justification

| Principle | Implication |
|---|---|
| Stackelberg (leader–follower) | the defender moves first; optimise minimax, not `E[·]` at a given variance |
| Endogenous threat volatility | option models (Baldwin–Clark) **understate** security risk — variance is a function of the decision |
| Gordon–Loeb (2002) | investment ≤ ~`1/e ≈ 37%` of expected loss; the most vulnerable asset is not necessarily the priority |
| Economics of security (Anderson 2001) | many failures are a problem of **incentives**, not of measurement |

Scientific basis: `tools/Analiza.md` §5.3, §5.6.
