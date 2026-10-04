<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-10 — A secret enters at the process boundary and never leaves through a record

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration |
| **CIA** | **Confidentiality** |
| **Quality (25010)** | Security — confidentiality, accountability |
| **Enforcement** | AST for c.1 and c.2 (candidate rule `credential_default`); review for c.3–c.5; **not in `wem_lint`** — declared blind spot (D-11) |
| **Reference frame** | ISO/IEC 27002:2022 §5.17, §8.15 [61]; NIST SP 800-53 `IA-5(7)` *No embedded unencrypted static authenticators* |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

> **Demarcation.** [`NFRQ-SEC-09`](NFRQ-SEC-09-no-secret-in-version-control.md) governs the
> **repository**; this entry governs the **running process** — how a credential comes in and where
> it must not go out. [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md) governs the audit
> record in general; c.3 here is its application to credentials.

## 1. Statement

A component obtains a credential **only** through the configuration adapter and a secret resolver
(environment, `.env` file, cloud secret store) at construction time. A credential is **never** a
default value in a signature, a class attribute, a template file or a container image, and it
**never** appears in an audit record, an exception message, a metric label or a stack trace.

## 2. Acceptance criteria

1. **No credential default.** No parameter named `password`, `passwd`, `token`, `secret`,
   `api_key`, `private_key` (or ending in `_password`, `_token`, `_secret`, `_key`) has a non-empty
   string literal default anywhere in `src/`. *(machine-checkable — AST)*
2. **No credential class attribute.** The same names carry no non-empty literal as a class-level
   or module-level constant, except a key **name** (`KEY_PASSWORD = "password"`). *(machine-checkable — AST)*
3. **Records carry presence, not value.** An audit record or exception may say *that* a
   credential was supplied (`auth=api_key`) and never *what* it was; `repr`/`str` of a connection
   redacts credential fields. *(by review; AST for `repr` of connection classes is a candidate)*
4. **Resolution is declared.** Every connection's `ALLOWED` names the credential keys it accepts
   and the docstring names the resolver path (`.env` section, environment variable, secret store).
   *(by review)*
5. **A container image carries no credential.** A `Dockerfile` or compose file references
   `${VAR:?}` or a secret mount; no `ENV PASSWORD=` and no baked `.env`. *(machine-checkable — text)*

## 3. Verification

c.1, c.2 and c.5 by AST/text search on demand; c.3 and c.4 by review at the entry that introduces a
connection. A default credential in a constructor signature is the case c.1 exists for; the
current trees carry none (local scan of 2026-09-19, not published).

## 4. Justification

| Principle | Implication |
|---|---|
| Least privilege (Saltzer & Schroeder) [23] | a default credential grants the *default* privilege to every caller who forgot to configure one |
| P-23 | the boundary is the process start; a literal moves the boundary into the source |
| Accountability (25010) | a record that shows the value defeats the record's purpose: whoever reads the log holds the key |
| Reference standard: ISO/IEC 25010 — Security (confidentiality, accountability) | |

## 5. Boundary

Test fixtures under `tests/` may carry placeholder credentials that open nothing
(`john.smith@acme.test`, `change-me`); they are outside `src/` and outside c.1–c.2, and they are
still subject to [`NFRQ-SEC-09`](NFRQ-SEC-09-no-secret-in-version-control.md) c.3.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
