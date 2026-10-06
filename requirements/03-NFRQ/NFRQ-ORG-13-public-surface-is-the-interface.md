<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-13 — The public surface of a contract class is its interface

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — modularity, modifiability |
| **Enforcement** | **not measured** — `wem_lint` has no rule for it; declared blind spot (D-11). Baseline: `2026-09-29-converter-contract-verification` |
| **Reference frame** | Parnas, information hiding [P-11]; ISO/IEC 25010 modularity |
| **Raised by** | `HLRQ-CNV` `BR-CNV-03` — documented rule of 2026-09-29 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A class that is a **contract** other classes inherit (`ConverterBase`, `Conversion`) declares
which methods and properties every child must respect; **only those are public**. Every other
method or property of the contract and of its children carries the prefix `_`, including hooks
the children override. A child may add a public member only when its own requirement names it
as API.

## 2. Acceptance criteria

1. **Public ⊆ interface.** The public methods and properties defined in a subclass of a
   contract class belong to the contract's declared interface or to the child's declared API.
   *(machine-checkable — AST/introspection)*
2. **Hooks are private.** A method a base calls and a child overrides has the prefix `_`.
   *(machine-checkable — AST)*
3. **Class-level declarations** (`CONFIG`, `PARSER`, `FORMATTERS`, …) are constants of the
   contract, not methods, and are not counted. *(by definition)*
4. **The interface is written down** in the requirement of the contract (`FRQ-CNV`,
   `FRQ-CNV-24.2`), not derived from the code. *(by review)*

## 3. Verification

Introspection of `vars(c)` for every class inheriting the contract class, against the list in
the requirement. **Blind spots (D-11):** members inherited from a third-party base, dynamic
attributes, and callers outside `src/`.

## 4. Rationale

| principle | implication |
|---|---|
| A module is a boundary around a decision that may change (P-11) | a public helper is a promise to every caller; a private one is a decision that can move |
| Renaming costs are proportional to the public surface | 22 members outside the interface on eight classes are 22 promises nobody wrote down |
| A child obeys the parent's contract, not its accidents | hooks that are public look like API and get called |

## 5. Open

1. **Scope beyond converters.** The rule reads as general; whether it extends to other contract
   classes (`GenericPipeline`, `GenericDriver`) is undecided.
2. **Protected hooks.** A `_hook` overridden by a child is a convention; Python enforces
   nothing, and the AST check must not flag the override itself.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-29) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
