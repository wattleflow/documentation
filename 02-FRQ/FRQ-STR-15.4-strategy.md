# FRQ-STR-15.4 — Obitelj strategija

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `STR`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `Strategy(Wattleflow, IStrategy, ABC)` i četiri obitelji — `StrategyGenerate`, `StrategyCreate`, `StrategyRead`, `StrategyWrite` |
| **Sestrinski** | [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) (traži `StrategyCreate`) · [`FRQ-REP-15.7`](FRQ-REP-15.7-repository.md) (čitanje/pisanje) · [`FRQ-DRV-15.6`](FRQ-DRV-15.6-driver.md) (strategija piše kroz driver) |
| **Izvedba** | `workflow/src/wattleflow/concrete/strategy.py` (64 linije) |

## 1. Predmet

Strategija je **zamjenjivi algoritam jedne operacije**. Cijeli modul ima 64 linije i to je
namjerno: generički sloj ovdje ne donosi ponašanje nego **rječnik**. Sve četiri podklase imaju
identično tijelo — proslijedi na `execute` — a razlikuju se samo **imenom metode**:

| obitelj | metoda | potpis iznad `execute` | vraća |
|---|---|---|---|
| `StrategyGenerate` | `generate(caller, **kwargs)` | — | `ITarget \| None` |
| `StrategyCreate` | `create(caller, **kwargs)` | — | `ITarget \| None` |
| `StrategyRead` | `read(caller, identifier, **kwargs)` | dodaje `identifier` | `ITarget \| None` |
| `StrategyWrite` | `write(caller, facade, **kwargs)` | dodaje `facade` | **`bool`** |

**Zašto uopće četiri klase kad je tijelo isto.** Zato što ime metode nosi namjeru na mjestu
poziva: blackboard traži `StrategyCreate` i zove `create`, spremište traži `StrategyWrite` i zove
`write`. Da postoji samo `execute`, pozivatelj bi morao znati koju je vrstu strategije dobio, a
provjera tipa (`assert isinstance(strategy_create, StrategyCreate)` u blackboardu) ne bi imala
što provjeriti. Obitelj je dakle **tipska tvrdnja**, ne kod.

Jedina apstraktna metoda je `execute`; specijalizacija piše nju i ništa drugo.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi strategiju iz konfiguracije | preset ključevi, `repository` |
| **A2** | Pozivatelj primitiva (`IWattleflow`) — blackboard, spremište, pipeline | `caller` + operacijski argumenti |
| **A3** | `Strategy` — predmet ovog zahtjeva | algoritam |
| **A4** | `GenericDriver` — vanjski sustav kad ga operacija treba | prima poziv iz `execute` |

`caller` je **obvezni** prvi argument svake operacije: strategija po njemu zna tko je traži, i to
je ono što ulazi u audit trag (`DR-WFL-021`).

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira strategiju → `__init__` |
| **EV02** | blackboard traži novu stavku → `create(caller, **kwargs)` |
| **EV03** | spremište čita → `read(caller, identifier, **kwargs)` |
| **EV04** | spremište piše → `write(caller, facade, **kwargs)` |
| **EV05** | pipeline traži izvedeni sadržaj → `generate(caller, **kwargs)` |

## 4. Preduvjeti

1. Specijalizacija je implementirala `execute` — inače je klasa apstraktna.
2. `caller` je `IWattleflow`.
3. Strategija koja treba spremište ili driver dobiva ga kroz konstruktor odnosno `kwargs`;
   generički sloj to **ne propisuje** (vidi §11 t.2).

## 5. Normalan tok

1. **EV01** — konstruktor samo otvara kooperativni lanac (`super().__init__(**kwargs)`); ništa se
   ne izdvaja ni ne pamti. Sav `**kwargs` ide naviše (`HLRQ-15` §4 t.2).
2. **EV02–EV05** — pozivatelj zove metodu **svoje** obitelji; ona proslijedi na `execute` s
   `caller=` i pripadnim argumentom (`identifier` odnosno `facade`) kao imenovanim.
3. `execute` izvede algoritam i vrati `ITarget` ili `None`.
4. Kod `write` se rezultat prevodi u `bool` testom `is not None`.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `execute` nije implementiran | `TypeError` pri instanciranju | apstraktna klasa se ne gradi |
| algoritam ne nađe traženo | `execute` vraća `None` | `read`/`create` vraćaju `None`, `write` vraća `False` |
| `execute` digne iznimku | **prolazi nepromijenjena** kroz obiteljsku metodu | generički sloj ne omata — vidi §11 t.1 |
| `execute` vrati `False` iz `write` obitelji | `False is not None` → **`True`** | neuspjeh se prijavljuje kao uspjeh — vidi §11 t.3 |

## 7. Rezultat

Pozivatelj je dobio ishod operacije ne znajući kako je izvedena, a tip strategije koju drži
jamči da je pozvao operaciju koja za nju ima smisla. Zamjena algoritma je izmjena konfiguracije
(`BR-15-01`) i ne dira nijednog pozivatelja.

## 8. Kriteriji prihvaćanja

1. `execute` je jedina apstraktna metoda cijele obitelji. ✅
2. Svaka obitelj dodaje **samo** ime i potpis, bez vlastite logike. ✅
3. Sve četiri prosljeđuju `caller` kao imenovani argument. ✅
4. Modul deklarira `__all__` s pet imena; import closure je `wattleflow.*` + `abc`. ✅
5. `StrategyWrite.write` vraća `True` točno kad je pisanje uspjelo. ❌ — vidi §11 t.3
6. Kvar strategije izlazi kao `StrategyException` (`BR-15-09`). ❌ — vidi §11 t.1
7. Klase deklariraju `__slots__`. ❌ — vidi §11 t.4

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | `command grep -n '@abstractmethod' concrete/strategy.py` | jedna pojava |
| 2 | pregled četiriju tijela | svako je jedan `return self.execute(...)` |
| 3 | pregled potpisa | `caller=caller` u sva četiri |
| 4 | pregled modula | `__all__` = `Strategy`, `StrategyCreate`, `StrategyGenerate`, `StrategyRead`, `StrategyWrite` |
| 5 | pregled `write` | `return self.execute(...) is not None` |
| 6 | `command grep -c 'StrategyException' processors/src/wattleflow/strategies/documents/*.py` | ~41 mjesta omata **svako za sebe** (`TODO.md` klaster D3) |
| 7 | `command grep -n '__slots__' concrete/strategy.py` | nema pojave |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` i `processors` radna stabla 2026-08-27, CPython 3.11 (Linux/WSL2).
**Mjereno stablo:** `concrete/strategy.py` + `processors/src/wattleflow/strategies/`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Strategy` je rezervirani primitiv; *cross-cutting* sposobnost (rutiranje, digest) je **helper**, ne strategija |
| `NFRQ-SEC-02` | `__all__` izlaže pet imena; obitelj je javna, `execute` je ugovor |
| `NFRQ-SEC-03` | clean core tier; third-party (OCR, PDF) živi **isključivo** u specijalizacijama u processorsu (`CLAUDE.md` §7.4) |
| `NFRQ-ORG-08` | 41 ručno prepisan `try/except → StrategyException` je otvoreni DRY klaster (`TODO.md` D3) — kandidat za `@strategy_guard` |
| `NFRQ-OBS-01` | strategija ne otvara vlastitu jedinicu posla; trag joj daje pozivatelj preko `caller` |

## 11. Otvoreno

1. **Generički sloj ne omata kvar strategije.** `BR-15-09` traži da svaki sloj iznimku omota u
   vlastiti razred; ovdje se to ne događa, pa svaka od ~41 specijalizacije u processorsu piše
   vlastiti `try/except → StrategyException`. To je klaster **D3** iz `TODO.md`, a kandidat je
   dekorator `@strategy_guard` ili — u coreu — `GenericStrategy.call()`. Drugo dira autoritativni
   `core/` (`CLAUDE.md` §2.5) pa traži `DR-COR`; prvo ne dira i može ući ovdje.
2. **Kanal prema driveru nije u ugovoru.** Write-strategije danas dohvaćaju driver iz
   `kwargs.get("driver")`, create-strategije mu **nemaju pristup** — otvoreno pitanje zabilježeno
   u `TODO.md` (Tika refaktor). Generički sloj o driveru ne govori ništa, pa je mehanizam stvar
   dogovora između pozivatelja i specijalizacije.
3. **`write` prijavljuje neuspjeh kao uspjeh.** `self.execute(...) is not None` znači da
   `execute` koji vrati `False` (legitimna vrijednost za „nisam zapisao") daje `write() == True`.
   Ispravan test je istinitost rezultata, ne razlika od `None` — ili `execute` u `write` obitelji
   mora imati zaseban ugovor povratne vrijednosti. Nije mjereno koliko write-strategija vraća
   `False`; potencijal za tihi gubitak podatka postoji i bez toga.
4. **Nema `__slots__`** ni u jednoj od pet klasa, iako `HLRQ-15` §4 t.5 to traži za klasu koja
   drži stanje. Bazne strategije stanje ne drže, ali specijalizacije ga drže i nasljeđuju
   `__dict__` — slot se ne može uvesti odozdo ako ga baza nije otvorila.
5. **`ITarget | None` u potpisu `generate`/`create` nije obvezujuće.** Vraćanje `None` je
   legitiman ishod „nisam ništa proizveo", ali nigdje nije razgraničeno od kvara. Pozivatelji ga
   danas tumače različito.
