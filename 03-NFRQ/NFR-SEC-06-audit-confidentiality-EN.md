# NFR-SEC-06 — Audit-record confidentiality

| | |
|---|---|
| **Status** | **Accepted** (2026-08-13) |
| **Quality (25010)** | Security — confidentiality |
| **Decision** | [`DR-WFL-008`](../workflow/dr/DR-WFL-008-audit-record-content.md) |
| **Enforcement** | review plus AST for c.1; **`wem_lint` does not measure this today** — declared blind spot (D-11) |
| **Legal frame** | *Privacy Act 1988* (Cth), APP 3 and APP 11; NIST `AU-3`, `AU-3(3)`; ISO/IEC 27002 §8.11 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](0-NFRQ-EN.md#language)) |

> **Demarcation.** This requirement governs **what must not enter the record**.
> [`NFR-OBS-01/02/03`](NFR-OBS-01-audit-levels-EN.md) govern the record's **readability and
> cost** — which record, at which level, with which fields, and how many times.

## 1. Statement

An audit record carries the **named fields of `AU-3`** — event type, time, place, source,
outcome, subject identity.

**Document content and personal data enter only as a *measure* or an *identity*** — type, size,
count, identifier, digest — **never as a value**.

## 2. Acceptance criteria

1. **No splat.** `**kwargs` is **not forwarded** into a logging call: a component's **input
   surface** (`NFR-ORG-07`, `ALLOWED`) and its **recording surface** are different sets. The
   criterion is shared with [`NFR-OBS-02`](NFR-OBS-02-audit-fields-EN.md) c.5.
2. **PII is bounded explicitly** — `AU-3(3)`, minimisation.
3. **Redaction depends on the destination.** A log crossing a trust boundary is
   **de-identified** (ISO/IEC 27002 §8.11); with multiple handlers, redaction is a property of
   the ***(record, handler)* pair**, not of the record alone.
4. The operating point is subject to
   [`NFR-SEC-04`](NFR-SEC-04-detection-operating-point-EN.md): **redaction that disables
   diagnosis leads to bypass and is not an improvement**.

## 3. Verification

Criterion 1 is **statically visible** (the splat appears in the AST) and is the only automatable
one — but `wem_lint` does **not** enforce it today; declared blind spot (D-11), not coverage.
Criteria 2–4 are by review.

## 4. Open

* **A structured record reaching the handler is a precondition for c.3.** Today `_log_msg`
  flattens the fields into text, so the handler cannot see the fields it would redact.
* **The `au-*` family is not in the NIST→ISM crosswalk**, so the OSCAL gate cannot rely on it.

Both are tracked in `DR-WFL-008` §Cost.

## 5. Why `NFR-ORG-07` is missing

Criterion 1 cites `NFR-ORG-07` (input surface / `ALLOWED`) — a requirement **measured by the lint
as an error** (`preset_allowed_declaration`) that nevertheless **has no entry in this register**.
Declared blind spot (D-11); opening the entry requires a DR (D-03). See
[`0-NFRQ`](0-NFRQ-EN.md) §ORG.
