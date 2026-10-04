<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-11 — The class is the unit of code; a module function is a declared exception

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — modularity, analysability, modifiability |
| **Enforcement** | AST; **not in `wem_lint` yet** — declared blind spot (D-11). Candidate rule: `module_function_declaration` |
| **Policy** | `STANDARDS.md` §2.9 · extends [`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

Code in `src/` is written as **classes**. A helper is a method of the class that reads it, or —
when several classes read it — a method of a **qualified class** that holds its constants
([`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md), `STANDARDS.md` §2.9 t.4).

A **module-level function** exists only when **both** hold:

1. it is part of the module's public surface — named in `__all__` — or is a dunder the language
   protocol names (`__getattr__`, `__dir__`; PEP 562);
2. its reason for being a function is one the **language or an external contract** imposes, from
   the closed list below, and that reason is **declared next to the definition**.

A function that fails either test is a hidden member of some class: its name does not say whose
knowledge it carries, it cannot reference `cls`, it escapes inheritance and override, and it
stays behind when the class is renamed.

## 2. Exceptions — the closed list of reasons

| reason | when it applies | example |
|---|---|---|
| `pep562` | the language protocol names the function and requires it at module level | `__getattr__` / `__dir__` of a lazily aggregating package |
| `decorator` | a decorator factory or decorator: the language applies it by name at class-definition time, before any instance exists | `oscal_policy(...)`, `oscal_driver(...)` |
| `entry-point` | a signature a runtime, build tool or standard fixes as a function | `main()` of a script, `PyInit_<module>`, a `setup.py` hook |
| `standard-signature` | an external API contracts a callable with a fixed signature that a method cannot satisfy without a bound instance | a `logging` filter function, a `signal` handler, a `key=` callable published for reuse |

Any other reason enters this table **with a documented change** (D-12). "Shorter to write" is not a reason.

**Out of scope:** `tests/`, `tools/`, `examples/` and constants modules (`Enum` bodies and plain
module constants — [`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md) §4).

## 3. Acceptance criteria

1. No module-level function in `src/` that is **not** in `__all__` and is not a PEP 562 dunder.
   *(machine-checkable — AST)*
2. Every module-level function in `__all__` carries, on the line above `def`, the declaration
   `# NFRQ-ORG-11: <reason>` with a reason from §2. *(machine-checkable — AST + comment)*
3. The reason vocabulary is the table in §2; a reason outside it is a finding, not a new
   category. *(machine-checkable)*
4. A helper read by exactly one class is a method of that class; a helper read by several is a
   method of a qualified class. *(by review; the candidate list is derived by AST — one reader
   or many — never transcribed into text, D-13)*

## 4. Verification

AST search over `src/wattleflow` of each distribution: top-level `FunctionDef`, membership in
`__all__`, the declaration comment. **Not automated today** — until `wem_lint` carries the rule
the check is run on demand and the result is a finding-vector, never a claim of cleanliness
(D-11, `CONFORMANCE.md` §9). Migration of the existing sites is tracked in
`workflow/TODO.md`; the count is read from the search, not from this text.

## 5. Justification

| Principle | Implication |
|---|---|
| Information hiding (Parnas) | a function without an owning class hides *whose* decision it implements; the class is the unit that names the secret |
| Open/closed [69] & substitution [67][68] | a method can be overridden and reached through `cls`; a free function cannot — [`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md) c.1 already forbids the hard-coded class name for that reason |
| Attack surface ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md)) | an importable function outside `__all__` is surface without a contract; `__all__` is the declared surface (`STANDARDS.md` §2.7) |
| Analysability | one rule for where a helper lives ([`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality.md) for the package, this entry for the class) removes a per-case judgement |
| Reference standard: ISO/IEC 25010 — Maintainability (modularity, analysability, modifiability) | |

## 6. Boundary

This is a rule about **where code lives**, not about paradigm purity: a `@staticmethod` on a
qualified class is a function in every respect but its address. Where a framework or standard
demands a bare callable, §2 applies and the declaration records it — the rule does not fight the
language.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
