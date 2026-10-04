<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-DRV — Generički driver i odgođeni proxy
| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `DRV` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericDriver(Wattleflow, IDriver, IObserver, ABC)` i `LazyDriverProxy`; uz njih `DriverMetadata`, `DriverState`, `DriverAction`, `TRANSITIONS` |
| **Sestrinski** | [`FRQ-CON`](FRQ-CON-connection.md) (pristup kroz koji driver radi) · [`FRQ-REP`](FRQ-REP-repository.md) (`RepositoryWithDriver`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/driver.py` (366 linija) |
| **Dijagrami** | inline (02, 03, 06, 08, 09, 10) — pogledi izvedeni iz koda, ne izvor istine (D-13) |

# Index
- [01. Interface](#01-interface)
- [02. Class Diagrams](#02-class-diagrams)
- [03. Context Diagram](#03-context-diagram)
- [04. User Diagram](#04-user-diagram)
- [05. Events](#05-events)
- [06. Use Case Diagrams](#06-use-case-diagrams)
- [07. Constraints and Preconditions](#07-constraints-and-preconditions)
- [08. Sequence Diagrams](#08-sequence-diagrams)
- [09. Flow Chart Diagrams](#09-flow-chart-diagrams)
- [10. State Machine](#10-state-machine)
- [11. Non-Functional Requirements](#11-non-functional-requirements)
- [12. Results](#12-results)
- [13. Acceptance Criteria](#13-acceptance-criteria)
- [14. Verification](#14-verification)
- [15. Open issues](#15-open-issues)
- [16. References](#16-references)
- [17. Change history](#17-change-history)

## 01. Interface

<p style="gld">
Driver je **operacija nad vanjskim sustavom**: konekcija daje pristup, driver ga koristi. Podjela
je stroga jer se zamjenjuju neovisno — isti Postgres pristup opslužuje driver koji piše retke i
driver koji čita metapodatke.

Modul nosi dvije klase koje rješavaju dva različita problema:
</p>

| klasa | problem |
|---|---|
| `GenericDriver` | životni ciklus resursa: kad se učitava, kad se pušta, što kad padne |
| `LazyDriverProxy` | **trošak** tog ciklusa: ne graditi driver ni konekciju dok ih nitko ne treba |

Automat drivera nije automat konekcije. Konekcija razlikuje *engine* i *sesiju*; driver razlikuje
*učitan* i *degradiran*: `DEGRADED` je stanje u koje vodi neuspjeh učitavanja **i** neuspjeh
puštanja, i iz kojeg vodi ponovni pokušaj (`LOAD`) — dakle kvar ne završava životni ciklus
(`BR-PRC-01`).

`DriverMetadata` je dataclass s četiri polja (`name`, `version`, `protocol`, `capabilities`) —
samoopis kojim driver kaže što uopće zna raditi.

## 02. Class Diagrams



```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericDriver

top to bottom direction

interface IDriver
interface IObserver
class Wattleflow
class StateMachine
class ConnectionManager
enum DriverState {
  PENDING
  LOADING
  LIVE
  PAUSED
  DEGRADED
  UNLOADING
  UNLOADED
}
enum DriverAction {
  LOAD
  LOAD_OK
  LOAD_FAIL
  UNLOAD
  UNLOAD_OK
  UNLOAD_FAIL
  PAUSE
  RESET
}
class DriverMetadata <<dataclass>> {
  +name : str
  +version : str
  +protocol : str
  +capabilities : list[str]
}
abstract class GenericDriver {
  +can(action : DriverAction) : bool
  +ensure_live()
  +ensure_unloaded()
  +operation(action : Operation, **kwargs) : bool
  +pause()
  +reset()
  +update(event : Any, **kwargs)
}
abstract class LazyDriverProxy {
  +driver : GenericDriver | None
  +load()
  +metadata()
  +read(uri : str, **kwargs)
  +write(uri : str, **kwargs)
  +update(event : Any, **kwargs)
  +close()
  +reset()
  +unload()
  +release()
}
Wattleflow <|-down- GenericDriver
Wattleflow <|-down- LazyDriverProxy
IDriver <|.down. GenericDriver
IDriver <|.down. LazyDriverProxy
IObserver <|.down. GenericDriver
IObserver <|.down. LazyDriverProxy
GenericDriver *-down- StateMachine : _fsm
GenericDriver -right-> DriverState : state
GenericDriver .right.> DriverAction
LazyDriverProxy o-- "0..1" GenericDriver : _driver
LazyDriverProxy -right-> "1" ConnectionManager : _conn_mgr
@enduml
```



## 03. Context Diagram



```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericDriver

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "GenericDriver, LazyDriverProxy", "Performs operations on an external system")

System_Ext(wf, "WorkflowFactory", "Builds a driver or proxy")
System_Ext(dm, "DriverManager", "Holds registered drivers")
System_Ext(st, "Strategy or RepositoryWithDriver", "Requests an operation")
System_Ext(cm, "ConnectionManager", "Provides a connection to the proxy")
System_Ext(vs, "External system", "File, database, service")

Rel_L(wf, sys, "Builds driver")
Rel_R(dm, sys, "Connects, disconnects, unloads")
Rel_U(st, sys, "Calls read and write")
Rel_D(cm, sys, "Provides named connection")
Rel_L(sys, vs, "Reads and writes resource")
@enduml
```



## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **WorkflowFactory** | `WorkflowFactory` — gradi driver ili proxy iz konfiguracije | ime konekcije, preset ključevi |
| **DriverManager** | `DriverManager` — vlasnik registriranih drivera | ime drivera |
| **A3** | Strategija ili spremište — traži operaciju | `read` / `write` |
| **A4** | `GenericDriver` — predmet ovog zahtjeva | resurs i stanje |
| **A5** | `ConnectionManager` — daje pristup proxyju | imenovana konekcija |
| **IObservable** | `IObservable` — obavještava driver | `update(event, **kwargs)` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | WorkflowFactory instancira driver → `__init__` |
| **EV02** | prvi zahtjev za operacijom → `ensure_live()` → `LOAD` → `LOAD_OK` / `LOAD_FAIL` |
| **EV03** | A3 traži `read` / `write` |
| **EV04** | privremeno mirovanje → `pause()` → `PAUSE` |
| **EV05** | puštanje resursa → `ensure_unloaded()` → `UNLOAD` → `UNLOAD_OK` / `UNLOAD_FAIL` |
| **EV06** | oporavak → `reset()` → `RESET` |
| **EV07** | IObservable obavještava → `update(event, **kwargs)` |
| **EV08** | prvi pristup proxyju → `_ensure_ready()` |
| **EV09** | kraj životnog ciklusa → `__del__` |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericDriver

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {

  actor IObservable as IObservable
  actor WorkflowFactory
  actor DriverManager
  actor ConnectionManager
  actor "Strategy or RepositoryWithDriver" as A1

  usecase "Instantiate driver" as EV01
  usecase "Load resource on first request" as EV02
  usecase "Read or write" as EV03
  usecase "Pause" as EV04
  usecase "Release resource" as EV05
  usecase "Recover `reset`" as EV06
  usecase "Receive notification" as EV07
  usecase "Prepare proxy on first access" as EV08
  usecase "End life cycle" as EV09
  IObservable --> EV07
  WorkflowFactory --> EV01
  DriverManager --> EV04
  DriverManager --> EV05
  DriverManager --> EV06
  ConnectionManager --> EV08
  DriverManager --> EV09
  A1 --> EV02
  A1 --> EV03
  A1 --> EV08
}
@enduml
```



## 07. Constraints and Preconditions

1. Konekcija koju driver koristi je registrirana pod imenom koje proxy zna (`_conn_name`).
2. Specijalizacija je implementirala `load`, `close`, `read`, `write` i `metadata()`: apstraktni su kroz `IDriver`, pa driver bez njih ne može nastati (`TypeError`).
3. Preset je izgrađen; dopušteni ključevi dolaze iz `ALLOWED` na tipu (`NFRQ-ORG-07`).

## 08. Sequence Diagrams

#### `GenericDriver`

1. **EV01** — automat se gradi u `PENDING`; preset preuzima konfiguraciju.
2. **EV02** — `ensure_live()` (iz `operation(Operation.Connect)`): ako je već `LIVE` ili `LOADING`, tihi izlaz. Inače `LOAD` →
   `self.load()` → `LOAD_OK`.
3. **EV03** — operacija se izvodi nad učitanim resursom.
4. **EV05** — `ensure_unloaded()` (iz `operation(Operation.Disconnect)`): `UNLOAD` → `self.close()` → `UNLOAD_OK`.
5. **EV07** — `update` prijavljuje događaj **kao jedno imenovano polje**: `event=getattr(event,
   "name", event)`, a ostatak ide pod `kwargs`. Time ključ pozivatelja nikad ne može postati
   kontrolni argument.
6. **EV09** — `__del__` provjerava postoji li `_fsm` pa odustaje ako ne — polusagrađena instanca
   nema ni automat ni logger, a interpreter može već rušiti logging aparat.

#### `LazyDriverProxy`

1. **EV01** — proxy pamti **tvornicu** (`Callable[[], GenericDriver]`), menadžer konekcija i ime
   konekcije. Ništa se ne gradi. Razina zapisa se postavlja na `WARNING` — proxy je po prirodi
   brbljav.
2. **EV08** — `_ensure_ready()`: dohvati konekciju iz menadžera (pri svakom pozivu), zatraži spajanje samo ako nije stvorena (`created`), izgradi driver
   ako ga nema, pa `ensure_live()`.
3. Javni članovi `read`, `write`, `metadata`, `load` delegiraju kroz `_ensure_ready()`; `close`, `reset`, `unload` (alias `close`) i `release` rade samo nad već izgrađenim driverom.
4. `__getattr__` delegira **samo javna imena**. Privatna i dunder imena dižu `AttributeError`
   odmah — inače bi `copy`, `pickle`, `hasattr` ili debugger otvorili konekciju i poništili
   cijelu svrhu proxyja.
5. `update` **ne budi** driver: prosljeđuje samo ako driver već postoji.
6. `close()` pušta resurs ali čuva driver; `release()` pušta i **odbacuje** driver, pa ga sljedeći
   poziv gradi ispočetka.


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericDriver

  participant "WorkflowFactory" as F
  participant "GenericDriver" as D
  participant "DriverManager" as M
  participant "Strategy | RepositoryWithDriver" as S
  participant "External system" as V
  F -> D : __init__ (PENDING)
  activate D #e8b924
  S -> D : read or write
  activate D #e8b924
  activate S  #e8b924
  D -> D : ensure_live()
  D -> D : load() (LOAD, LOAD_OK -> LIVE)
  D -> V : operate on resource
  activate V #e8b924
  V --> D : result
  deactivate V
  D --> S : result
  deactivate S
  M -> D : ensure_unloaded()
  D -> D : close() (UNLOAD, UNLOAD_OK)
  deactivate D
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `ensure_live` iz stanja iz kojeg `LOAD` nije dopušten | `DriverException` s imenom stanja | automat se ne zaobilazi |
| `load()` padne | `LOAD_FAIL` **ako ga automat još prima** → `DEGRADED`, iznimka se diže | ponovni pokušaj je moguć |
| podklasa je već sama primijenila `LOAD_FAIL` | drugi `apply` se **preskače** | `ValueError` iz automata ne bi zamijenio pravi uzrok kvara |
| `close()` padne | `UNLOAD_FAIL` → `DEGRADED`, iznimka se diže | isto |
| `ensure_unloaded` kad `UNLOAD` nije dopušten | tihi `return` | idempotentno |
| akcija koju `operation` ne podržava (`Start`, `Stop`) | `warning`, vraća `False` | isto kao `ProcessorManager.operation` |
| `pause` / `reset` kad prijelaz nije dopušten | tihi `return` | idempotentno |
| proxy: konekcija nije stvorena (`created` je `False`) | `conn.request(action=Operation.Connect)` | povezivanje je dio prvog pristupa; mirujuća konekcija se ne traži ponovno |
| proxy: `close()` / `reset()` / `release()` dok je još lijen | tihi `return` | puštanje neizgrađenog resursa nije kvar |
| proxy: `__del__` nakon neuspjele konstrukcije | `release()` u `try/except Exception: pass` | destruktor ne diže |
| proxy: pristup `_`-imenu | `AttributeError` bez izgradnje | introspekcija ne budi konekciju |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.



```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericDriver

start
:ensure_live();
if (state LIVE or LOADING?) then (yes)
:silent exit;
stop
else (no)
endif
if (LOAD allowed?) then (no)
:DriverException (state name);
stop
else (yes)
endif
:LOAD; load();
if (load() fails?) then (yes)
if (state machine still accepts LOAD_FAIL?) then (yes)
  :LOAD_FAIL -> DEGRADED;
else (no)
  :second apply is skipped;
endif
:exception is raised;
stop
else (no)
:LOAD_OK -> LIVE;
endif
:read / write on resource;
if (UNLOADED or UNLOAD not allowed?) then (yes)
:silent exit;
stop
else (no)
endif
:ensure_unloaded(): UNLOAD; close();
if (close() fails?) then (yes)
:UNLOAD_FAIL -> DEGRADED (if accepted); exception is raised;
else (no)
:UNLOAD_OK;
endif
stop
@enduml
```

## 10. State Machine

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption State Diagram
title GenericDriver

  [*] -down-> PENDING
  PENDING -down-> LOADING : LOAD
  UNLOADED -right-> LOADING : LOAD
  PAUSED -up-> LOADING : LOAD
  DEGRADED -up-> LOADING : LOAD
  LOADING -down-> LIVE : LOAD_OK
  LOADING -right[#$WF_ERROR,thickness=2]-> DEGRADED : LOAD_FAIL
  LIVE -down-> PAUSED : PAUSE
  LIVE -right-> UNLOADING : UNLOAD
  PAUSED -right-> UNLOADING : UNLOAD
  DEGRADED -right-> UNLOADING : UNLOAD
  UNLOADING -down-> UNLOADED : UNLOAD_OK
  UNLOADING -left[#$WF_ERROR,thickness=2]-> DEGRADED : UNLOAD_FAIL
  UNLOADED -up-> PENDING : RESET
@enduml
```
## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Driver` je rezervirani primitiv; operacija je odvojena od pristupa ([`FRQ-CON`](FRQ-CON-connection.md)) |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | driver ne poznaje ni pipeline ni platno; kompromitacija drivera doseže jedan vanjski sustav |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | proxy odbija delegirati `_`-imena — introspekcijska površina ne otvara resurs |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; svaki third-party klijent živi u specijalizaciji u `blackwattle`, lazy (`ARCHITECTURE.md` §7.4) |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | cijeli životni ciklus je `DEBUG`; proxy je dodatno stišan na `WARNING` |
| [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) | `update` nosi `event` i `kwargs` kao imenovana polja, nikad splat ([`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) k.1) |
| OSCAL | dekorater `@oscal_driver` stoji na **specijalizacijama** u `blackwattle`; ovaj sloj nema OSCAL referencu |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi §Otvoreno |

## 12. Results

Operacija nad vanjskim sustavom izvedena je nad resursom čije je stanje u svakom trenutku
poznato, a neuspjeh je ostavio driver u stanju iz kojeg vodi ponovni pokušaj. Uz proxy, ni driver
ni konekcija ne nastaju dok ih neka operacija stvarno ne zatraži — workflow s deset konfiguriranih
drivera otvara samo one koje koristi.

## 13. Acceptance Criteria

1. Automat se gradi u generičkoj klasi; `DEGRADED` je dostupan iz oba smjera kvara. ✅
2. Knjigovodstvo stanja **nikad** ne zamjenjuje pravi uzrok kvara. ✅
3. `ensure_*`, `pause`, `reset` su idempotentni. ✅
4. `__del__` obiju klasa preživi neuspjelu konstrukciju. ✅
5. Proxy ne gradi ništa do prvog **javnog** pristupa; introspekcija ga ne budi. ✅
6. `update` prosljeđuje payload kao imenovano polje. ✅
7. `metadata()` proxyja govori istinu — vraća metapodatke omotanog drivera. ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Ugovor drivera (`load`/`close`/`read`/`write`) je provediv: hookovi su apstraktni kroz `IDriver`, driver koji izostavi jedan ne može se instancirati. ✅
10. Komentari su na UK engleskom (`STANDARDS.md` §2.4). ✅
11. `__all__` je zadnja naredba modula `driver.py` (kao u ostalim modulima sloja `concrete`), iza definicija koje imenuje; `TRANSITIONS` ostaje izvan njega jer ga nitko ne uvozi (`STANDARDS` §2.7 t.1). ✅
12. Upit proxyja o stanju drivera (`state`, `can()`, `pause()`) ne spaja, ne gradi niti nastavlja driver: dok je proxy lijen `state` je `PENDING`, `can()` odgovara kao za `PENDING`, a `pause()` ne radi ništa; nakon gradnje sve se delegira. ✅
13. `DriverMetadata.capabilities` je lista bez duplikata iz kontroliranog vokabulara (`DriverCapability`, `NFRQ-ORG-12`); neispravan popis odbija se pri izgradnji (`TypeError` za tip, `ValueError` za nepoznat ili ponovljen zapis). Novi zapis traži izmjenu vokabulara (D-12). ✅
14. Proxy šalje zahtjev za spajanje samo kad on ima što raditi (`GenericConnection.created` je `False`), ne pri svakom pozivu; konekciju i dalje dohvaća iz menadžera pri svakom pozivu (prati `hot_swap`), a stanja u kojima zahtjev diže iznimku (`CONNECTING`, `CLOSING`, `FAILED`) ostaju vidljiva. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | pregled `TRANSITIONS` | `LOAD_FAIL` i `UNLOAD_FAIL` oba vode u `DEGRADED`; `LOAD` je dopušten iz `DEGRADED` |
| 2 | pregled obiju `except` grana | `if self._fsm.can(...)` prije `apply`, uz komentar koji navodi razlog |
| 3 | pregled ranih `return` grana | četiri metode (`ensure_live`, `ensure_unloaded`, `pause`, `reset`), idempotentni rani izlazi |
| 4 | pregled oba `__del__` | `object.__getattribute__(self, "_fsm")` odnosno `try/except Exception: pass` |
| 5 | pregled `LazyDriverProxy.__getattr__` | `if name.startswith("_"): raise AttributeError(name)` |
| 6 | pregled `update` | `event=getattr(event, "name", event)`, ostatak pod `kwargs=` |
| 7 | pregled `metadata` | instance-metoda uz komentar zašto ne `classmethod` |
| 8 | pregled modula | `__all__` = 5 imena; uvozi `logging`, `abc`, `dataclasses`, `enum`, `typing`, `collections.abc` + `wattleflow.*` |
| 9 | `workflow/tests/test_driver_contract.py` (5 testova); izvršavanje nad `GenericDriver` | `__abstractmethods__` = `close`, `load`, `metadata`, `read`, `write`; podrazred bez hookova diže `TypeError` s imenima nedostajućih. Raniji mjerni `grep '@abstractmethod' concrete/driver.py` = 0 bio je slijep za nasljeđenu apstraktnost iz `IDriver` (D-11). Testovi prolaze prije i poslije ispravka komentara (127/127), jer je netočan bio komentar i ovaj dokument, ne ponašanje |
| 10 | `workflow/tests/test_source_language.py` (2 testa); pregled `workflow/src` | jedini hrvatski komentar bio je `# stvarni driver` u `driver.py`; zamijenjen engleskim. Zaštitni test traži hrvatska slova u `workflow/src` i prolazi (129/129, `ruff` čist); mutacija s hrvatskim slovima ga ruši. **Granica (D-11):** test ne vidi hrvatsku riječ bez dijakritika (taj komentar ga ne bi pao), to hvata samo pregled |
| 11 | `workflow/tests/test_module_layout.py` (4 testa) | prije ispravka pada test položaja (`driver.py`, `serialisation.py` imali su `__all__` na vrhu); nakon premještanja 133/133 u `workflow/tests`, `import *` radi, `ruff check` čist nad izmijenjenim datotekama. Empirija: u `concrete/` 19 od 21 modula imalo je `__all__` na dnu; u širem `workflow/src` 30 na dnu, 18 na vrhu; u `blackwattle` 140 na vrhu, 45 na dnu (tamo pravilo ne vrijedi) |
| 12 | `workflow/tests/test_driver_proxy.py` (8 testova, zamjenski manager i driver) | prije ispravka pada 6 testova: `proxy.state` je gradio driver i spajao konekciju (`built=1`, `connect=1`) i uvijek vraćao `LIVE`, a na pauziranom driveru ga je nastavljao (`ensure_live`); nakon ispravka 141/141 u `workflow/tests`, `ruff` bez nalaza osim E501 (zanemaren); mutacija (`can` uvijek `True`) ruši test |
| 13 | `workflow/tests/test_driver_metadata.py` (8 testova); jednokratna provjera svih 22 `DriverMetadata` poziva iz `blackwattle/drivers` | prije ispravka pada 5 testova odbijanja (nepoznato, duplikat, tip popisa, tip zapisa, velika slova); nakon njega 149/149 u `workflow/tests`; svih 22 stvarnih deklaracija prihvaćeno, 0 odbijeno. Potrošača `capabilities` u kodu nema (`grep`), pa je ovo ograda, ne ponašanje. **Granica (D-11):** ne provjerava se je li deklarirana sposobnost doista implementirana (npr. `find`, `stream`, `push` nemaju istoimenu metodu) |
| 14 | `workflow/tests/test_driver_proxy.py` (`RequestPerCallTest`, `FailureStaysVisibleTest`, 4 testa); mjerenje `bench_proxy.py` | stari kod: 6 zahtjeva umjesto 1 (`6 != 1`); `proxy.read` 1738 ns (33× izravnog poziva od 53 ns), 1000/1000 zahtjeva i 2 audit zapisa po pozivu uz DEBUG; nakon ispravka 420 ns (8×), 0/1000 zahtjeva, 156/156 u `workflow/tests`; mutacija svojstva `created` ruši tri testa; pad konekcije (`FAILED`) i dalje diže `ManagerException` |

**Dopunska mjera (k.9).** Svih **22** drivera u `blackwattle/src/wattleflow/drivers/` (izravno ili nasljeđivanjem) ima sve četiri
metode i `metadata()`; to je sada i jamstvo, ne samo stanje (`IDriver`).

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — odjeljak 13;
platforma — `workflow` i `blackwattle` radna stabla, CPython (Linux/WSL2).
**Mjereno stablo:** `concrete/driver.py` + `blackwattle/src/wattleflow/drivers/`.

## 15. Open issues

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/driver.py`
- Testovi: `workflow/tests/test_driver_contract.py`, `workflow/tests/test_driver_metadata.py`, `workflow/tests/test_driver_proxy.py`, `workflow/tests/test_module_layout.py`, `workflow/tests/test_source_language.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | Ispravljen opis `_ensure_ready` u normalnom toku (korak 2) i u alternativnim tokovima: provjera je `created`, ne `connected`. |
| v0.0.5 | 2026-10-04 | DEF-DRV-06 zatvoren u `v0.0.1.23`: `_ensure_ready` pita `GenericConnection.created` umjesto `connected` (sesija je otvorena samo unutar `connect()`, pa je prijašnja provjera bila uvijek lažna za mirujuću konekciju); kriterij 14 i testovi. Odjeljak 15 prazan: svi defekti `FRQ-DRV zatvoreni. |
| v0.0.5 | 2026-10-04 | DEF-DRV-05 zatvoren u `v0.0.1.23`: kontrolirani vokabular `DriverCapability` (`wattleflow.enums.capability`, 15 zapisa iz svih 22 deklaracija) i provjera u `DriverMetadata.__post_init__`; kriterij 13 i 8 testova `test_driver_metadata.py`. Provjera da je sposobnost stvarno implementirana ostaje deklarirana slijepa pjega. |
| v0.0.5 | 2026-10-04 | DEF-DRV-04 zatvoren u `v0.0.1.23`: `LazyDriverProxy` ima lijeno-sigurne `state`, `can()` i `pause()` (nikad ne grade, ne spajaju niti nastavljaju driver); kriterij 12 i 8 testova `test_driver_proxy.py`. Tvrdnja dokumenta da proxy nema te članove bila je netočna: postojali su kroz `__getattr__`, ali su svaki upit pretvarali u učitavanje. |
| v0.0.5 | 2026-10-04 | DEF-DRV-03 zatvoren u `v0.0.1.23`: `__all__` premješten na dno u `driver.py` i `serialisation.py`, pravilo upisano u `STANDARDS` §2.7 t.5 (dokumentirana izmjena, D-03), kriterij 11 i 4 testa `test_module_layout.py`. `TRANSITIONS` ostaje izvan `__all__`. |
| v0.0.5 | 2026-10-04 | DEF-DRV-02 zatvoren u `v0.0.1.23`: komentar `# stvarni driver` zamijenjen engleskim, dodan zaštitni test `test_source_language.py` (uz deklariranu granicu). |
| v0.0.5 | 2026-10-04 | DEF-DRV-01 zatvoren u `v0.0.1.23`: premisa je bila netočna, hookovi su apstraktni kroz `IDriver`; ispravljen komentar u `driver.py` i tekst dokumenta, 5 testova `test_driver_contract.py`. |
| v0.0.5 | 2026-10-03 | Otvorene stavke u 15 preimenovane u defekte `DEF-DRV-<nn>`; reference na njih preusmjerene. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: „Strategy or repository” → „Strategy or RepositoryWithDriver”; oznaka veze DriverManager → driver prema kodu; DriverMetadata, GenericDriver i LazyDriverProxy potpisi i tipovi, `operation`, `driver`; ishod `UNLOAD_FAIL` (ako ga automat prima). |
| v0.0.5 | 2026-10-02 | Dodan dijagram stanja za `GenericDriver` (tablica prijelaza `TRANSITIONS`). |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
