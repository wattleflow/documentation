<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-07 — Component input surface (`ALLOWED`)

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — analysability; Security — confidentiality (attack surface, [`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md)) |
| **Enforcement** | `wem_lint` — `preset_allowed_declaration` (`error`; the forwarding finding is a `warning`) |
| **Criterion** | `tools/dictionary.json` → `preset_allowed` (declaration name, scope, synonyms, forwarding) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A component whose optional runtime configuration is carried by `PresetDecorator` declares the keys it
permits in **one** place: a class attribute named exactly `ALLOWED`, holding a **list of key names**.
The component's **input surface** is that list; nothing outside it is configuration.

Keys that are not declared are discarded. Declaring the surface correctly is therefore the whole of
the component's configuration contract: a declaration the decorator cannot resolve leaves the
component with an empty or wrong whitelist, and the configuration its caller passes is **not
applied**. The framework reports the discarded keys at runtime (§5.2), but the component keeps
running on its defaults. That state is wrong by construction and statically visible, which is why the
lint raises it as `error`.

## 2. Acceptance criteria

1. **Name.** The attribute is named `ALLOWED`. A synonym (`__allowed__`, `__ALLOWED__`,
   `ALLOWED_KWARGS`, `allowed`) is never resolved, so it is a violation. Finding `preset-allowed-name`.
2. **Scope.** The declaration is a **class** attribute. A module-level declaration is never found
   through `type(parent).ALLOWED` and no subclass can extend it. Finding `preset-allowed-scope`.
3. **Type.** The value is a `list` of `str`. A bare string or any other type is a violation: iterated,
   a string yields single characters, and the whitelist becomes a set of letters. Checked statically
   where the value is a literal (`preset-allowed-type`) and at run time, where `PresetGate.resolve`
   raises `TypeError` for a declaration that is not a list of `str`.
4. **Inheritance.** The declaration is **unioned across the MRO**: a subclass declares only what it
   adds and never names its parent. A key already declared by a base is not repeated.
5. **Framework keys.** The keys the framework consumes itself — `allowed`, `formatting`, `handler`,
   `level`, `name` (`PresetGate.FRAMEWORK`) — reach the preset and are never reported as undeclared.
   The list is closed: a legacy spelling is not added to it.
6. **Dependency key `driver`.** A component that declares `driver` in `ALLOWED` takes a driver, and
   takes it as **required**: the workflow factory resolves `configuration.driver` (a name) to the
   driver object and fails when it is missing ([`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) §5). A
   component that can run without a driver does not declare it. This is the only reading of
   `ALLOWED` outside the decorator that is not input validation (`_driver_context`).

The runtime declares **names only**; any value passes. The C++ port declares typed keys (`Key<T>`) and
rejects a value of the wrong type at construction. The difference is intentional (author decision
2026-09-13, `cpp/README.md`).

## 3. Verification

AST check over the distribution (`wem_lint`, criteria 1–3, plus the forwarding warning of §5.1). The
run-time check of criterion 3 is in `PresetGate.resolve`. Criteria 3–5 and the decorator's contract
are covered by `workflow/tests/test_preset.py` (`unittest`, `ARCHITECTURE.md`). Four deliberate
mutations of `preset.py` (type check removed, MRO union reduced to the target class, a legacy key
added to `FRAMEWORK`, undeclared keys stored) each fail the suite. Criteria 1, 2 and 6 are covered
by the lint only (criterion 6 by review).

## 4. Justification

| Principle | Implication |
|---|---|
| Single source of truth | the permitted keys are stated once, on the type, not once per call |
| Fail visibly | a declaration that cannot be resolved is a build error, not a configuration that quietly does not apply |
| Least privilege | the input surface is the declared list; the recording surface of the audit record is a different set ([`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md) c.1) |

## 5. Open

1. **`allowed=` override.** `PresetGate.resolve` accepts an explicit `allowed=` that *replaces* the
   whole declaration, while the lint warns about a class that declares `ALLOWED` and still forwards
   `allowed=` (`preset-allowed-forwarded`). One call site passes it (`api` `youtube.py`), redundantly.
   Decision: forbid the override, or keep it and drop the lint warning.
2. **Level for an undeclared key.** `PresetGate.report_unknown` warns and discards; the converters'
   `refuse_unknown` raises `ConfigurationError`. Decision: one behaviour for the framework.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | Entry opened for `NFRQ-ORG-07`: criteria 1–6 (name, scope, type, inheritance, framework keys, dependency key `driver`). |
