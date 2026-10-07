<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-16 — Nothing is installed or acquired from code at run time

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Security — integrity, authenticity; Reliability — maturity |
| **Enforcement** | **not measured** — `wem_lint` has no rule for it; declared blind spot (D-11). Baseline: `2026-09-29-install-commands-in-code` |
| **Reference frame** | Saltzer & Schroeder, *least privilege*, *economy of mechanism* [23]; NIST SP 800-53 `CM-7` (least functionality), `CM-11` (user-installed software), `SI-7` (integrity) |
| **Raised by** | `HLRQ-23` `BR-23-05` — documented rule of 2026-09-29: *never, in any case, may a library be installed from Python code* |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

No wattleflow component installs, upgrades or removes a library, package, executable or system
package at run time, and none downloads code. A dependency that is missing is **reported by naming
the requirement** (package and version range), never by a command to run. Data a component needs
to run — model weights, language data — has a **location named in configuration**; when it is
absent, the first start may fetch it into that location and nowhere else (documented decision,
2026-09-29). Code is never fetched.

## 2. Acceptance criteria

1. **No install call.** No `src/` module calls a package manager (`pip`, `python -m pip`,
   `pip.main`, `ensurepip`, `conda`, `mamba`, `uv`, `poetry`, `apt`, `apk`, `npm`) through
   `subprocess`, `os.system` or an API. *(machine-checkable — AST)*
2. **Messages name a requirement, not a command.** No string literal in `src/` (comments
   excluded) contains an install command; an error names the package and version range.
   *(machine-checkable — AST over string constants)*
3. **Data has a configured place.** A component that needs model or language data reads its
   location from configuration (`OCRConfig.model_dir` for PaddleOCR); a missing file is fetched
   there on first start and into no other place; a different location already loaded in the
   process is refused. A network transfer of **code** is never made. *(unit test with a fake
   engine; by review for the fetch itself)*
4. **A missing dependency fails early and named.** The first use of the component raises a named
   error that says what is missing, models and data included, and how to provide it; nothing is
   checked in advance (`NFRQ-SEC-17` criterion 3). *(unit test)*
5. **A container installs at build, not at start.** A `Dockerfile` may install pinned packages; an
   entrypoint, compose command or workflow does not. *(machine-checkable — text)*
6. **No waiver.** The criterion carries no `declared` or `documented` waiver; a finding is
   removed, not tolerated. *(by review)*

## 3. Verification

An AST scan of `src/` for calls and string constants matching criteria 1 and 2, plus a text scan
of `Dockerfile`, compose and shell files for criterion 5. **Blind spots (D-11):** a command
assembled at run time from parts, an install performed by a third-party library's own hook, and
data acquired by a library the component merely imports (PaddleOCR fetches its models on first
use, `FRQ-OCR-23.2` §6 c) — the AST cannot see the last two.

## 4. Rationale

| principle | implication |
|---|---|
| Code that installs code is an arbitrary-execution path | a component's blast radius ([`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md)) grows to the whole environment |
| Hash-pinned locks and an SBOM ([`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md) c.4) describe the environment | an install at run time makes the description false the moment it runs |
| Reproducibility (`D-10`) | the same run on the same image must use the same code |
| Least functionality (`CM-7`) | a converter converts; provisioning is the operator's act |

## 5. Open

1. **Advice in messages.** A message such as "install X" is not an install, but it points the
   reader at a command; criterion 2 forbids the command and keeps the requirement. Scope:
   the ~60 messages in `src/` at the time of writing are a worklist, not a waiver.
2. **Whether a hash or signature** of a model fetched on first start is mandatory is undecided;
   `DriverLanguageModel` (HLRQ-13) already acquires models through a driver declared in configuration.
3. **A `wem_lint` rule** (`SEC-16`) for criteria 1 and 2 needs a documented change ([`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md) c.5 for any gate).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-06 | Criterion 4: the named failure at first use replaces the advance `available()` check (author's decision). |
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-29) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
