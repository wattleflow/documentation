# Usklađenje s core v0.0.0.46 — zapis o izmjenama

| | |
|---|---|
| **Datum** | 2026-07-28 |
| **Opseg** | `wattleflow-workflow` (`src/`, `tools/`, pakiranje) + `documentation/` |
| **Povod** | core v0.0.0.38 → v0.0.0.46: sloj sučelja pretvoren u čiste ugovore |
| **Analiza** | [CORE-v38-v46-ANALYSIS.md](../analysis/CORE-v38-v46-ANALYSIS.md) |
| **Odluke** | [DR-WFL-005](../dr/DR-WFL-005-dictionary-absorbs-code-vocabulary.md), [DR-WFL-006](../dr/DR-WFL-006-core-dependency-pin-and-versioning.md) |
| **C-vektor** | [conformance/2026-07-28-workflow.yaml](../conformance/2026-07-28-workflow.yaml) |

---

## 1. Zašto

Core je u rasponu v0.0.0.38 → v0.0.0.46 postao **isključivo sloj ugovora**. Sva
funkcionalnost koja je živjela u sučeljima ispala je i morala se nadomjestiti u
`concrete/` sloju workflowa.

Jedna izmjena nosi gotovo sve posljedice: `IWattleflow` je iz **konkretne** klase
s `__init__`-om koji postavlja `self.name` postao **ABC s apstraktnim read-only
`name`** ([DR-COR-001](../../../core/documentation/DR-COR-001.md),
[DR-COR-002](../../../core/documentation/DR-COR-002.md)).

## 2. Mjereni učinak

| | prije | poslije |
|---|---|---|
| klase apstraktne na `name` | **37** | **2** |
| `import wattleflow.concrete` | `TypeError` | radi |
| `.py` u wheelu | **0** | 64 |
| paketa u wheelu | 0 | 11 |
| build warninga | 5 | 0 |
| ORG-03 prekršaji (TypeVar) | 2 | 0 |

Preostale dvije (`StateMachine`, `GuardedStateMachine`) čekaju odluku — vidi §7.

## 3. Izmjene u `src/`

**Identitet.** `AuditLogger(Wattleflow, ILogger)` otključao je 24 klase odjednom;
`Wattleflow` mixin dodan još u `ConnectionObserverInterface`, `DocumentAdapter`,
`DocumentFacade`, `GenericMemento`, `Orchestrator`, `LazyIterator`,
`LazyAsyncIterator`, `ThreadSafeObservable`.

**Uklonjeni `T` TypeVar** (core ga više ne izvozi; role-TypeVarovi su module-scoped):

| datoteka | bilo | sada | uloga |
|---|---|---|---|
| `concrete/document.py` | `Generic[T]` | `Generic[Content]` | payload dokumenta |
| `concrete/document.py` | `DocumentAdapter(Generic[T])` | `Generic[Adaptee]` | Adapter uloga |
| `concrete/blackboard.py` | `Generic[T]` | `Generic[Item]` | stavka na platnu |

Uloge se uvoze iz `wattleflow.core.transactional` — jedan kanonski termin po ulozi
(NFR-ORG-03 kriterij 6). Time je ORG-03 pao na nulu prekršaja.

**Ostalo.** `concrete/__init__.py` izvozi 5 novih simbola (`Wattleflow`, `Singleton`,
`LazyIterator`, `LazyAsyncIterator`, `ThreadSafeObservable`);
`document.py` više ne uvozi `from wattleflow.concrete import AuditLogger` (uvoz kroz
vlastiti paket u tijeku inicijalizacije) nego izravni modul; `concrete/_version.py`
obrisan.

Usput ispravljen `# noqa: E401` → `F401` u `helpers/config.py` i `config_adapter.py`:
E401 je „više uvoza u jednom retku", pa suppression na guarded-import fallbacku nikad
nije radio. Preostaje jedan pre-postojeći `E501` u `constants/audit.py` (dugi JSON
format-string; lomljenje bi naštetilo čitljivosti).

## 4. `tools/wem_lint.py` — v1.2.0 → v1.4.0

Alat se **nije mogao pokrenuti**: njegova pravila implementiraju `IStrategy`, pa su
i ona postala apstraktna na `name`.

**Popravci prema novom coreu**
* bootstrap `src/` na `sys.path` — alat radi iz repoa bez instalacije
* `Wattleflow` mixin na svih 6 klasa; uvozi se iz `wattleflow.concrete`, ne iz lokalne
  kopije (samoreferentnost — `dictionary.yaml: wem-lint`)
* `PyFileIterator(LazyIterator[Path])` — `IIterator` više ne nosi lijenu mašineriju
* `execute_strategy(caller, **kwargs)` — usklađeno s novim `IStrategyContext`

Posljedica je deklarirana u docstringu: **lint ne može mjeriti `concrete/` sloj koji se
ne uvozi.** Instrument dijeli sudbinu mjerenog; to je izbor, ne kvar.

**Usklađenje s doktrinom**

| zahtjev | izvor | izvedba |
|---|---|---|
| vektorski nalaz bez skalara | `vektorski-nalaz`, METHODOLOGY §3b/§6.1 | blok `== VECTOR ==`, brojači po (NFR, kind, strogost) |
| trojka reproducibilnosti | `trojka-reproducibilnosti` | blok `== REPRODUCIBILITY ==` |
| slijepe pjege deklarirane | `slijepa-pjega` | dimenzija `EXC` — 3 modula izvan opsega + 2 iz registra |
| akronimi ne kažnjavaju dok traje odluka | METHODOLOGY §9 | strogost se **čita iz registra**, ne iz koda ([DR-WFL-004](../dr/DR-WFL-004-acronym-identifier-casing.md)) |
| C-snimka | `c-snimka` | `--snapshot`; označava se `finding-vector` dok nije zeleno |

Popravljena je i doktrinarna rupa koju je alat nosio: `--quiet` je brisao INFO **prije**
vektora, pa su slijepe pjege nestajale i iz zapisa. Sada krati samo ispis detalja —
prekidač ne smije moći pretvoriti *„nije mjereno"* u *„mjereno čisto"*.

## 5. Registri i pakiranje

**Rječnik apsorbira vokabular koda** ([DR-WFL-005](../dr/DR-WFL-005-dictionary-absorbs-code-vocabulary.md)).
`tools/naming_registry.yaml` ukinut; sadržaj je blok `code:` u `dictionary.yaml`.
Provjera prije brisanja: **11/11 ključeva preneseno**. Akronimi ostaju **dva odvojena
ključa** — presjek skupova je prazan i mora ostati prazan.

**Pakiranje** ([DR-WFL-006](../dr/DR-WFL-006-core-dependency-pin-and-versioning.md)).
sdist v0.0.0.86 nije sadržavao **nijednu** `.py` datoteku; `wattleflow` je bio
nepinovan iako core više nije unatrag kompatibilan. Oboje popravljeno, uz invariant
koji veže `MANIFEST.in` i `packages.find`.

## 6. DR numeracija i verzioniranje zapisa

### 6.1 Shema oznaka

Uvedena shema **`DR-COR` | `DR-WFL` | `DR-PRC` | `DR-CAD`** — serija po projektu,
oznaka globalno jedinstvena bez središnjeg brojača. Odluka pripada seriji **onog
projekta čiji artefakt mijenja**; odluka donesena u coreu koja *posljedično* mijenja
workflow ostaje `DR-COR`.

Indeksi: [DR-WFL-INDEX.md](../dr/DR-WFL-INDEX.md),
[DR-INDEX.md](../../../core/documentation/DR-INDEX.md).

Pretraga je zatekla **tri paralelne sheme u tri repozitorija**. Autoritativna je bila
core serija (`DR-001…014` + indeks); ostalo su stariji nacrti istih odluka (v. §7 t.8).

| bilo | sada |
|---|---|
| `ADR-001…014` (core, datoteke i docstringovi) | `DR-COR-001…014` |
| `DR-ORG-04` / `ADR-ORG-04` | `DR-WFL-001` |
| `DR-ORG-06` / `ADR-ORG-06` | `DR-WFL-002` |
| `DR-ORG-07` / `ADR-ORG-07` | `DR-WFL-003` |
| `DR-015` (referenciran iz tri dokumenta, nikad zapisan) | `DR-WFL-004` |

**`ADR-nnn` je uklonjen u potpunosti** — 26 datoteka u četiri repozitorija
(`core`, `workflow`, `documentation`, `processors`), uključujući docstringove u
`core/src/wattleflow/core/` i `core/tools/wem_lint.py`. Core datoteke preimenovane
(`DR-001.md` → `DR-COR-001.md`), naslovi i unutarnje unakrsne reference ažurirani.

Oznaka je ostala samo na dva mjesta, oboje namjerno:

* **migracijske tablice** (`bilo → sada`) — bez stare oznake tablica prestaje biti mapa;
* **`PHILOSOPHY.,md`** — *„naslijeđeni paket `ADR-001…014` vodi se kao `DR-COR-001…014`"*.

Generičku **proznu** riječ „ADR" nisam dirao: `METHODOLOGIA.md` §412 izrijekom kaže da
starije formulacije taj naziv namjerno zadržavaju jer opisuju stare verzije, a rječnik
zabranjuje samo `ADR kao naziv novih zapisa`.

### 6.2 Verzija zapisa

Svaki od 20 zapisa dobio je polje `Verzija` i sekciju `## Povijest zapisa`. Razlika je
doktrinarna, ne kozmetička: **verzija pripada zapisu, ne odluci.** Rječnik kaže da je
`odluka` nepromjenjiva i da je kasnija *nadomješta, nikad ne prepisuje* — preimenovanje
oznake zato ne smije izgledati kao promjena odluke.

```
Status:  prihvaćen (2026-07-24)            <- stanje ODLUKE
Verzija: 2 (2026-07-28) — v1: 2026-07-24   <- stanje ZAPISA
```

Verzije su izvedene iz vremena: internog polja `Datum` za nastanak, mtimea za izmjenu.

| serija | v | prvi zapis | razlog v2 |
|---|---|---|---|
| `DR-COR-001…014` | 2 | 2026-07-24 | `ADR-0NN` → `DR-COR-0NN` |
| `DR-WFL-001` | 2 | 2026-07-02 | `DR-ORG-04` → `DR-WFL-001`; `ORG-NN` veza premještena u polje *Realizira* |
| `DR-WFL-002` | 2 | 2026-07-09 | `DR-ORG-06` → `DR-WFL-002` |
| `DR-WFL-003` | 2 | 2026-07-15 | `DR-ORG-07` → `DR-WFL-003` |
| `DR-WFL-004/005/006` | 1 | 2026-07-28 | nastali u ovom prolazu |

Kod svih 17 zapisa s v2 stoji izrijekom **„sadržaj odluke nepromijenjen"**. Polje
`Povijest` predložak je već predviđao; precizirano je u `## Povijest zapisa` jer
`DR-COR-001` ima zasebnu `## Povijest` o *odluci* (povučen prvotni prijedlog), a te se
dvije ne smiju pobrkati.

Oba indeksa dobila su stupce `v` i `prvi zapis`, pa se stanje vidi bez otvaranja zapisa.
Povijest je i dalje **ručno vođena** — v. §7 t.7.

## 7. Ostaje otvoreno

**Odluke koje čekaju**

1. **`StateMachine.name`** — [state_machine.py:47](../../../workflow/src/wattleflow/concrete/state_machine.py)
   i `:99` pišu `self.name = …`, a `name` je read-only property. Klasa traži
   instance-scoped ime; DR-COR-002 propisuje ime izvedeno iz tipa. Preimenovati u
   `label`, ili zapisati iznimku.
2. **`DocumentAdapter.adaptee`** — `IAdapter.__init__(adaptee)` je nestao; konstruktor
   zove nepostojeći `__init__`, `self._adaptee` se nikad ne postavlja.
3. **`Scheduler` je izgubio singleton semantiku** — `IScheduler` više ne nasljeđuje
   `ISingleton`; guard `if not hasattr(self, "_initialised")` je mrtav kod.
   Core docstring propisuje `class Scheduler(Wattleflow, IScheduler[Event], Singleton)`.
4. **`constants` u `code.domains`** — paket ne uvozi ništa iz wattleflowa (čisti list),
   pa lint daje 2 × `ORG-01 misfiled-leaf ERROR`. Greška **registra**, ne koda.
5. **Tri tag oznake na jednom commitu** (`v0.0.0.85/86/87` → `b26c5d0`). Dok stoje,
   prijelaz na `dynamic = ["version"]` nije moguć.

**Nalazi o valjanosti instrumenta (hipoteza H3)**

6. **Filter opsega skriva stvarni prekršaj.** `helpers/config.py` je izvan opsega jer
   uvozi `yaml` — a upravo on nosi ORG-01 ciklus `helpers → concrete.exception`.
   Clean-core filter i ORG-01 pravilo sudaraju se: filter briše modul prije nego ga
   pravilo vidi. Sada je barem **deklarirano** kao slijepa pjega, ali nije izmjereno.

7. **DR zapisi nisu pod verzijskom kontrolom.** `documentation/` je gitignoriran u core
   repozitoriju, `*/hr/*` u dokumentacijskom. METHODOLOGY §8.1 traži *„revizibilnu
   povijest uključujući povučene odluke"*, a natuknica `odluka` kaže da je odluka
   nepromjenjiva i da je kasnija **nadomješta, nikad ne prepisuje**. Nijedno nije
   provedivo nad nepraćenim datotekama. Nova serija `workflow/dr/` leži izvan `*/hr/*`
   pa je praćena; core serija nije.

8. **Zastarjeli nacrti odluka u tri repozitorija.** Popis u
   [DR-WFL-INDEX.md](../dr/DR-WFL-INDEX.md) §„Zapisi koji NISU u ovoj seriji".
   Najozbiljniji: `documentation/core/DR.md` tvrdi da je *korijen frameworka konkretan*
   — suprotno prihvaćenoj `DR-COR-001`. Čeka odluku o supersession-u.

## 8. Sljedeći korak

FR/NFR: novi zahtjev za konformnost korijenskom ugovoru identiteta (37 klasa ga nije
implementiralo, a nijedan postojeći NFR to ne pokriva). Oznaka nije dodijeljena —
`NFR-ORG-06` je proturječan: `NFR.md` ga nema, `METHODOLOGY` §11 i `dictionary.yaml`
ga spominju. Uskladiti prije dodjele.
