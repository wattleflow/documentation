# NFRQ-ORG-05 — Encapsulating self-referencing helper methods

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-07-09) |
| **Quality (25010)** | Maintainability — modifiability, modularity, analysability |
| **Enforcement** | static (AST) check; c.3 shares `prohibited_standalone` with ORG-02. **Introduces no runtime.** |
| **Policy** | `CLAUDE.md` §2.9 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

Related **stateless** helper functions shared by **several classes or strategies** are wrapped in
a **qualified class** — not free module functions, not a generic `Helper`
([`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature-EN.md)) — which holds the associated **constants as
class attributes** and the **methods**. Within such a class:

* a method whose body references **its own class** (a class constant or a sibling method) goes
  through `cls` and is declared **`@classmethod`**;
* a method that **touches no member** of its class (a pure function of its arguments) stays
  **`@staticmethod`**.

A hard-coded own-class name (`ClassName.CONST`, `ClassName.sibling()`) inside a `@staticmethod`
is a **violation**: it breaks renaming and overriding.

## 2. Acceptance criteria

1. No `@staticmethod` references its enclosing class by hard-coded name
   (`<EnclosingClass>.<member>`) in its body; such a method must be a `@classmethod` using `cls`.
2. `@staticmethod` is permitted **only** when the body references no member of the enclosing
   class.
3. Member constants carry no redundant class-name prefix — `Layout.SUBDIR`, not
   `Layout.LAYOUT_SUBDIR`.

**Exception (c.1).** A method whose signature already binds a parameter named `cls` (a domain
type, e.g. `Attribute.mandatory(caller, name, cls: type, …)`, also used as the keyword `cls=`) is
**not** converted to a `@classmethod` — that would duplicate the name — until the parameter is
renamed (`cls` → `kind`/`expected`). That is a separate API decision; until then it remains a
`@staticmethod` and the lint recognises this as **deferred**, not as a violation.

## 3. Verification

Static AST check (c.1, c.2) plus `prohibited_standalone` (c.3, shared with ORG-02).
**Criteria 1 and 2 are not in `wem_lint` today** — a candidate for extension; until then a
declared blind spot (D-11). The build fails only once the criterion is automated.

## 4. Reference pattern and migration

Pattern: `MailAttachmentLayout` in `strategies/documents/mail.py` — `resolve`/`write` as
`@classmethod` (`cls.FLAT`, `cls.label`) alongside a pure `label` as `@staticmethod`.

Enforcement is a piecewise legacy migration. **The candidate list is derived by AST search, not
transcribed into this text** (D-13, `CLAUDE.md` §9); its state is tracked in
[`workflow/TODO.md`](../workflow/TODO.md). A separate form of the same rule — **private module
functions with their module constants** — is closed in `wattleflow-workflow` and
`wattleflow-processors`; the remainder is `wattleflow-cad`.

Constants in `constants/filetype.py` deliberately stay at module level: inside an `Enum` body
they would become enum members.

## 5. Justification

| Principle | Implication |
|---|---|
| Information Hiding (Parnas) | constants and sibling methods sit behind the class's qualified namespace, not scattered across the module |
| GRASP — High Cohesion, Pure Fabrication | related stateless helpers plus their constants form one cohesive unit |
| Open–Closed / Liskov | `cls` resolves the member through a subclass; a hard-coded name breaks overriding |
| Least Astonishment | the decorator choice **expresses intent**: `static` = pure, `classmethod` = uses class state |
| DRY / Once and Only Once | the class name is not repeated in the body → renaming happens in one place |
