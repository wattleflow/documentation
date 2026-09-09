# NFRQ-ORG-10 — A learned artefact as a decision criterion

| | |
|---|---|
| **Status** | Proposal (2026-09-05). No holder in code yet — the rule is written **before** the first model, not after |
| **Quality (25010)** | Maintainability (analysability, modifiability); Reliability (maturity) |
| **Enforcement** | **by review** plus the artefact manifest; **not a CI gate** — there is no CI (`CLAUDE.md` §4) |
| **`[M]` charter** | [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md) |
| **Operating point** | [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point-EN.md) — threshold, FP/FN cost ratio; not restated here (D-13) |
| **Category** | `ORG` chosen for want of a better axis; the register has no measurement/model axis. **Provisional** (D-12) — see §6 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A trained model is a **criterion that cannot be read**. Every other criterion in this project —
`tools/dictionary.json`, a template file, a rule — states its reasons in text a reviewer can
check. A weights file states none. That is not a reason to refuse one, but it is the reason it
carries obligations the others do not.

The rule is therefore not "no models". It is: **a learned artefact decides only what a
deterministic rule demonstrably cannot, and only while it carries its provenance.**

## 2. Acceptance criteria

1. **Deterministic first.** The learned path is reached only on input the deterministic detector
   has declared degraded. A model that runs on clean input, where a rule decides confidently,
   is refused regardless of its accuracy.
2. **It must beat the baseline it replaces**, on the same held-out set, by a margin the entry
   declares. A model that merely ties **loses**: it buys nothing and costs analysability.
3. **`[M]` Accuracy is measured out-of-sample on a REAL held-out set** — never on the synthetic
   corpus used during development. Scale: ratio; reported per class (precision, recall), never as
   one number (D-09). Diagnostic only; promotion to a gate needs a DR (`NFRQ-DEF-02` c.5).
4. **The artefact carries a manifest**: training-corpus digest, feature specification, library and
   version, split and seed, the metrics of c.3, and a date. The manifest is **versioned separately
   from the tool that runs it** (D-10) — a retrained model is a criterion change, not a code
   upgrade, and must not pass as one.
5. **Retraining from the recorded inputs reproduces the metrics** within a declared tolerance.
   A model whose training cannot be repeated is a finding, not an artefact.
6. **Opacity is declared, not hidden** (D-11). Every verdict a model produces is recorded together
   with the model identifier **and the deterministic evidence that was available** at the time, so
   a disputed verdict can be re-examined without the model.
7. **Locality.** The artefact and its libraries belong to `wattleflow-processors`, which is
   already outside the zero-trust scope by declaration (`CLAUDE.md` §7.4) — so `scikit-learn`,
   `numpy` and `scipy` there are **not** a violation of `NFRQ-SEC-03`. No clean-core distribution
   may depend on them, directly or lazily.
8. **The model file is data, not code.** Wheel `RECORD` integrity covers shipped modules; a model
   loaded from a configured path is outside it. Its digest is verified at load, or its absence is
   declared as a blind spot.
9. **Loading a model is never implicit.** Absent or unreadable, the component falls back to the
   deterministic path and says so at `WARNING`; it does not fail the run and does not silently
   guess.

## 3. Verification

| criterion | method | status |
|---|---|---|
| 1, 9 | code review of the call site — is the model reachable on non-degraded input? | reviewable today |
| 2, 3 | held-out evaluation against the deterministic baseline, both on the same set | **no real corpus exists** — aspiration (D-05) |
| 4, 5 | inspection of the manifest; a repeat run from the recorded inputs | no artefact exists yet |
| 6 | inspection of the audit record for `model id` + evidence fields | reviewable once holders exist |
| 7 | `wem_lint` `clean_core_imports` already refuses such an import in a clean-core tree | automated |
| 8 | digest check at load, or a declared blind spot | not implemented |

**Reproducibility triple (D-10):** tool — review plus the evaluation script named by the holder;
criterion — §2 above; platform — declared by the holder's entry. **Declared blind spot:** nothing
here is measured by `wem_lint` except c.7.

## 4. Why

| Principle | Implication |
|---|---|
| D-10 (criterion separate from tool) | a retrained model changes the criterion; versioning it with the code hides that |
| D-11 (declared blind spots) | a verdict that cannot be reproduced by reading is a blind spot, and must be named as one |
| D-09 (vector, not scalar) | one accuracy number is not a verdict; per-class rates are |
| Goodhart / Campbell (`NFRQ-DEF-02`) | a metric that becomes a target stops measuring — hence c.3's ban on gating before validation |
| Occam / cost of abstraction | c.2: an opaque mechanism that does not outperform a readable one is a net loss |
| P-19 (conformance ≠ quality) | a model passing its own metric proves conformance to a declared criterion, nothing more |

## 5. Where this rule already applies

* [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point-EN.md) — every classifier chooses a point
  on the ROC; a model is a classifier and inherits that obligation whole.
* [`NFRQ-ORG-09`](NFRQ-ORG-09-external-standard-EN.md) — a specialisation over an external
  mechanism exposes that mechanism's model rather than re-deriving it; a learned artefact is such
  a mechanism.

## 6. Open

1. **The category is provisional.** `ORG` is the nearest fit, but the axis this belongs to —
   measurement and learned artefacts — does not exist in the register. Either it is opened
   through a DR (D-12) or this stays under `ORG`. Cost of a new axis: one more category to keep
   coherent; cost of staying: `ORG` widens beyond structure.
2. **The margin in c.2 is not fixed.** Whether "beats the baseline" means one percentage point
   or ten depends on the cost of a wrong verdict, which is a per-holder question, not a global
   constant.
3. **No corpus policy.** Training on real correspondence engages *Privacy Act 1988* (Cth), APP 3
   and APP 11 — already the project's own anchor (`DR-WFL-008`). Whether such a corpus may be
   retained at all, and under what retention rule, is undecided and blocks c.3.
