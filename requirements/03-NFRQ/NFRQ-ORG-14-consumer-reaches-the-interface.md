<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-ORG-14 — A consumer reaches a converter only through its interface

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability — modularity, modifiability |
| **Enforcement** | **not measured** — `wem_lint` has no rule for it; declared [blind spot](../../GLOSSARY.md#slijepa-pjega) (D-11). Baseline: `2026-10-05-converter-implementations-register` §3 (code reading, not run) |
| **Reference frame** | Parnas, [information hiding](../../GLOSSARY.md#information-hiding) [P-11]; [ISO](../../GLOSSARY.md#abbr-iso)/IEC 25010 modularity |
| **Raised by** | `FRQ-CNV-24.9` *(proposal)*; counterpart of `NFRQ-ORG-13` on the caller's side |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A [module](../../GLOSSARY.md#viseznacnost-modul) outside `converters/` that reads or writes a format for which a converter exists shall reach
that format only through the converter's interface (`convert`, `document`, `parse`, `config`) and the
API the converter's own requirement names. It shall not import or call the converter's parser, an
engine parser for that format, or any other internal class of the converter. A format with no
converter is out of scope; its factory parsers and formatters remain the way in.

## 2. Acceptance criteria

1. **No parser past the boundary.** No module outside `converters/` imports the `PARSER` of any
   converter, or an engine parser for a format that has a converter. *(machine-checkable — import graph)*
2. **Interface calls only.** Calls on a converter instance outside `converters/` are limited to the
   interface of `FRQ-CNV-24.1` and the API named in the converter's requirement.
   *(machine-checkable — [AST](../../GLOSSARY.md#abbr-ast), where the instance's type is statically known)*
3. **Collaborators through the strategy.** A collaborator (for example, a converter for images) is
   handed to the converter's strategy at construction, never called by the consumer. *(by review)*
4. **The consumer register is written down.** Every consumer of a converter appears in the register
   with its conformance, not derived on demand from the code. *(by review)*

## 3. Verification

Import-graph and AST analysis of `blackwattle/src/wattleflow` excluding `converters/`, against the list
of converters and their `PARSER` declarations. **Blind spots (D-11):** instances whose type is not
statically known (criterion 2), dynamic imports, and callers outside `src/`.

## 4. Rationale

| [principle](../../GLOSSARY.md#princip) | implication |
|---|---|
| A module is a boundary around a [decision](../../GLOSSARY.md#odluka) that may change (P-11) | a consumer that calls the parser binds itself to the decision the converter was meant to hide |
| One way to a format (`NFRQ-ORG-08`) | two consumers of the same format through two parsers drift apart in limits, errors and output |
| A collaborator belongs to the conversion, not the caller (`HLRQ-24` `BR-24-05`) | a consumer that bypasses the strategy also bypasses the collaborator, so a scanned page loses its text |

## 5. Open

1. **Identifier and category.** `ORG-14` is provisional; entry into the register requires a documented change (D-03).
2. **Scope beyond converters.** Whether the rule extends to other contexts with an internal part
   (driver and connection, processor and blackboard) is undecided, as for `NFRQ-ORG-13` §5 t.1.
3. **A converter's document type as a consumer's content type** is allowed by `FRQ-CNV-24.9` §01; whether
   that counts as reaching the interface is open (`FRQ-CNV-24.9` §15 t.3).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-05 | First entry, raised by `FRQ-CNV-24.9`; proposal — **the identifier is provisional**. |
