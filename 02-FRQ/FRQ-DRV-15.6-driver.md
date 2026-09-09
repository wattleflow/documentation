# FRQ-DRV-15.6 — Generički driver i odgođeni proxy

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `DRV`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericDriver(Wattleflow, IDriver, IObserver, ABC)` i `LazyDriverProxy`; uz njih `DriverMetadata`, `DriverState`, `DriverAction`, `TRANSITIONS` |
| **Sestrinski** | [`FRQ-CON-15.5`](FRQ-CON-15.5-connection.md) (pristup kroz koji driver radi) · [`FRQ-REP-15.7`](FRQ-REP-15.7-repository.md) (`RepositoryWithDriver`) · [`FRQ-DRV-13`](FRQ-DRV-13-llm-model.md) (specijalizacija u processorsu) |
| **Izvedba** | `workflow/src/wattleflow/concrete/driver.py` (363 linije) |

## 1. Predmet

Driver je **operacija nad vanjskim sustavom**: konekcija daje pristup, driver ga koristi. Podjela
je stroga jer se zamjenjuju neovisno — isti Postgres pristup opslužuje driver koji piše retke i
driver koji čita metapodatke.

Modul nosi dvije klase koje rješavaju dva različita problema:

| klasa | problem |
|---|---|
| `GenericDriver` | životni ciklus resursa: kad se učitava, kad se pušta, što kad padne |
| `LazyDriverProxy` | **trošak** tog ciklusa: ne graditi driver ni konekciju dok ih nitko ne treba |

Automat drivera nije automat konekcije. Konekcija razlikuje *engine* i *sesiju*; driver razlikuje
*učitan* i *degradiran*: `DEGRADED` je stanje u koje vodi neuspjeh učitavanja **i** neuspjeh
puštanja, i iz kojeg vodi ponovni pokušaj (`LOAD`) — dakle kvar ne završava životni ciklus
(`BR-15-06`).

`DriverMetadata` je dataclass s četiri polja (`name`, `version`, `protocol`, `capabilities`) —
samoopis kojim driver kaže što uopće zna raditi.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi driver ili proxy iz konfiguracije | ime konekcije, preset ključevi |
| **A2** | `DriverManager` — vlasnik registriranih drivera | ime drivera |
| **A3** | Strategija ili spremište — traži operaciju | `read` / `write` |
| **A4** | `GenericDriver` — predmet ovog zahtjeva | resurs i stanje |
| **A5** | `ConnectionManager` — daje pristup proxyju | imenovana konekcija |
| **A6** | `IObservable` — obavještava driver | `update(event, **kwargs)` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira driver → `__init__` |
| **EV02** | prvi zahtjev za operacijom → `ensure_live()` → `LOAD` → `LOAD_OK` / `LOAD_FAIL` |
| **EV03** | A3 traži `read` / `write` |
| **EV04** | privremeno mirovanje → `pause()` → `PAUSE` |
| **EV05** | puštanje resursa → `ensure_unloaded()` → `UNLOAD` → `UNLOAD_OK` / `UNLOAD_FAIL` |
| **EV06** | oporavak → `reset()` → `RESET` |
| **EV07** | A6 obavještava → `update(event, **kwargs)` |
| **EV08** | prvi pristup proxyju → `_ensure_ready()` |
| **EV09** | kraj životnog ciklusa → `__del__` |

## 4. Preduvjeti

1. Konekcija koju driver koristi je registrirana pod imenom koje proxy zna (`_conn_name`).
2. Specijalizacija je implementirala `load`, `close`, `read`, `write` — **po disciplini, ne po
   ugovoru** (vidi §11 t.1).
3. Preset je izgrađen; dopušteni ključevi dolaze iz `ALLOWED` na tipu (`NFRQ-ORG-07`).

## 5. Normalan tok

### `GenericDriver`

1. **EV01** — automat se gradi u `PENDING`; preset preuzima konfiguraciju.
2. **EV02** — `ensure_live()`: ako je već `LIVE` ili `LOADING`, tihi izlaz. Inače `LOAD` →
   `self.load()` → `LOAD_OK`.
3. **EV03** — operacija se izvodi nad učitanim resursom.
4. **EV05** — `ensure_unloaded()`: `UNLOAD` → `self.close()` → `UNLOAD_OK`.
5. **EV07** — `update` prijavljuje događaj **kao jedno imenovano polje**: `event=getattr(event,
   "name", event)`, a ostatak ide pod `kwargs`. Time ključ pozivatelja nikad ne može postati
   kontrolni argument (`DR-WFL-018` t.3).
6. **EV09** — `__del__` provjerava postoji li `_fsm` pa odustaje ako ne — polusagrađena instanca
   nema ni automat ni logger, a interpreter može već rušiti logging aparat.

### `LazyDriverProxy`

1. **EV01** — proxy pamti **tvornicu** (`Callable[[], GenericDriver]`), menadžer konekcija i ime
   konekcije. Ništa se ne gradi. Razina zapisa se postavlja na `WARNING` — proxy je po prirodi
   brbljav.
2. **EV08** — `_ensure_ready()`: dohvati konekciju, poveži je ako nije povezana, izgradi driver
   ako ga nema, pa `ensure_live()`.
3. Svaki javni član (`read`, `write`, `metadata`, `load`) delegira kroz `_ensure_ready()`.
4. `__getattr__` delegira **samo javna imena**. Privatna i dunder imena dižu `AttributeError`
   odmah — inače bi `copy`, `pickle`, `hasattr` ili debugger otvorili konekciju i poništili
   cijelu svrhu proxyja.
5. `update` **ne budi** driver: prosljeđuje samo ako driver već postoji.
6. `close()` pušta resurs ali čuva driver; `release()` pušta i **odbacuje** driver, pa ga sljedeći
   poziv gradi ispočetka.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `ensure_live` iz stanja iz kojeg `LOAD` nije dopušten | `DriverException` s imenom stanja | automat se ne zaobilazi |
| `load()` padne | `LOAD_FAIL` **ako ga automat još prima** → `DEGRADED`, iznimka se diže | ponovni pokušaj je moguć |
| podklasa je već sama primijenila `LOAD_FAIL` | drugi `apply` se **preskače** | `ValueError` iz automata ne bi zamijenio pravi uzrok kvara |
| `close()` padne | `UNLOAD_FAIL` → `DEGRADED`, iznimka se diže | isto |
| `ensure_unloaded` kad `UNLOAD` nije dopušten | tihi `return` | idempotentno |
| `pause` / `reset` kad prijelaz nije dopušten | tihi `return` | idempotentno |
| proxy: konekcija nije povezana | `conn.request(action=Operation.Connect)` | povezivanje je dio prvog pristupa |
| proxy: `close()` / `reset()` / `release()` dok je još lijen | tihi `return` | puštanje neizgrađenog resursa nije kvar |
| proxy: `__del__` nakon neuspjele konstrukcije | `release()` u `try/except Exception: pass` | destruktor ne diže |
| proxy: pristup `_`-imenu | `AttributeError` bez izgradnje | introspekcija ne budi konekciju |

## 7. Rezultat

Operacija nad vanjskim sustavom izvedena je nad resursom čije je stanje u svakom trenutku
poznato, a neuspjeh je ostavio driver u stanju iz kojeg vodi ponovni pokušaj. Uz proxy, ni driver
ni konekcija ne nastaju dok ih neka operacija stvarno ne zatraži — workflow s deset konfiguriranih
drivera otvara samo one koje koristi.

## 8. Kriteriji prihvaćanja

1. Automat se gradi u generičkoj klasi; `DEGRADED` je dostupan iz oba smjera kvara. ✅
2. Knjigovodstvo stanja **nikad** ne zamjenjuje pravi uzrok kvara. ✅
3. `ensure_*`, `pause`, `reset` su idempotentni. ✅
4. `__del__` obiju klasa preživi neuspjelu konstrukciju. ✅
5. Proxy ne gradi ništa do prvog **javnog** pristupa; introspekcija ga ne budi. ✅
6. `update` prosljeđuje payload kao imenovano polje (`DR-WFL-018` t.3). ✅
7. `metadata()` proxyja govori istinu — vraća metapodatke omotanog drivera. ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Ugovor drivera (`load`/`close`/`read`/`write`) je provediv. ❌ — vidi §11 t.1
10. Komentari su na UK engleskom (`CLAUDE.md` §2.4). ❌ — vidi §11 t.2

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `TRANSITIONS` | `LOAD_FAIL` i `UNLOAD_FAIL` oba vode u `DEGRADED`; `LOAD` je dopušten iz `DEGRADED` |
| 2 | pregled obiju `except` grana | `if self._fsm.can(...)` prije `apply`, uz komentar koji navodi razlog |
| 3 | pregled ranih `return` grana | pet metoda, pet idempotentnih izlaza |
| 4 | pregled oba `__del__` | `object.__getattribute__(self, "_fsm")` odnosno `try/except Exception: pass` |
| 5 | pregled `LazyDriverProxy.__getattr__` | `if name.startswith("_"): raise AttributeError(name)` |
| 6 | pregled `update` | `event=getattr(event, "name", event)`, ostatak pod `kwargs=` |
| 7 | pregled `metadata` | instance-metoda uz komentar zašto ne `classmethod` |
| 8 | pregled modula | `__all__` = 5 imena; uvozi `logging`, `abc`, `dataclasses`, `enum`, `typing`, `collections.abc` + `wattleflow.*` |
| 9 | `command grep -c '@abstractmethod' concrete/driver.py` | **nula** — hookovi su namjerno nedeklarirani |
| 10 | pregled komentara | `# stvarni driver` uz `self._driver` |

**Dopunska mjera (k.9).** Svih **20** drivera u `wattleflow-processors` implementira sve četiri
metode i `metadata()` — disciplina danas drži. To je stanje, ne jamstvo.

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` i `processors` radna stabla 2026-08-27, CPython 3.11 (Linux/WSL2).
**Mjereno stablo:** `concrete/driver.py` + `processors/src/wattleflow/drivers/`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Driver` je rezervirani primitiv; operacija je odvojena od pristupa (`FRQ-CON-15.5`) |
| `NFRQ-SEC-01` | driver ne poznaje ni pipeline ni platno; kompromitacija drivera doseže jedan vanjski sustav |
| `NFRQ-SEC-02` | proxy odbija delegirati `_`-imena — introspekcijska površina ne otvara resurs |
| `NFRQ-SEC-03` | clean core tier; svaki third-party klijent živi u specijalizaciji u processorsu, lazy (`CLAUDE.md` §7.4) |
| `NFRQ-OBS-01` | cijeli životni ciklus je `DEBUG`; proxy je dodatno stišan na `WARNING` |
| `NFRQ-OBS-02` | `update` nosi `event` i `kwargs` kao imenovana polja, nikad splat (`NFRQ-SEC-06` k.1) |
| OSCAL | dekorater `@oscal_driver` stoji na **specijalizacijama** u processorsu; ovaj sloj nema OSCAL referencu (`DR-WFL-015`) |

## 11. Otvoreno

1. **Ugovor drivera je disciplina, ne ugovor.** `load`, `close`, `read`, `write` nisu
   `@abstractmethod` — komentar to naziva „subclass hooks, intentionally not abstract". Posljedica:
   driver koji zaboravi `load` instancira se, a poziv propada kroz `__getattr__` u preset, pa
   izlazi kao `AttributeError` umjesto kao `DriverException`. Danas svih 20 drivera hookove ima,
   ali to je isti razred tvrdnje koji `DR-WFL-014` odbija za OSCAL vrata: *„vrata su svojstvo
   hijerarhije, ne discipline"*. Odlučiti — ili apstraktne metode, ili zapisati zašto ovdje
   disciplina zadovoljava.
2. **Hrvatski komentar u kodu:** `self._driver: GenericDriver | None = None  # stvarni driver`.
   `CLAUDE.md` §2.4 i §3.2 traže UK engleski u `src/**/*.py` bez iznimke.
3. **`__all__` je na vrhu modula, prije definicija.** Radi, ali odudara od svih ostalih modula
   sloja, gdje stoji na dnu. Uz to `TRANSITIONS` nije u njemu — isti slučaj kao kod blackboarda
   (`FRQ-BBD-15.1` §11 t.3), samo ovdje nitko izvana tablicu ne uvozi.
4. **Proxy ne prosljeđuje automat.** `LazyDriverProxy` nema `state`, `can()` ni `pause()`, pa
   pozivatelj koji drži proxy ne može pitati u kojem je stanju driver iza njega — a `driver`
   property vraća `None` dok je lijen, pa ni zaobilazno. Ili proxy dobiva iste upitne članove, ili
   se zapisuje da stanje nije dio proxy ugovora.
5. **`DriverMetadata.capabilities` je slobodan popis stringova.** Komentar navodi
   `["read", "write", "stream"]`, ali ništa ne provjerava sadržaj ni odnos prema stvarno
   implementiranim metodama. Samoopis koji se ne provjerava je tvrdnja, ne svjedočanstvo (D-05).
6. **`_ensure_ready` svaki put dohvaća konekciju iz menadžera i provjerava `connected`.** Za
   petlju po stavkama to je dohvat po pozivu; nije mjereno je li mjerljivo — uočeno.
