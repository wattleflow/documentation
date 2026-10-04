<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-08 — Emitted-document confidentiality: a declared marking, no residue

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Security — confidentiality |
| **Enforcement** | **not measured** — `wem_lint` does not cover this class; declared blind spot (D-11) |
| **Legal frame** | *Privacy Act 1988* (Cth), APP 11 (`LITERATURE.md` key 64). The protective marking follows PSPF, which has **no key** in `LITERATURE.md` — see §5 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |
| **Raised by** | `HLRQ-19` `BR-02`, `BR-03` — recovered from existing code, not designed ahead of it |

> **Demarcation.** [`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md) governs what enters the
> **audit record**. This entry governs the **document the framework hands to people outside the
> process** — an office file, not a log line.

## 1. Statement

A component that emits a document in an office format (Word, PDF) **leaves no residue** of its
source or of the producing environment in the document's metadata beyond fields it declares, and
**carries a protective marking** that is visible on every page, on by default, and absent only by
an explicit choice.

Two witnesses exist in `blackwattle`: the Word converter and the PDF converter both clear metadata
by default; only the Word converter stamps a marking.

## 2. Acceptance criteria

1. **Declared metadata.** The component names the metadata fields it writes. Every other field is
   empty or carries a neutral value — **including values inherited from a built-in template**
   (creation and modification dates, revision).
2. **Marking on every page.** The marking is placed where it repeats per page (a section header),
   not once in the body.
3. **Default is marked.** Omitting the marking requires an explicit empty value; a missing or
   incomplete configuration yields the default marking, never an unmarked document.
4. **The marking is attributable.** Who or what chose the marking value — a configuration key, the
   document's own metadata, a classification component — is recorded on the document or in the
   audit record ([`NFRQ-OBS-02`](NFRQ-OBS-02-audit-fields.md)), without disclosing content
   ([`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md)).
5. **`[M]`** Count of office-format emitters without a declared metadata policy — **diagnostic,
   not a gate**. Charter: [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md).

## 3. Verification

Review of the emitter, and inspection of one emitted file's core properties and headers.

Checked on the Word converter by unittest in `blackwattle/tests/converters/` (criterion = this
entry §2 · Python 3.11.15, `python-docx` 1.2.0, Linux WSL2):

| criterion | Word converter |
|---|---|
| c.1 | **passes** — core fields empty or configured; no template dates, thumbnail, application name or `rsid` marks. Retained: an empty bibliography part |
| c.2 | **passes** — every section header |
| c.3 | **passes** — without the key the marking is `UNCLASSIFIED`; empty string omits it |
| c.4 | **fails** — the value comes from workflow configuration or the built-in default, and nothing records which |

The PDF converter is **not verified** under this entry. Machine measurement (c.5) is **not
implemented** — declared blind spot (D-11), not coverage.

## 4. Rationale

A document outside the process is outside every control the framework has: whatever its metadata
says is disclosed to every later reader, and whatever marking it lacks will not be added downstream.
Metadata is also the channel a user does not see when reviewing the visible text, which is why it
is cleared by default rather than on request.

Criterion 3 mirrors [`NFRQ-SEC-07`](NFRQ-SEC-07-emission-authorisation.md) c.3: a default that
fails open turns an incomplete configuration file into a disclosure.

## 5. Open

1. **PSPF is not a registered reference.** The marking values and their order follow the code's
   classification enumeration; the register cannot yet say which PSPF release that reflects.
   Adding the key is an append to `LITERATURE.md` (D-12).
2. **Derived marking.** For Word the marking comes from configuration with an `UNCLASSIFIED` default;
   whether it should be derived from the
   document is open (c.4).
3. **Scope.** Written for office formats because both witnesses are office formats. Whether it
   extends to every emitted file (CSV, JSON) is not claimed — a norm wider than its witnesses is an
   aspiration (D-05).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-13) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
