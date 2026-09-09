# NFRQ-ORG-03 — Type-variable nomenclature

| | |
|---|---|
| **Status** | Proposal (Draft) |
| **Quality (25010)** | Maintainability — analysability; PEP 8, PEP 484 (`TypeVar`, variance) |
| **Enforcement** | `wem_lint` — `typevar_role_vocabulary` (`warning`) |
| **Criterion** | `tools/dictionary.json` → `type_vars` (canonical role nouns + synonyms), shares `acronyms` with ORG-02 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A type variable's name names the **semantic role the type plays in the generic contract**, drawn
from a controlled vocabulary of roles — not the mechanism (the fact that it is a type). The name
encodes *what the type represents in the contract*, not *that it is a type*.

* **Grammar:** `<RoleNoun>` — one role noun from the register (`Key`, `Value`, `Message`,
  `Destination`, `Context`, `Result`, `State`, `Action`, `Vertex`, `Edge`, `Input`, `Output`,
  `Element`…). `KeyType` is a violation; `Key` is correct.
* **No mechanism suffixes** (`T`, `Type`, `_t`) — they carry zero discriminating information,
  since every type variable is a type (`YieldT` → `Output`, `SendT` → `Input`).
* **Bare `T` is reserved** for **one unbounded** "any element" parameter of a generic with no
  semantic specialisation. As soon as a parameter carries a role, it is named by that role.
* **A multi-parameter generic** names each parameter with a **distinct** role noun:
  `Generic[Key, Value]`, not `Generic[T, U]`.
* **Variance is not encoded in the name** — it is declared through
  `TypeVar(..., covariant=True)`, never through a `_co`/`_contra` suffix.

## 2. Acceptance criteria

1. Every `TypeVar` name resolves against the registered role vocabulary; an unregistered name
   fails the build until the vocabulary is extended **through a DR**.
2. No name carries a mechanism suffix (`T`, `Type`, `_t`). Exception: the bare identifier `T`.
3. A bare single letter is permitted only for **one** unbounded parameter within a single
   generic; two or more single-letter parameters in the same `Generic[…]` is a violation.
4. Acronym casing follows PEP 8. **Suspended to `WARNING`** pending `DR-WFL-004` — as in
   [`NFRQ-ORG-02`](NFRQ-ORG-02-class-nomenclature-EN.md) §4.
5. The name does not encode variance; variance goes solely through `TypeVar` arguments.
6. One canonical term per role; using a registered synonym instead of the canonical term is a
   violation.

## 3. Verification

AST check: it finds `TypeVar(...)` assignments (an `ast.Assign` whose value is an `ast.Call` on
the name `TypeVar`), extracts the target name and the `covariant`/`contravariant` arguments, and
checks them against criteria 1–6. The rule is `severity: warning` today — the current state is a
worklist; raising it to `error` requires a DR.

## 4. Collapsing worklist

`Yielded→Output` · `Sent→Input` · `Return`/`Returned→Result` · `Elem→Element`.

Representative renames:

* `ICoroutine[YieldT, SendT, ReturnT]` → `ICoroutine[Output, Input, Result]`. `Result` is not
  currently used in the signatures — either drop it (`Generic[Output, Input]`) or keep it with
  explicit use in the return type.
* `WattleType` (iterator element) → a register decision: tolerate it as a branded "element" term,
  or rename to the canonical `Element`. Until decided, the lint reports a `WARNING`.

Names already aligned (`Context`, `Result`, `State`, `Key`, `Value`, `Message`, `Destination`,
`Vertex`, `Edge`, `Action`) enter the register as canonical, unchanged.

## 5. Justification

| Principle | Implication |
|---|---|
| Information theory (Shannon) | the `T`/`Type` suffix is a constant token — its mutual information with the parameter's identity is zero |
| Least Astonishment | the contract is readable without opening the `TypeVar` definition |
| Single fundamentum divisionis | classification on one basis — the role, not the mechanism |
| DRY | one canonical term per role |
| Information Hiding (Parnas) | "yield" is a coroutine mechanism, "output" is a role |
| Faceted Classification (Ranganathan) | roles are orthogonal axes (key ↔ value, input ↔ output) |
