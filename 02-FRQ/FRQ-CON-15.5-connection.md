# FRQ-CON-15.5 — Generička konekcija

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `CON`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericConnection(ConnectionObserverInterface, Generic[Connection], ABC)` — pristup vanjskom sustavu; uz njega `ConnectionObserverInterface`, `ConnectionState`, `ConnectionAction`, `TRANSITIONS` |
| **Sestrinski** | [`FRQ-DRV-15.6`](FRQ-DRV-15.6-driver.md) (radi kroz konekciju) · [`FRQ-CON-13.1`](FRQ-CON-13.1-huggingface.md) (specijalizacija u processorsu) · `FRQ-PTN-15.14` *(nenapisan)* (`ConnectionManager`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/connection.py` (430 linija) |

## 1. Predmet

Konekcija je **jedina granica prema vanjskom sustavu** (`BR-15-07`). Sve iznad nje — driver,
spremište, pipeline — poznaje samo njezino sučelje, nikad protokol.

Klasa razdvaja dvije razine trajanja koje se u praksi stalno miješaju:

| razina | što je | tko je vodi |
|---|---|---|
| **engine / pool** | dugotrajan resurs; gradi se jednom | `create_connection()` ↔ `disconnect()` |
| **sesija** | kratkotrajan zahvat unutar engine-a | `connect()` kao context manager |

Ta podjela je razlog zašto automat ima osam stanja umjesto dva: `CREATED` znači „engine postoji,
sesije nema", `CONNECTED` znači „sesija je otvorena". Zatvaranje sesije vraća u `CREATED`, ne u
`CLOSED` — engine preživljava.

**Automat je ovdje u generičkoj klasi**, za razliku od blackboarda (`FRQ-BBD-15.1` §11 t.1).
Specijalizacija ga ne gradi nego ga **primjenjuje** — i to je jedina stvar koju generički sloj od
nje traži da radi sama, jer prijelaze `CONNECT`/`CONNECT_OK`/`CONNECT_FAIL`/`DISCONNECT` može
znati samo tijelo `connect()`.

Tri apstraktne metode, s izričitom podjelom odgovornosti nad automatom:

| metoda | tko vodi automat |
|---|---|
| `create_connection()` | **generički sloj** (`ensure_created`) — specijalizacija ga ne dira |
| `disconnect()` | **generički sloj** (`ensure_closed`) — isto |
| `connect()` | **specijalizacija** — ona jedina zna kada sesija počinje i završava |

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi konekciju iz konfiguracije | `connection_name`, `lazy_loading`, preset ključevi |
| **A2** | `ConnectionManager` — vlasnik registriranih konekcija | ime konekcije |
| **A3** | `GenericDriver` / strategija — traži sesiju | `connect()` / `context()` |
| **A4** | `GenericConnection` — predmet ovog zahtjeva | engine i sesija |
| **A5** | `IObserver` — pretplatnik na promjene | `update(owner, **kwargs)` |

Vanjski sustav (baza, broker, poslužitelj) **nije akter** — ne pokreće ništa; sudionik je toka i
izvor kvara.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira konekciju → `__init__` |
| **EV02** | gradnja engine-a → `CREATE` → `CREATE_OK` / `CREATE_FAIL` |
| **EV03** | A3 otvara sesiju → `CONNECT` → `CONNECT_OK` / `CONNECT_FAIL` |
| **EV04** | sesija se zatvara → `DISCONNECT` |
| **EV05** | A2 traži operaciju → `request(action=Operation.Connect/Disconnect)` |
| **EV06** | rušenje engine-a → `CLOSE` → `CLOSE_OK` / `CLOSE_FAIL` |
| **EV07** | oporavak iz `FAILED`/`CLOSED` → `RESET` |
| **EV08** | pretplatnik se prijavljuje → `subscribe(observer)` |
| **EV09** | kraj životnog ciklusa → `__del__` |

## 4. Preduvjeti

1. `connection_name` je zadan i nije prazan — inače `ConnectionException` **prije** nego objekt
   postane upotrebljiv (`BR-15-02`).
2. Specijalizacija deklarira dopuštene konfiguracijske ključeve kroz `ALLOWED` class-atribut;
   `PresetDecorator` ih razrješava iz **tipa**, ne iz instance (`NFRQ-ORG-07`).
3. Specijalizacija je implementirala sve tri apstraktne metode.
4. Tajne dolaze kao reference razrješive kroz konfiguracijski lanac, nikad kao literal
   (`BR-15-08`).

## 5. Normalan tok

1. **EV01** — `connection_name` se izdvaja i provjerava; prazna vrijednost je kvar.
2. `PresetDecorator` preuzima konfiguraciju; automat se gradi u stanju `NEW`.
3. **EV02** — ako `lazy_loading` nije zatražen, `ensure_created()` se poziva odmah: `CREATE` →
   `create_connection()` → `CREATE_OK`. Kvar daje `CREATE_FAIL` i **ponovno diže** izvornu
   iznimku, s `DEBUG` tragom.
4. Konstruktor prijavljuje `Constructor/Completed` s imenom, stanjem i presetom — bez ijedne
   vrijednosti tajne (`BR-15-08`).
5. **EV03** — `connect()` (ili `context()`, koji ga omata) otvara sesiju. Prijelaze vodi tijelo
   specijalizacije: `CONNECT` prije otvaranja, `CONNECT_OK` na uspjeh, `CONNECT_FAIL` na kvar,
   `DISCONNECT` u `finally`.
6. Konekcija se može koristiti i kao `with` objekt: `__enter__` pamti context manager u
   `_context`, `__exit__` ga zatvara i **uvijek** ga poništava u `finally`.
7. **EV05** — `request(action=…)` prevodi `Operation.Connect` u `ensure_created()`, a
   `Operation.Disconnect` u `ensure_closed()`.
8. **EV09** — `__del__` prvo provjerava postoji li `_fsm`; polusagrađena instanca nema ni automat
   ni logger, pa bi prijava ondje zakopala pravu iznimku (`DR-WFL-014` t.4). Inače
   `ensure_closed()`.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `connection_name` prazan ili izostavljen | `ConnectionException` iz konstruktora | konekcija bez identiteta ne nastaje |
| `create_connection()` padne | `CREATE_FAIL` → stanje `FAILED`, iznimka se diže dalje | kvar je vidljiv i stanju i pozivatelju |
| `ensure_created` iz stanja iz kojeg `CREATE` nije dopušten | `ConnectionManagerException` s imenom stanja | automat se ne zaobilazi |
| `ensure_created` kad je engine već tu | tihi `return` | idempotentno |
| `ensure_closed` iz `CLOSED`/`NEW`, ili kad `CLOSE` nije dopušten | tihi `return` | idempotentno |
| `disconnect()` padne | `CLOSE_FAIL` → `FAILED`, iznimka se diže | rušenje koje nije uspjelo ne prijavljuje se kao uspjeh |
| stanje `FAILED` | `_ensure_created()` **odmah odustaje** | ne ponavlja se gradnja koja je već pala |
| oporavak | `reset()` vraća `FAILED`/`CLOSED` u `NEW` — tiho ako nije dopušten | ponovna gradnja je moguća |
| observer digne iznimku u `notify` | `logging.warning`, petlja se nastavlja | jedan pretplatnik ne ruši ostale |
| observer nije `IObserver` | `TypeError` iz `subscribe` | registar pretplatnika ostaje tipiziran |
| nepoznata akcija u `request` | `RuntimeError` | vidi §11 t.3 |

## 7. Rezultat

Vanjski sustav je dostupan kroz jedno imenovano sučelje koje dijele svi driveri u workflowu — isti
engine, isti proxy, isti kredencijal. Stanje pristupa je u svakom trenutku čitljivo (`state`,
`connected`), a prijelaz koji automat ne dopušta ne izvodi se (`BR-15-05`).

## 8. Kriteriji prihvaćanja

1. Automat se gradi u generičkoj klasi i pokriva engine i sesiju kao **odvojene** razine. ✅
2. `create_connection` i `disconnect` ne diraju automat; `connect` ga vodi. ✅
3. `ensure_created` / `ensure_closed` / `reset` su idempotentni. ✅
4. `connection_name` je obvezan i provjeren prije uporabe. ✅
5. `__del__` preživi neuspjelu konstrukciju bez maskiranja izvorne iznimke (`DR-WFL-014` t.4). ✅
6. `connection` property vraća sesiju **samo** u stanju `CONNECTED`. ✅
7. Kvar jednog observera ne ruši obavještavanje ostalih. ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Sve javne metode klase pripadaju konekciji. ❌ — vidi §11 t.1
10. Poruke kvara su na UK engleskom (`CLAUDE.md` §2.4/§3.2). ❌ — vidi §11 t.2

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `TRANSITIONS` | 8 stanja, 11 akcija; `DISCONNECT` vraća u `CREATED`, ne u `CLOSED` |
| 2 | pregled docstringova apstraktnih metoda | „do not manage FSM state here" ×2; upute za prijelaze u `connect` |
| 3 | pregled ranih `return` grana | tri metode, tri idempotentna izlaza |
| 4 | pregled `__init__` | `raise ConnectionException` prije `PresetDecorator` |
| 5 | pregled `__del__` | `object.__getattribute__(self, "_fsm")` u `try`, rani `return` |
| 6 | pregled propertyja | `return self._connection if self.connected else None` |
| 7 | pregled `notify` | `try/except Exception` → `logging.warning`, petlja se nastavlja |
| 8 | pregled modula | `__all__` = 4 imena; uvozi `logging`, `abc`, `enum`, `contextlib`, `typing`, `collections.abc` + `wattleflow.*` |
| 9 | `command grep -n '_connections\|notify_observers' concrete/connection.py` | `hot_swap` referira oba, a klasa nema nijedno |
| 10 | pregled poruka | „nije registrirana", „Hot-swap failao, stara konekcija ostaje aktivna" |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` i `processors` radna stabla 2026-08-27, CPython 3.11 (Linux/WSL2).
**Mjereno stablo:** `concrete/connection.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Connection` je rezervirani primitiv; pristup i operacija su odvojeni (`FRQ-DRV-15.6`) |
| `NFRQ-SEC-01` | jedna konekcija po pod-sustavu; kompromitacija jednog pristupa ne doseže druge |
| `NFRQ-SEC-02` | konfiguracijska površina je `ALLOWED` na tipu, ne slobodan `**kwargs` (`NFRQ-ORG-07`) |
| `NFRQ-SEC-03` | clean core tier; svaki third-party klijent (psycopg2, kafka-python…) živi u specijalizaciji u processorsu, lazy |
| `NFRQ-SEC-06` | konstruktorski zapis nosi ime, stanje i preset — nikad vrijednost tajne |
| `NFRQ-OBS-01` | cijeli životni ciklus je `DEBUG`; konekcija ne otvara vlastitu jedinicu posla |
| OSCAL | dekorateri (`ac-3`, `ia-5`, `sc-8`, `sc-13`) stoje na **specijalizacijama** u processorsu; ovaj sloj nema OSCAL referencu (`DR-WFL-015`) |

## 11. Otvoreno

1. **`hot_swap` ne pripada ovoj klasi i ne može se izvesti.** Metoda referira `self._connections`
   i `self.notify_observers(...)` — ni jedno ne postoji na `GenericConnection`. Oba postoje na
   `ConnectionManager` (`concrete/manager.py:56`, `__slots__ = ("_connections",)`). Poziv bi
   propao kroz `__getattr__` u preset, pa u `AttributeError`. Nijedan pozivatelj u tri stabla ne
   koristi metodu. Preseliti u `ConnectionManager` ili obrisati — kako god, danas je to javna
   metoda koja pada.
2. **Dvije poruke kvara su na hrvatskom**, i to s anglicizmom: „Connection '…' nije registrirana."
   i „Hot-swap failao, stara konekcija ostaje aktivna". `CLAUDE.md` §3.2 traži UK engleski u kodu
   bez iznimke. Obje su u `hot_swap` (t.1), pa se rješavaju zajedno s njom.
3. **`request` diže goli `RuntimeError`.** Svaki drugi kvar ovog modula izlazi kao
   `ConnectionException` ili `ConnectionManagerException` (`BR-15-09`); ovdje ne. Nepoznata akcija
   je kvar konfiguracije i zaslužuje isti razred.
4. **`version` se nikad ne postavlja.** `self._version: str = None` u konstruktoru i property koji
   ga vraća — nijedno mjesto u trima stablima mu ne dodjeljuje vrijednost. Ili ga specijalizacija
   treba puniti (pa to mora biti u ugovoru), ili se briše.
5. **`__getattr__` prolazi cijeli MRO pri svakom promašaju.** Za svaki neuspjeli pristup gradi se
   set svih `__slots__` kroz `type(self).__mro__`. Rezultat je konstantan po tipu i mogao bi se
   izračunati jednom; danas se računa po pozivu. Nije mjereno — uočeno.
6. **Zatečeni pravopis `formating`** spomenut je u komentaru konstruktora kao razlog zašto se
   stara grana uklonila, ali komentar ostaje bez zaključka o tome smije li se taj ključ još
   pojaviti u konfiguraciji. Isti razred problema kao `fmt` u blackboardu (`FRQ-BBD-15.1` §5 t.1)
   — zatečeni nazivi ključeva nemaju popis.
