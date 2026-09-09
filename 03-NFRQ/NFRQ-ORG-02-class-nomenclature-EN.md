# NFRQ-ORG-02 — Class nomenclature

| | |
|---|---|
| **Status** | In force |
| **Quality (25010)** | Maintainability — analysability; PEP 8 (naming conventions) |
| **Decisions** | [`DR-WFL-005`](../04-DR/DR-WFL-005-dictionary-absorbs-code-vocabulary.md) (code vocabulary) · [`DR-WFL-004`](../04-DR/DR-WFL-004-acronym-identifier-casing.md) (**open** — acronym casing) |
| **Enforcement** | `wem_lint` — `base_family_membership` (**error**), `prohibited_standalone` (**error**), `acronym_case` (`warning`, suspended) |
| **Criterion** | `tools/dictionary.json` — `domains`, `subjects`, `operations`, `acronyms`, `qualifiers`, `bases` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

## 1. Statement

A pipeline class name is composed of **controlled, orthogonal facets in a fixed grammar**, so
that the name encodes the **input→output contract** rather than the internal mechanism.

**Grammar:** `Pipeline + <Subject> + <Operation | ToTarget> + [Qualifier]`

| Facet | Content | Note |
|---|---|---|
| **Subject** | format, source or domain | the leading facet; **equal to the canonical subject of the domain package** containing the class (case-insensitive) → `pipelines/pdf/PipelinePDFExtractText` |
| **Operation** | closed verb vocabulary: `Extract`, `Redact`, `Clean`, `Repair`, `Translate`, `Write` | converters express the source→target relation with the connective `To` (`CSVToSheets`) |
| **Qualifier** | engine, language, variant (`Spacy`, `Stanza`, `En`, `Hr`) | optional |

The scientific basis for the *Subject-first* order is
[`NFRQ-APX-01`](NFRQ-APX-01-facet-order-EN.md).

* The universal verb `transform()` is **not** in the name: a token predictable for every class
  carries zero discriminating information.
* The `Pipeline` marker is reserved for classes implementing the public pipeline contract.
* Generic role nouns (`Manager`, `Helper`, `Utility`, `Piece`, `Sheet`, `Parser`) are forbidden
  **standalone**; they are permitted as a *qualified* facet (`DriverS3UriParser`).

**Two regimes.** The grammar above governs **pipeline** classes. **Helper** classes are governed
only by: no `Pipeline` prefix, no standalone generic role noun, domain-qualified
(`SheetNestingPlacement`, not `Placement`).

## 2. Acceptance criteria

1. A class inheriting a pipeline base matches
   `Pipeline<Subject>(<Operation>|To<Target>)(<Qualifier>)?`; any other shape is a violation.
2. The `Pipeline` prefix appears **only** on classes implementing the public pipeline contract.
3. Every facet token belongs to its registered vocabulary; an unregistered token fails the build
   until the vocabulary is extended **through a DR**.
4. Acronym casing follows **PEP 8** — all letters uppercase (`PDF`, `CSV`, `RDF`).
   **Suspended to `WARNING`** pending
   [`DR-WFL-004`](../04-DR/DR-WFL-004-acronym-identifier-casing.md); see §4.
5. The Subject facet equals the canonical subject of the domain package containing the class
   (case-insensitive).
6. No class name consists solely of a forbidden generic role noun.

## 3. Verification

Linting of name grammar and vocabulary; the build fails on criteria 1, 2, 3, 5, 6.
Criteria 1/3/5 require a **controlled vocabulary register** — `tools/dictionary.json`, under DR
governance (`DR-WFL-005`). Synonym collapsing for the current catalogue:
`Redaction→Redact`, `Cleanup`/`Correction→Clean`, `Fix→Repair`.

## 4. Open — acronym casing

`DR-WFL-004` **has not been decided**. Criterion 4 is therefore suspended: the `acronym_case`
rule reports a `WARNING` under a declared waiver
(`acronym_identifier_casing.status: undecided`) and **does not fail the build**. Neither side
(`PDF` vs `Pdf`) is enforced by mass renaming — enforcing an undecided rule breaks the cascade
(D-02). The criterion text and the severity must stay in step; a divergence is a finding, not
the state in force.

## 5. Justification

| Principle | Implication for naming |
|---|---|
| Least Astonishment | the name is predictable from its facets, and the facets recoverable from the name |
| Faceted Classification (Ranganathan) | orthogonal axes avoid the combinatorial explosion of a single hierarchy |
| Information theory (Shannon) | constant tokens (`Transform`) are removed as noise |
| Information Hiding (Parnas) | the name expresses the external contract, not the wrapped implementation |
| Single fundamentum divisionis | one basis of classification per axis |
| DRY | one controlled term per facet; repeating the subject in both path and name is a deliberate trade-off for collocation ([`NFRQ-APX-01`](NFRQ-APX-01-facet-order-EN.md) §3.4) |

## 6. Traceability

Type variables — [`NFRQ-ORG-03`](NFRQ-ORG-03-typevar-nomenclature-EN.md).
Module placement — [`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality-EN.md).
Capabilities — [`NFRQ-ORG-04`](NFRQ-ORG-04-crosscutting-capability-EN.md).
