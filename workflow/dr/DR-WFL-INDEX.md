# DR indeks — wattleflow-workflow (WFL)

Predložak polja: Status / Kontekst / Odluka / Ugovor / Cijena / Svjedočanstvo /
Registar (+ Povijest gdje se odluka mijenjala).
Načelo: proširenje vokabulara ili promjena ugovora traži DR, ne ad-hoc edit
(`dictionary.yaml → governance.change_mechanism: DR`).

## Shema oznaka

Serija po projektu; oznaka je globalno jedinstvena bez središnjeg brojača:

| prefiks | projekt | serija |
|---|---|---|
| `DR-COR` | wattleflow-core | `core/documentation/DR-*.md` (001–014) |
| `DR-WFL` | wattleflow-workflow | **ova serija** |
| `DR-PRC` | wattleflow-processors | otvorena 2026-08-20 (`DR-PRC-001`), **bez indeksa** |
| `DR-CAD` | wattleflow-cad | još ne postoji |

Odluka pripada seriji **onog projekta čiji artefakt mijenja**. Odluka donesena u
coreu koja *posljedično* mijenja workflow ostaje `DR-COR` (npr. `DR-COR-003`
Singleton → `concrete/`); workflow je bilježi kao ulaz, ne kao vlastitu odluku.

## Registar

| DR | Naslov | Status | v | prvi zapis |
|---|---|---|---|---|
| [001](DR-WFL-001-helper-capabilities.md) | Cross-cutting sposobnosti kao helper klase (routing, file-discovery) | prihvaćen | 2 | 2026-07-02 |
| [002](DR-WFL-002-distribution-locality.md) | Lokalnost distribucije, pakiranje i validacija opskrbnog lanca | aktivan | 2 | 2026-07-09 |
| [003](DR-WFL-003-guarded-optional-dependency.md) | Čuvana opcionalna ovisnost sa stdlib fallbackom | aktivan | 2 | 2026-07-15 |
| [004](DR-WFL-004-acronym-identifier-casing.md) | Pisanje akronima u identifikatorima koda | **otvoren** | 1 | 2026-07-28 |
| [005](DR-WFL-005-dictionary-absorbs-code-vocabulary.md) | Rječnik apsorbira vokabular koda; `naming_registry.yaml` ukinut | prihvaćen (t.1 mijenja `DR-WFL-020`) | 1 | 2026-07-28 |
| [006](DR-WFL-006-core-dependency-pin-and-versioning.md) | Pin na core ovisnost; verzioniranje ostaje ručno | prihvaćen | 1 | 2026-07-28 |
| [007](DR-WFL-007-lazy-aggregate-public-api.md) | Agregatni `__init__.py` razrješava imena odgođeno (PEP 562) | prihvaćen | 1 | 2026-08-04 |
| [008](DR-WFL-008-audit-record-content.md) | Sadržaj audit zapisa: `AU-3` kao allow-lista | prihvaćen | 1 | 2026-08-13 |
| [009](DR-WFL-009-foundation-owns-identity-and-exception-root.md) | Temeljni sloj posjeduje identitet objekta i korijen taksonomije iznimaka | prihvaćen | 1 | 2026-08-16 |
| [010](DR-WFL-010-packaging-allowlist-over-namespace-tree.md) | Pakiranje nad PEP 420 stablom: `namespaces = true` i allowlist | prihvaćen | 1 | 2026-08-16 |
| [011](DR-WFL-011-config-format-as-variant.md) | Konfiguracija: format je varijanta, ugovor pretrage je zajednički | prihvaćen (izmijenjen `DR-WFL-012`) | 1 | 2026-08-18 |
| [012](DR-WFL-012-config-split-by-distribution.md) | Konfiguracija se dijeli po distribucijama: JSON u jezgri, YAML u processorsu | prihvaćen | 1 | 2026-08-19 |
| [013](DR-WFL-013-oscal-requirement-category.md) | Kategorija `OSCAL` u registru zahtjeva; os je sposobnost, ne uloga klase | prihvaćen | 1 | 2026-08-21 |
| [014](DR-WFL-014-oscal-gate-before-construction.md) | OSCAL vrata provjeravaju prije konstrukcije; `__del__` ne prijavljuje na polusagrađenom objektu | prihvaćen | 1 | 2026-08-22 |
| [015](DR-WFL-015-compliance-layer-in-processors.md) | Sloj usklađenosti (OSCAL, PSPF) pripada `wattleflow-processors`; zasebna distribucija se povlači | prihvaćen | 2 | 2026-08-22 |
| [016](DR-WFL-016-vocabulary-follows-its-reader.md) | Vokabular koji jezgra ne čita seli u processors (`wattleflow.enums`) | prihvaćen (t.2 mijenja `DR-WFL-017`) | 1 | 2026-08-22 |
| [017](DR-WFL-017-enums-belongs-to-the-core.md) | `wattleflow.enums` je dijeljeno ime; dijeljeni paket nema `__init__.py` | **prijedlog** | 2 | 2026-08-22 |
| [018](DR-WFL-018-audit-levels-fields-and-volume.md) | Razina, polja i volumen audit zapisa; `NFR-OBS-*` i lint provedba | prihvaćen (t.5 mijenja `DR-WFL-021`) | 1 | 2026-08-23 |
| [019](DR-WFL-019-helper-fan-in-is-diagnostic.md) | Dijeljeni helper traži potrošača izvan police; prijava je statistika (`INFO`), ne presuda | prihvaćen | 2 | 2026-08-24 |
| [020](DR-WFL-020-code-criterion-leaves-the-dictionary.md) | Kriterij koda napušta rječnik; `tools/dictionary.json` je jedini izvor | prihvaćen | 1 | 2026-08-24 |
| [021](DR-WFL-021-audit-trace-follows-the-call-order.md) | Audit trag slijedi pozivni red; korak u lancu prijavljuje ulaz, granicu piše vlasnik jedinice | prihvaćen | 1 | 2026-08-24 |

## Prijelaz s prethodnih oznaka

| bilo | sada | napomena |
|---|---|---|
| `DR-ORG-04`, `ADR-ORG-04` | `DR-WFL-001` | `ORG-NN` u oznaci sugerirao je vezu 1:1 s `NFR-ORG-NN` koja ne postoji — veza se sada bilježi u polju *Realizira* |
| `DR-ORG-06`, `ADR-ORG-06` | `DR-WFL-002` | |
| `DR-ORG-07`, `ADR-ORG-07` | `DR-WFL-003` | |
| `DR-015` | `DR-WFL-004` | zapis nije postojao — bio je referenciran iz tri dokumenta |

`ADR` kao naziv **novih** zapisa je zabranjen (`dictionary.yaml →
viseznacnost-adr.forbidden_usages`); u povijesnim referencama se zadržava.

## Zapisi koji NISU u ovoj seriji

Sljedeće datoteke sadrže **starije nacrte** odluka koje su u međuvremenu prihvaćene u
core seriji. Nisu premještene ovamo jer bi udvostručile odluku; čekaju odluku o
supersession-u:

| datoteka | nacrt od | prihvaćeno kao |
|---|---|---|
| `workflow/concrete/DR.md` (`DR-001` Singleton) | 2026-07 | `DR-COR-003` |
| `workflow/hr/adr/DR-ORG-05-observable.md` (`DR-005`) | 2026-07 | `DR-COR-005` |
| `workflow/hr/adr/DR-007-iterator.md` (`DR-007`, `DR-008`) | 2026-07 | `DR-COR-007`, `DR-COR-008` |
| `core/DR.md` (`DR-001`, `DR-002`) | 2026-07 | `DR-COR-001`, `DR-COR-002` — **sadržajno proturječan**: tvrdi da je korijen konkretan, a prihvaćena odluka je apstraktni korijen |
| `processors/docs/adr/*` | — | identične kopije gornjih nacrta u trećem repozitoriju |

## Otvoreno

**DR zapisi nisu pod verzijskom kontrolom.** `documentation/` je gitignoriran u core
repozitoriju, `*/hr/*` u dokumentacijskom. `METHODOLOGY.md` §8.1 traži *„revizibilnu
povijest uključujući povučene odluke"*, a natuknica `odluka` kaže da je odluka
nepromjenjiva i da je kasnija *nadomješta, nikad ne prepisuje* — nijedno nije
provedivo nad nepraćenim datotekama. Ova serija (`workflow/dr/`) leži izvan
`*/hr/*` pa je praćena; core serija nije.
