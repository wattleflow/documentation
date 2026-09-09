# FRQ-BBD-15.1 — Generički blackboard

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `BBD`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericBlackboard(Wattleflow, IBlackboard, Generic[Item], ABC)` — dijeljeni radni prostor između pipelinea i spremišta; uz njega `BlackboardState`, `BlackboardAction` i tablica `TRANSITIONS` |
| **Sestrinski** | [`FRQ-PIP-15.2`](FRQ-PIP-15.2-pipeline.md) (piše na platno) · [`FRQ-REP-15.7`](FRQ-REP-15.7-repository.md) (prima flush) · [`FRQ-STR-15.4`](FRQ-STR-15.4-strategy.md) (`StrategyCreate`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/blackboard.py` (231 linija) |

## 1. Predmet

Blackboard je **platno između dvije brzine**: pipeline proizvodi rezultate po stavci, a spremište
ih prima u serijama. Bez njega bi svaka transformacija sama odlučivala kada i kamo piše, pa bi
promjena spremišta bila izmjena svakog pipelinea.

Klasa drži tri stvari i ništa više:

| član | uloga |
|---|---|
| `_canvas: Item` | radno platno; generički parametar — specijalizacija bira oblik |
| `_repositories: list[IRepository]` | odredišta na koja `flush` odašilje |
| `_strategy_create: StrategyCreate` | kako nastaje nova stavka na platnu |

Platno se izvana vidi **samo za čitanje** (`canvas` vraća `MappingProxyType`) — tko želi promjenu,
prolazi kroz `write`/`delete`, gdje je promjena vidljiva automatu i auditu.

Klasa je **apstraktna u šest točaka**: `count`, `clean`, `create`, `delete`, `flush`, `read`,
`write`. Generički sloj dakle propisuje *ugovor i životni ciklus*, a ne *politiku* — koliko je
stavki na platnu i što flush znači, zna specijalizacija.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi blackboard iz konfiguracije | `strategy_create`, `canvas`, preset ključevi |
| **A2** | `GenericPipeline` — piše rezultat transformacije | `facade` i pozivatelj |
| **A3** | `GenericProcessor` — vlasnik ciklusa; traži `flush` i `clean` | granica ciklusa |
| **A4** | `IRepository` — odredište | prima `write` pri flushu |
| **A5** | `GenericBlackboard` — predmet ovog zahtjeva | platno i stanje |

Spremište je akter jer ga `flush` poziva; **platno nije akter** — ono je stanje, ne sudionik.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira blackboard → `__init__` (`BR-15-01`) |
| **EV02** | spremište se pridružuje → `REGISTER` |
| **EV03** | A2 piše rezultat → `WRITE` |
| **EV04** | A3 zatvara ciklus → `FLUSH` |
| **EV05** | oporavak iz snimke → `LOAD` |
| **EV06** | kvar → `FAIL` |
| **EV07** | kraj životnog ciklusa → `CLEAN` / `__del__` |

## 4. Preduvjeti

1. `strategy_create` je instanca `StrategyCreate` — provjereno `assert`-om **prije** poziva
   `super().__init__`, dakle prije nego objekt postoji.
2. `canvas` je predan konstruktoru; generički sloj mu ne propisuje tip.
3. Barem jedno spremište je registrirano prije prvog `flush`-a — inače flush nema odredište.
4. `defer_flush` je postavljen (zadano `True`): platno se puni kroz ciklus i prazni na kraju.

## 5. Normalan tok

1. **EV01** — konstruktor prihvaća `fmt` kao alias za `formatting` (zatečeni naziv koji nikad
   nije stizao do loggera), postavlja `defer_flush=True` ako ga pozivatelj nije zadao, pa
   prosljeđuje **cijeli** `**kwargs` naviše (`HLRQ-15` §4 t.2).
2. `PresetDecorator` preuzima ostatak ključeva; od te točke `__getattr__` razrješava
   konfiguracijska imena kroz preset, a ne kroz `__dict__`.
3. Konstruktor prijavljuje `Constructor/Started` pa `Constructor/Completed` — obje na `DEBUG`
   (`NFRQ-OBS-01`), jer konstrukcija nije jedinica posla.
4. **EV02** — spremište ulazi u `_repositories`; platno je od tada u sinkronizaciji s odredištima.
5. **EV03** — `write(pipeline, facade)` stavlja rezultat na platno. Platno je *prljavo* dok se ne
   isprazni.
6. **EV04** — `flush(caller)` odašilje sadržaj platna svakom registriranom spremištu i vraća
   platno u čisto stanje.
7. **EV07** — `__del__` prvo provjerava je li `_preset` uopće postavljen; ako `__init__` nije
   dovršio, destruktor **odmah odustaje** umjesto da kroz `__getattr__` maskira pravu iznimku.
   Inače: `Delete/Started` → `clean()` → oslobađanje strategije i preseta → `Delete/Completed`.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `strategy_create` nije `StrategyCreate` | `AssertionError` prije konstrukcije | objekt ne nastaje s neispravnom strategijom |
| `__init__` padne prije `_preset` | `__del__` odustaje, `__getattr__` diže običan `AttributeError` | prava iznimka nije maskirana |
| pristup nepostojećem imenu nakon konstrukcije | `PresetDecorator.__getattr__` odlučuje | konfiguracijska imena i atributi klase dijele jedan ulaz |
| `flush` bez registriranog spremišta | ovisi o specijalizaciji — generički sloj je apstraktan | **nije propisano** (vidi §11 t.2) |
| iznimka unutar `__del__` | `error(...)` u `try`, a i taj poziv je u `try/except Exception: pass` | destruktor nikad ne diže (`HLRQ-15` §4 t.4) |
| prijelaz koji tablica ne dopušta | automat odbija — **ako ga specijalizacija ima** | vidi §11 t.1 |

## 7. Rezultat

Rezultati transformacije skupljaju se na jednom mjestu i odlaze u spremišta u serijama, na granici
ciklusa koju određuje procesor. Pipeline ne zna koliko spremišta postoji ni kojeg su tipa;
spremište ne zna koji ga je pipeline proizveo. Zamjena spremišta je izmjena konfiguracije
(`BR-15-01`).

## 8. Kriteriji prihvaćanja

1. `Wattleflow` prethodi `Generic[Item]` u popisu baza, pa `IWattleflow` sjeda ispred `Generic` u
   MRO — bez toga se `GenericBlackboard + IOriginator` ne linearizira. ✅
2. `canvas` vraća **nepromjenjiv** pogled; izravna izmjena izvana nije moguća. ✅
3. `strategy_create` se provjerava prije `super().__init__`. ✅
4. `__del__` preživi neuspjelu konstrukciju bez maskiranja izvorne iznimke. ✅
5. Modul deklarira `__all__`; `__slots__` pokriva sva četiri člana. ✅
6. Import closure modula je `stdlib ∪ wattleflow` (`NFRQ-SEC-03`). ✅
7. Tablica prijelaza pokriva `FAIL` iz svakog neterminalnog stanja i `CLEAN` iz svakog stanja
   (`BR-15-06`). ✅
8. Automat se stvarno primjenjuje na svaki prijelaz platna. ❌ — vidi §11 t.1

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `class GenericBlackboard(Wattleflow, IBlackboard, Generic[Item], ABC)` + komentar iznad | redoslijed potvrđen; `LargeBlackboard` se linearizira |
| 2 | pregled `canvas` propertyja | `MappingProxyType(self._canvas)` |
| 3 | pregled `__init__` | `assert` je prva naredba |
| 4 | pregled `__del__` / `__getattr__` | `object.__getattribute__` u `try`, rani `return` |
| 5 | pregled modula | `__all__ = ["BlackboardAction", "BlackboardState", "GenericBlackboard"]`; `__slots__` = 4 imena |
| 6 | pregled uvoza | `abc`, `enum`, `types`, `typing`, `collections.abc` + `wattleflow.*` |
| 7 | prebrojavanje `TRANSITIONS` | `FAIL` iz `IDLE`/`READY`/`DIRTY`; `CLEAN` iz sva četiri |
| 8 | `command grep -n '_fsm' processors/src/wattleflow/blackboards/*.py` | `small`, `large`, `bundle` imaju automat; **`claude` ga nema** |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` i `processors` radna stabla 2026-08-27, CPython 3.11 (Linux/WSL2).
**Mjereno stablo:** `concrete/blackboard.py` + `processors/src/wattleflow/blackboards/`.

## 10. Nefunkcionalni zahtjevi

Registar i obveze skupine: [`HLRQ-15` §6](../01-HLRQ/HLRQ-15-generic-layer.md#6-nefunkcionalni-zahtjevi).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Blackboard` je rezervirani primitiv; platno nije „mali repozitorij" i ne preuzima njegovu ulogu |
| `NFRQ-SEC-01` | platno ne poznaje spremište osim kroz `IRepository`; kvar drivera ne doseže platno |
| `NFRQ-SEC-02` | `__all__` izlaže tri imena; `TRANSITIONS` je izvan njega, a specijalizacija ga ipak uvozi — vidi §11 t.3 |
| `NFRQ-SEC-03` | modul je u clean core tieru; nijedan third-party uvoz |
| `NFRQ-OBS-01` | konstrukcija i destrukcija su `DEBUG`; jedinica posla je flush, i nju prijavljuje specijalizacija |
| `NFRQ-ORG-08` | životni ciklus i preset se ne prepisuju po specijalizacijama — žive ovdje |

## 11. Otvoreno

1. **Automat je izvezen, ali nije primijenjen u generičkoj klasi.** `BlackboardState`,
   `BlackboardAction` i `TRANSITIONS` postoje, ali `GenericBlackboard` ih **nijednom ne
   referira** — nema `_fsm`, nema `apply`, nema provjere prijelaza. Vlasništvo automata prepušteno
   je specijalizaciji, i tri od četiri je preuzimaju (`small`, `large`, `bundle`);
   `ClaudeBlackboard` nema automat uopće, pa nad njim `BR-15-05` ne vrijedi. Ovo je **asimetrija
   prema sestrama**: `GenericConnection`, `GenericDriver` i `GenericProcessor` grade automat u
   generičkoj klasi (`concrete/{connection,driver,processor}.py`). Odlučiti: podići automat u
   `GenericBlackboard` (jednako ponašanje, jedno mjesto) ili zapisati prepuštanje kao namjeru.
2. **Ponašanje `flush`-a bez spremišta nije propisano.** Metoda je apstraktna, a nijedan kriterij
   ne kaže je li prazan popis odredišta kvar, upozorenje ili tišina. Tri specijalizacije danas
   odgovaraju različito — nije mjereno, samo uočeno.
3. **`TRANSITIONS` nije u `__all__`, a uvozi se preko granice distribucije.**
   `processors/blackboards/bundle.py` radi `from … import TRANSITIONS`. Ime koje `__all__` ne
   izlaže je po `NFRQ-SEC-02` t.2 privatno; ili ulazi u `__all__`, ili specijalizacija gradi
   vlastitu tablicu.
4. **`Mapping[Item]` je neispravna anotacija.** `collections.abc.Mapping` prima dva parametra
   (`Mapping[K, V]`). Kod radi samo zato što `from __future__ import annotations` anotaciju nikad
   ne evaluira; `get_type_hints()` nad ovom klasom bi pao. Uz to `canvas` tvrdi *mapping*, dok
   generički parametar `Item` ne obvezuje specijalizaciju na mapu — `MappingProxyType` nad
   ne-mapom diže `TypeError`.
5. **`defer_flush` je ovdje zadano `True`, a `GenericProcessor` isti ključ vodi kao zastarjeli**
   (`flush_per_cycle`, uz `warning` pri uporabi). Blackboard ga dakle *postavlja* dok ga procesor
   *odbija*. Uskladiti — inače svaki blackboard tiho pali deprecation upozorenje u procesoru.
6. **`blackboards/__init__.py` je eager agregat** (`from .bundle import *` …), što poništava
   odgodu na razini modula (`DR-WFL-007` §4). Blackboards su na popisu preostalih paketa u
   `TODO.md`; ovaj zapis to samo potvrđuje kao zatečeno.
