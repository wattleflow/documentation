# NFRQ-SEC-04 — Detection operating point and psychological acceptability

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-07-22, sketch) |
| **Quality (25010)** | Security; Usability |
| **Enforcement** | **by review** — control telemetry; **not a CI gate** |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

Every security control is a **classifier**: a false positive costs convenience (a legitimate user
blocked), a false negative costs a breach. A control chooses an **operating point on the ROC**,
not "stricter".

A mechanism that gets bypassed **does not protect**: **psychological acceptability**
(Saltzer–Schroeder, principle P8) is originally a **security principle**, not an after-the-fact
compromise.

## 2. Acceptance criteria *(largely by review)*

1. The threshold is set from the **cost ratio** and the **base-rate prevalence**, not ad hoc
   "stricter".
2. **`[M]`** Friction is measured: the **bypass rate** (the primary KPI — an *outcome*, not an
   attitude), plus steps and time per task. **Ratio** scale; **diagnostic**.
3. An improvement is a shift of the **ROC curve** (a better sensor or better context), **not** a
   move of the threshold along it.

## 3. Verification

Control telemetry (FP/FN, bypass rate) plus a review of the operating point. **Not a CI gate.**
No telemetry exists today — criterion 2 is an aspiration (D-05), not a measure in use.

## 4. Where this rule already applies

* [`NFRQ-OBS-01`](NFRQ-OBS-01-audit-levels-EN.md) — an `ERROR` that is not a failure is a false
  alarm and spends the attention a real failure needs; same anchor (signal detection theory).
* [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality-EN.md) c.4 — redaction that disables diagnosis
  leads to bypass and **is not an improvement**.

## 5. Justification

| Principle | Implication |
|---|---|
| Signal detection theory | threshold = `P(signal)/P(noise) · C_FP/C_FN`; moving the threshold is movement **along** the ROC, not improvement |
| False-positive paradox (base rate) | at low attack prevalence even a good classifier yields mostly false alarms |
| Compliance budget (Beautement–Sasse–Wonham 2008) | exceeding the effort budget produces **bypass**, not resistance |
| Psychological acceptability (Saltzer–Schroeder; Adams–Sasse 1999) | the user is not the adversary; bypass is a signal of a **bad operating point** |

Scientific basis: `tools/Analiza.md` §5.4.
