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
| `DR-PRC` | wattleflow-processors | još ne postoji |
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
| [005](DR-WFL-005-dictionary-absorbs-code-vocabulary.md) | Rječnik apsorbira vokabular koda; `naming_registry.yaml` ukinut | prihvaćen | 1 | 2026-07-28 |
| [006](DR-WFL-006-core-dependency-pin-and-versioning.md) | Pin na core ovisnost; verzioniranje ostaje ručno | prihvaćen | 1 | 2026-07-28 |

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
