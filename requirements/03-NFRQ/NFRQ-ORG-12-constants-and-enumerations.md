<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-12 — Constants and enumerations live in their designated modules

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — modularity, analysability, reusability |
| **Enforcement** | AST; **not in `wem_lint` yet** — declared blind spot (D-11). Candidate rule: `constant_placement` |
| **Policy** | `STANDARDS.md` §2.6, §2.9 · complements [`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md) and [`NFRQ-ORG-11`](NFRQ-ORG-11-classes-over-module-functions.md) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A value that names part of a **shared vocabulary** — an enumerated set, or a constant read by
more than one class — is defined once, in the module designated for it, and imported from there:

| kind | designated package | form |
|---|---|---|
| enumerated type read by several roles | `wattleflow.enums.<topic>` | an `Enum` subclass (`str` mixin where the value travels as text) |
| constant read by several classes | `wattleflow.constants.<topic>` | a module-level `UPPER_SNAKE` name in `__all__` |

It is not restated as a literal at the point of use: no string set in a filter, no bare string
compared against a record field, no second copy of the value in another module.

## 2. Boundary

**Designated is not the same as central.** The designated module of a vocabulary is the one whose
subject it names, and a vocabulary owned by a single unit belongs with that unit:

1. A constant read by **one** class is that class's own parameter: it stays a class attribute and
   is reached through `cls` (`STANDARDS.md` §2.9, [`NFRQ-ORG-05`](NFRQ-ORG-05-self-referencing-helpers.md)).
2. An enumerated type that is **part of one primitive's state machine** stays beside that machine,
   in the primitive's own module, together with its transition table. States, actions and
   transitions are one unit; splitting them puts half a machine in each of two files, and the
   specialisations that read them already import the primitive they inherit from.
3. `enums/` holds the vocabularies **no single unit owns** — those several roles emit or read
   (`Event`, `Measure`, `MetricTarget`).

Either moves to `constants/` or `enums/` when a second owner appears.

## 3. Acceptance criteria

1. An `Enum` subclass defined outside `enums/` is read by the unit that defines it, and by the
   specialisations of that unit only — otherwise it belongs in `enums/`. *(AST derives the reader
   set; the verdict is by review, per §2 t.2)*
2. No module-level `UPPER_SNAKE` assignment read by more than one module is defined outside
   `constants/`. *(machine-checkable — AST, reader count derived, D-13)*
3. A value that belongs to an enumerated set is referenced through its member, not as a literal.
   *(by review; a literal equal to a member value is a candidate, not a verdict)*
4. `enums/` and `constants/` are PEP 420 namespaces shared across distributions, so a module name
   exists in **one** distribution only, and a module lives in the lowest distribution whose code
   reads it (the dependency arrow blackwattle → workflow → core). *(machine-checkable)*
5. Every such module declares `__all__` (`STANDARDS.md` §2.7). *(machine-checkable)*

## 4. Verification

AST search over `src/wattleflow` of each distribution: `ClassDef` with an `Enum` base, module-level
`UPPER_SNAKE` targets and their importers, module names under `enums/` and `constants/` across
trees. **Not automated today** — run on demand; the result is a finding-vector, never a claim of
cleanliness (D-11, `CONFORMANCE.md` §9). Migration of existing sites is tracked in
`workflow/TODO.md`; the list comes from the search, not this text.

**Out of scope:** `tests/`, `tools/`, `examples/`.

## 5. Justification

| Principle | Implication |
|---|---|
| Single source of truth ([`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication.md)) | a vocabulary written in two places drifts; the second copy is the one nobody updates |
| Cohesion (Parnas, [`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality.md)) | what changes together lives together: a state machine's states, actions and transitions are one decision, and a central `enums/` would split it for no reader's benefit |
| Analysability | the set of values a field may take is read in one file, not reconstructed from call sites |
| Controlled vocabulary (D-12) | a new member is a visible edit to a designated module, reviewable as such |
| Reference standard: ISO/IEC 25010 — Maintainability (modularity, analysability, reusability) | |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: **Accepted** (2026-09-21) — introduced by  §2 corrected by its v2 (2026-09-23) |
