<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-09 — No secret in version control

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration: a credential enters at the start boundary, from an environment the repository does not track, and in readable form never becomes part of code, template or history |
| **CIA** | **Confidentiality** |
| **Quality (25010)** | Security — confidentiality |
| **Enforcement** | on-demand scanner over the tracked tree and the full history (local tool; the scan and its result are **not published**); **not in `wem_lint`** — declared blind spot (D-11) |
| **Reference frame** | Saltzer & Schroeder, *open design* [23]; ISO/IEC 27002:2022 §5.17 *Authentication information*, §8.12 *Data leakage prevention* [61]; NIST SP 800-53 `IA-5`, `SA-3(1)` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

No tracked file of any wattleflow repository, in any commit on any branch, carries the **value**
of a credential — a password, a token, an API key, a private key, a connection URL with embedded
credentials. A tracked file may carry the **name** of a key, a **placeholder** from the closed list
in §2 c.3, or a **reference** to where the value comes from.

## 2. Acceptance criteria

1. **Tree and history are clean.** The scanner reports no credential-shaped literal with a
   non-placeholder value over `HEAD` and over every blob reachable from any ref.
   *(machine-checkable; binary per finding)*
2. **Secret files are ignored by pattern, not by path.** `.env`, `.envrc`, `*.pem`, `*.key`,
   `id_*` match an ignore rule that holds at any depth, so a moved directory cannot expose them.
   *(machine-checkable — `git check-ignore`)*
3. **Placeholders are a closed list:** `change-me`, `<name>`, `${VAR}`, `user:password@host` in a
   URL-scheme docstring, and a value identical to its key name. Anything else with a credential
   key is a finding. *(machine-checkable)*
4. **An example file lists keys, never values.** `.env.example` beside every instance; every key
   the instance's compose file references appears in it. *(machine-checkable — set comparison)*
5. **A found secret is rotated before it is removed**, and the removal is recorded in the documentation; a
   history rewrite is a separate decision of the maintainer, never an agent's action.
   *(by review)*

## 3. Verification

The scanner runs on demand over the five repositories and writes a finding-vector into the
local, unpublished analysis directory with the reproducibility triple (tool, criterion, platform, measured tree, rules).
A green run is evidence for the date it ran, not a standing claim (`CONFORMANCE.md` §9). No CI runs
it.

## 4. Justification

| Principle | Implication |
|---|---|
| Open design (Saltzer & Schroeder) | protection must not depend on the secrecy of the mechanism — only on the key; a key in the repository is no key |
| P-17 — a boundary that hides a decision hides provenance | a secret in history has provenance that cannot be withdrawn: every clone carries it |
| P-21 — concatenation is not composition | a template with the value written in concatenates code and environment; the `${VAR:?}` contract composes them |
| Blast radius ([`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md)) | a leaked credential extends the blast radius of a repository read to every system the credential opens |
| Reference standard: ISO/IEC 25010 — Security (confidentiality) | |

## 5. Boundary

Constants that **name** a classification, a key or a field (`SECRET = "<uuid>"` in a protective
marking enum, `KEY_PASSWORD = "password"`) are vocabulary, not credentials; the scanner's
placeholder list recognises them. Encrypted secrets stored in a repository (SOPS, git-crypt) are
outside this entry — they would need their own decision.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
