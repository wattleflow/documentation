<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-CON — GenericConnection

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericConnection(ConnectionObserverInterface, Generic[Connection], ABC)` — pristup vanjskom sustavu; uz njega `ConnectionObserverInterface`, `ConnectionState`, `ConnectionAction`, `TRANSITIONS`, `ManagerException` |
| **Sestrinski** | [`FRQ-DRV`](FRQ-DRV-driver.md) (radi kroz konekciju) · [`FRQ-MGR`](FRQ-MGR-managers.md) (`ConnectionManager`) |
| **Izvedba** | [`connection.py`](../../../workflow/src/wattleflow/concrete/connection.py) (433 linije) |
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

Konekcija je **jedina granica prema vanjskom sustavu** (`BR-DRV-01`). Sve iznad nje — driver,
spremište, pipeline — poznaje samo njezino sučelje, nikad protokol.

Klasa razdvaja dvije razine trajanja koje se u praksi stalno miješaju:

| razina | što je | tko je vodi |
|---|---|---|
| **engine / pool** | dugotrajan resurs; gradi se jednom | `create_connection()` ↔ `disconnect()` |
| **sesija** | kratkotrajan zahvat unutar engine-a | `connect()` kao context manager |

Ta podjela je razlog zašto automat ima osam stanja umjesto dva: `CREATED` znači „engine postoji,
sesije nema", `CONNECTED` znači „sesija je otvorena". Zatvaranje sesije vraća u `CREATED`, ne u
`CLOSED` — engine preživljava.

**Automat je ovdje u generičkoj klasi**, za razliku od blackboarda ([`FRQ-BBD`](FRQ-BBD-blackboard.md) odjeljak 13 t.1).
Specijalizacija ga ne gradi nego ga **primjenjuje** — i to je jedina stvar koju generički sloj od
nje traži da radi sama, jer prijelaze `CONNECT`/`CONNECT_OK`/`CONNECT_FAIL`/`DISCONNECT` može
znati samo tijelo `connect()`.

Tri apstraktne metode, s izričitom podjelom odgovornosti nad automatom:

| metoda | tko vodi automat |
|---|---|
| `create_connection()` | **generički sloj** (`ensure_created`) — specijalizacija ga ne dira |
| `disconnect()` | **generički sloj** (`ensure_closed`) — isto |
| `connect()` | **specijalizacija** — ona jedina zna kada sesija počinje i završava |

`created` je istinito točno u stanjima `CREATING`, `CREATED` i `CONNECTED`, u kojima `request(action=Connect)` ne radi ništa (u `CONNECTING`, `CLOSING` i `FAILED` diže iznimku, a u `NEW` i `CLOSED` gradi engine); pozivatelj ga koristi da preskoči zahtjev koji bi samo ostavio audit zapis. `connected` znači da je sesija otvorena i vrijedi samo unutar `connect()`.

`version` je verzija udaljenog sustava kako je on sam prijavljuje; `None` znači nepoznato ili nije primjenjivo. Generička klasa je ne puni, a specijalizacija je smije postaviti samo iz odgovora udaljenog sustava.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericConnection

top to bottom direction

interface IObservable
interface IObserver
class PresetDecorator
class Wattleflow
class StateMachine
enum ConnectionState {
  NEW
  CREATING
  CREATED
  CONNECTING
  CONNECTED
  CLOSING
  CLOSED
  FAILED
}
enum ConnectionAction {
  CREATE
  CREATE_OK
  CREATE_FAIL
  CONNECT
  CONNECT_OK
  CONNECT_FAIL
  DISCONNECT
  CLOSE
  CLOSE_OK
  CLOSE_FAIL
  RESET
}
abstract class ConnectionObserverInterface {
  -_observers : dict[str, IObserver]
  +subscribe(observer)
  +subscribe_observer(observer)
  +transfer_observers(target)
  +notify(owner, **kwargs)
}
abstract class "GenericConnection<Connection>" as GC {
  +connection_name : str
  +connected : bool
  +connection : Connection | None
  +state : ConnectionState
  +version : str
  +can(action) : bool
  +ensure_created()
  +ensure_closed()
  +reset()
  +operation(action, **kwargs) : bool
  +request(**kwargs) : Any
  +context()
  {abstract} +create_connection()
  {abstract} +connect()
  {abstract} +disconnect()
}
Wattleflow <|-down- ConnectionObserverInterface
IObservable <|.down. ConnectionObserverInterface
ConnectionObserverInterface <|-down- GC
ConnectionObserverInterface o-- "0..*" IObserver
GC *-down- StateMachine : _fsm
GC *-down- PresetDecorator : _preset
GC -right-> ConnectionState : state
GC .right.> ConnectionAction
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericConnection

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "GenericConnection", "wattleflow.concrete.connection: access to external system with state machine")

System_Ext(wf, "WorkflowFactory", "Builds connection")
System_Ext(dr, "GenericDriver", "Uses session")
System_Ext(cm, "ConnectionManager / LazyDriverProxy", "Manager and proxy")
System_Ext(ob, "GenericDriver (IObserver)", "Subscriber")
System_Ext(vs, "External system", "Database, service")

Rel_D(wf, sys, "Builds")
Rel_D(dr, sys, "Requests session")
Rel_R(cm, sys, "Connect, disconnect, replace")
Rel_L(ob, sys, "Subscribes to changes")
Rel_D(sys, vs, "Opens engine and session")

Lay_R(wf, dr)
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi konekciju iz konfiguracije | `connection_name`, `lazy_loading`, preset ključevi |
| **A2** | `ConnectionManager` / `LazyDriverProxy` — traže povezivanje, odspajanje i zamjenu | ime konekcije |
| **A3** | `GenericDriver` / strategija — traži sesiju | `connect()` / `context()` |
| **A4** | `GenericConnection` — predmet ovog zahtjeva | engine i sesija |
| **A5** | `IObserver` — pretplatnik na promjene | `update(owner, **kwargs)` |

Vanjski sustav (baza, broker, poslužitelj) **nije akter** — ne pokreće ništa; sudionik je toka i
izvor kvara.

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira konekciju → `__init__` |
| **EV02** | gradnja engine-a → `CREATE` → `CREATE_OK` / `CREATE_FAIL` |
| **EV03** | A3 otvara sesiju → `CONNECT` → `CONNECT_OK` / `CONNECT_FAIL` |
| **EV04** | sesija se zatvara → `DISCONNECT` |
| **EV05** | A2 traži operaciju → `request(action=Operation.Connect/Disconnect)` (proxy: povezivanje; `ConnectionManager.__del__`: odspajanje) |
| **EV06** | rušenje engine-a → `CLOSE` → `CLOSE_OK` / `CLOSE_FAIL` |
| **EV07** | oporavak iz `FAILED`/`CLOSED` → `RESET` |
| **EV08** | pretplatnik se prijavljuje → `subscribe(observer)` |
| **EV09** | kraj životnog ciklusa → `__del__` |
| **EV10** | A2 zamjenjuje konekciju (`ConnectionManager.hot_swap`) → `transfer_observers(target)`, `notify(Event.Swap, …)`, zatim `operation(Disconnect)` na staroj |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericConnection

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
    actor "WorkflowFactory" as A1
    actor "ConnectionManager \n LazyDriverProxy" as A2
    actor "GenericDriver / StrategyWrite" as A3
    actor "GenericDriver (as IObserver)" as A5
  
    usecase "Instantiate connection" as EV01
    usecase "Build engine" as EV02
    usecase "Open session" as EV03
    usecase "Close session" as EV04
    usecase "Connect or disconnect on request" as EV05
    usecase "Tear down engine" as EV06
    usecase "Recover (RESET)" as EV07
    usecase "Subscribe observer" as EV08
    usecase "End lifecycle" as EV09
    usecase "Replace connection (hot swap)" as EV10
    A1 --> EV01
    A3 --> EV02
    A3 --> EV03
    A3 --> EV04
    A2 --> EV05
    A2 --> EV06
    A2 --> EV07
    A5 --> EV08
    A2 --> EV09
    A2 --> EV10
}
@enduml
```

## 07. Constraints and Preconditions

1. `connection_name` je zadan i nije prazan — inače `ConnectionException` **prije** nego objekt
   postane upotrebljiv (`BR-WFL-02`).
2. Specijalizacija deklarira dopuštene konfiguracijske ključeve kroz `ALLOWED` class-atribut;
   `PresetDecorator` ih razrješava iz **tipa**, ne iz instance (`NFRQ-ORG-07`).
3. Specijalizacija je implementirala sve tri apstraktne metode.
4. Tajne dolaze kao reference razrješive kroz konfiguracijski lanac, nikad kao literal
   (`BR-PTN-04`).

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — `connection_name` se izdvaja i provjerava; prazna vrijednost je kvar.
2. `PresetDecorator` preuzima konfiguraciju; automat se gradi u stanju `NEW`.
3. **EV02** — ako `lazy_loading` nije zatražen, `ensure_created()` se poziva odmah: `CREATE` →
   `create_connection()` → `CREATE_OK`. Kvar daje `CREATE_FAIL` i **ponovno diže** izvornu
   iznimku, s `DEBUG` tragom.
4. Konstruktor prijavljuje `Constructor/Completed` s imenom, stanjem i presetom — bez ijedne
   vrijednosti tajne (`BR-PTN-04`).
5. **EV03** — `connect()` (ili `context()`, koji ga omata) otvara sesiju. Prijelaze vodi tijelo
   specijalizacije: `CONNECT` prije otvaranja, `CONNECT_OK` na uspjeh, `CONNECT_FAIL` na kvar,
   `DISCONNECT` u `finally`.
6. Konekcija se može koristiti i kao `with` objekt: `__enter__` pamti context manager u
   `_context`, `__exit__` ga zatvara i **uvijek** ga poništava u `finally`.
7. **EV05** — ulaz za `ConnectionManager` je `operation(action, **kwargs) -> bool`: `Operation.Connect` i
   `Operation.Disconnect` obavlja kroz `request(action=…)` i vraća `True`. `request` prevodi
   `Operation.Connect` u `ensure_created()`, a `Operation.Disconnect` u `ensure_closed()`; kvar
   iz `ensure_created()` ili `ensure_closed()` izlazi iz `request` neizmijenjen, a nepoznata ili izostavljena
   akcija diže `ConnectionException`.
8. **EV09** — `__del__` prvo provjerava postoji li `_fsm`; polusagrađena instanca nema ni automat
   ni logger, pa bi prijava ondje zakopala pravu iznimku. Inače
   `ensure_closed()`.

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericConnection

participant "WorkflowFactory" as F
participant "GenericConnection" as C
participant "GenericDriver" as D
participant "ConnectionManager" as M
participant "GenericConnection (replacement)" as N

participant "External system" as V
participant "GenericDriver (as IObserver)" as O

  F -> C : __init__(**kwargs) with connection_name
  activate C #gold
  C -> C : state machine NEW, ensure_created()
  C -> V : create_connection() (CREATE, CREATE_OK)
  activate V #gold
  deactivate V
  deactivate C
  D -> C : connect() or context()
  activate D #gold
  activate C #gold
  C -> V : open session (CONNECT, CONNECT_OK)
  activate V #gold
  deactivate V
  C --> D : session
  deactivate C
  D -> C : exit from connect() or context() (DISCONNECT in finally)
  activate C #gold
  deactivate C
  deactivate D
  O -> C : subscribe(observer)
  activate C #gold
  deactivate C
  C -> C : notify(owner, **kwargs) (called by specialisation)
  C -> O : update(owner, **kwargs)
  activate O #gold
  deactivate O
  M -> C : operation(Operation.Connect) (request, ensure_created())
  activate C #gold
  C --> M : True
  deactivate C
  M -> N : operation(Operation.Connect)
  activate N #gold
  N --> M : True
  deactivate N
  M -> C : transfer_observers(target)
  activate C #gold
  C -> N : subscribe(observer)
  activate N #gold
  deactivate N
  deactivate C
  M -> N : notify(Event.Swap, name, connection)
  activate N #gold
  N -> O : update(Event.Swap, name, connection)
  activate O #gold
  deactivate O
  deactivate N
  M -> C : operation(Operation.Disconnect)
  activate C #gold
  deactivate C
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `connection_name` prazan ili izostavljen | `ConnectionException` iz konstruktora | konekcija bez identiteta ne nastaje |
| `create_connection()` padne | `CREATE_FAIL` → stanje `FAILED`, iznimka se diže dalje | kvar je vidljiv i stanju i pozivatelju |
| `ensure_created` iz stanja iz kojeg `CREATE` nije dopušten | `ManagerException` s imenom stanja | automat se ne zaobilazi |
| `ensure_created` kad je engine već tu | tihi `return` | idempotentno |
| `ensure_closed` iz `CLOSED`/`NEW`, ili kad `CLOSE` nije dopušten | tihi `return` | idempotentno |
| `disconnect()` padne | `CLOSE_FAIL` → `FAILED`, iznimka se diže | rušenje koje nije uspjelo ne prijavljuje se kao uspjeh |
| stanje `FAILED` | `_ensure_created()` **odmah odustaje** | ne ponavlja se gradnja koja je već pala |
| oporavak | `reset()` vraća `FAILED`/`CLOSED` u `NEW` — tiho ako nije dopušten | ponovna gradnja je moguća |
| observer digne iznimku u `notify` | `logging.warning`, petlja se nastavlja | jedan pretplatnik ne ruši ostale |
| observer nije `IObserver` | `TypeError` iz `subscribe` | registar pretplatnika ostaje tipiziran |
| nepoznata ili izostavljena akcija u `request` | `ConnectionException` | stanje automata ostaje nepromijenjeno |
| akcija koju `operation` ne podržava (`Start`, `Stop`) | `warning`, vraća `False` | isto kao `ProcessorManager.operation` |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericConnection

start
:constructor: connection_name, preset, state machine NEW;
if (connection_name empty?) then (yes)
:ConnectionException;
stop
else (no)
endif
if (lazy_loading?) then (yes)
:engine is built later;
else (no)
if (state CREATED, CONNECTED or CREATING?) then (yes)
  :silent return;
elseif (CREATE allowed?) then (no)
  :ManagerException;
  stop
else (yes)
  :CREATE; create_connection();
  if (fault?) then (yes)
    :CREATE_FAIL -> FAILED; exception is raised;
    stop
  else (no)
    :CREATE_OK;
  endif
endif
endif
:connect(): CONNECT and open session;
if (fault on open?) then (yes)
:CONNECT_FAIL;
else (no)
:CONNECT_OK;
endif
:DISCONNECT in finally;
stop
@enduml
```

## 10. State Machine

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption State Diagram
title GenericConnection

  [*] -down-> NEW
  NEW -down-> CREATING : CREATE
  CLOSED --> CREATING : CREATE
  CREATING -down-> CREATED : CREATE_OK
  CREATING -[#$WF_ERROR,thickness=2]-> FAILED : CREATE_FAIL
  CREATED -down-> CONNECTING : CONNECT
  CONNECTING -down-> CONNECTED : CONNECT_OK
  CONNECTING -up[#$WF_ERROR,thickness=2]-> CREATED : CONNECT_FAIL
  CONNECTED -up-> CREATED : DISCONNECT
  CREATING -left-> CLOSING : CLOSE
  CREATED -left-> CLOSING : CLOSE
  CONNECTING -left-> CLOSING : CLOSE
  CONNECTED -left-> CLOSING : CLOSE
  FAILED -left-> CLOSING : CLOSE
  CLOSING -down-> CLOSED : CLOSE_OK
  CLOSING -right[#$WF_ERROR,thickness=2]-> FAILED : CLOSE_FAIL
  CLOSED -up-> NEW : RESET
  FAILED -up-> NEW : RESET
@enduml
```

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Connection` je rezervirani primitiv; pristup i operacija su odvojeni ([`FRQ-DRV`](FRQ-DRV-driver.md)) |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | jedna konekcija po pod-sustavu; kompromitacija jednog pristupa ne doseže druge |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | konfiguracijska površina je `ALLOWED` na tipu, ne slobodan `**kwargs` (`NFRQ-ORG-07`) |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; svaki third-party klijent (psycopg2, kafka-python…) živi u specijalizaciji u `blackwattle`, lazy |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | konstruktorski zapis nosi ime, stanje i preset — nikad vrijednost tajne |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | cijeli životni ciklus je `DEBUG`; konekcija ne otvara vlastitu jedinicu posla |
| OSCAL | dekorateri (`ac-3`, `ia-5`, `sc-8`, `sc-13`) stoje na **specijalizacijama** u `blackwattle`; ovaj sloj nema OSCAL referencu |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Vanjski sustav je dostupan kroz jedno imenovano sučelje koje dijele svi driveri u workflowu — isti
engine, isti proxy, isti kredencijal. Stanje pristupa je u svakom trenutku čitljivo (`state`,
`connected`), a prijelaz koji automat ne dopušta ne izvodi se (`BR-PTN-03`).

## 13. Acceptance Criteria

1. Automat se gradi u generičkoj klasi i pokriva engine i sesiju kao **odvojene** razine. ✅
2. `create_connection` i `disconnect` ne diraju automat; `connect` ga vodi. ✅
3. `ensure_created` / `ensure_closed` / `reset` su idempotentni. ✅
4. `connection_name` je obvezan i provjeren prije uporabe. ✅
5. `__del__` preživi neuspjelu konstrukciju bez maskiranja izvorne iznimke. ✅
6. `connection` property vraća sesiju **samo** u stanju `CONNECTED`. ✅
7. Kvar jednog observera ne ruši obavještavanje ostalih. ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Sve javne metode klase pripadaju konekciji; zamjena registrirane konekcije je `ConnectionManager.hot_swap` ([`FRQ-MGR`](FRQ-MGR-managers.md)). ✅
10. Poruke kvara su na UK engleskom (`CLAUDE.md` §2.4/§3.2). ✅
11. `version` je verzija udaljenog sustava ili `None`; generička klasa je ne puni i svojstvo je samo za čitanje. ✅
12. `__getattr__` računa set `__slots__` jednom po tipu, a razrješavanje slotova i preseta ostaje isto. ✅
13. `lazy_loading` je deklariran u `GenericConnection.ALLOWED` i nikad se ne prijavljuje kao nepoznat ključ; ostali nedeklarirani ključevi i dalje se prijavljuju. ✅
14. `created` je istinito točno kad je `request(action=Connect)` nulta operacija (`CREATING`, `CREATED`, `CONNECTED`). ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | pregled `TRANSITIONS` | 8 stanja, 11 akcija; `DISCONNECT` vraća u `CREATED`, ne u `CLOSED` |
| 2 | pregled docstringova apstraktnih metoda | „do not manage FSM state here" ×2; upute za prijelaze u `connect` |
| 3 | pregled ranih `return` grana | tri metode, tri idempotentna izlaza |
| 4 | pregled `__init__` | `raise ConnectionException` prije `PresetDecorator` |
| 5 | pregled `__del__` | `object.__getattribute__(self, "_fsm")` u `try`, rani `return` |
| 6 | pregled propertyja | `return self._connection if self.connected else None` |
| 7 | pregled `notify` | `try/except Exception` → `logging.warning`, petlja se nastavlja |
| 8 | pregled modula | `__all__` = `ConnectionAction`, `ConnectionState`, `GenericConnection`, `ConnectionObserverInterface` (`TRANSITIONS` nije u njemu); uvozi `logging`, `abc`, `enum`, `contextlib`, `typing`, `collections.abc` + `wattleflow.*` |
| 9 | pregled javnih metoda `GenericConnection` | nema `hot_swap`; `transfer_observers` je na `ConnectionObserverInterface` |
| 10 | pregled poruka kvara u `connection.py` i `manager.py` | sve na UK engleskom |
| 11 | `workflow/tests/test_connection_request.py` (`VersionContractTest`, 4 testa) | prije ispravka pada anotacija `str \| None`; nakon njega prolaze svi (75/75 u `workflow/tests`, `ruff check` čist) |
| 12 | `workflow/tests/test_connection_getattr.py` (8 testova); mjerenje `bench_getattr.py` | prije ispravka padaju 3 testa predmemorije; nakon njega 83/83 u `workflow/tests`, `ruff check` čist; mutacija (`getattr` umjesto `__dict__`) ruši test zasebnog seta po tipu. Cijena čitanja kroz preset: 1,6–2,9 µs → ~0,7 µs, neovisno o dubini MRO-a |
| 13 | `workflow/tests/test_connection_preset.py` (7 testova) | prije ispravka pada 5 testova (deklaracija, lažno upozorenje, naslijeđeni ALLOWED, preset); nakon njega 90/90 u `workflow/tests`, `ruff check` čist; `PresetGate.resolve` nad `SqliteConnection` iz `blackwattle` i dalje sadrži `lazy_loading` |
| 14 | `workflow/tests/test_connection_request.py` (`CreatedPropertyTest`, 3 testa) | tri testa: istinito u tri stanja, lažno u ostalih pet, i usporedba sa stvarnim učinkom `request(Connect)` u svih osam stanja; prije ispravka nema svojstva, nakon njega 156/156 |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — odjeljak 13;
platforma — `workflow` i `blackwattle` radna stabla, CPython (Linux/WSL2).
**Mjereno stablo:** `concrete/connection.py`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-CON-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

-  [`connection.py`](../../../workflow/src/wattleflow/concrete/connection.py)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | DEF-CON-05, -06 i -07 premješteni u dokumentaciju `blackwattle` (sudar `_version`: `FRQ-CON-16.1` §11 t.4; odstupanja specijalizacija: analiza `2026-10-04-connection-specialisation-deviations`). Odjeljak 15 prazan. |
| v0.0.5 | 2026-10-04 | Dodano svojstvo `GenericConnection.created` (uz DEF-DRV-06 u `FRQ-DRV`): istinito točno gdje je zahtjev za spajanje nulta operacija; kriterij 14 i 3 testa. |
| v0.0.5 | 2026-10-03 | DEF-CON-04 zatvoren u `v0.0.1.23`: `GenericConnection.ALLOWED = ["lazy_loading"]` (preset spaja ALLOWED kroz MRO); kriterij 13 i 7 testova `test_connection_preset.py`. Specijalizacije u `blackwattle` još navode `lazy_loading` u vlastitom ALLOWED (DEF-CON-07). |
| v0.0.5 | 2026-10-03 | DEF-CON-03 zatvoren u `v0.0.1.23`: set `__slots__` računa se jednom po tipu (`_slot_names` u rječniku klase); kriterij 12 i 8 testova `test_connection_getattr.py`. |
| v0.0.5 | 2026-10-03 | DEF-CON-03: izmjerena cijena `__getattr__` i zapisan prijedlog (set slotova po tipu); kod nije mijenjan. |
| v0.0.5 | 2026-10-03 | DEF-CON-02 zatvoren u `v0.0.1.23`: ugovor `version` (verzija udaljenog sustava ili `None`) zapisan u 01 i kriteriju 11, anotacija `str \| None`, 4 testa (`VersionContractTest`); odstupanja specijalizacija izdvojena u DEF-CON-06. |
| v0.0.5 | 2026-10-03 | Dodan DEF-CON-05 (sudar imena `_version`) uz analizu DEF-CON-02. |
| v0.0.5 | 2026-10-03 | DEF-CON-01 zatvoren u `v0.0.1.23`: `request` za nepoznatu ili izostavljenu akciju diže `ConnectionException` (BR-PTN-05); test `workflow/tests/test_connection_request.py` (6 testova, prije ispravka 3 padaju), 71/71 testova i `ruff check` prolaze. Dodan DEF-CON-04 (`lazy_loading` i `ALLOWED`). |
| v0.0.5 | 2026-10-03 | Ispravljena netočna tvrdnja u koraku 7 normalnog toka: `request` ne guta iznimke (provjereno prema `connection.py`). |
| v0.0.5 | 2026-10-03 | Otvorene stavke u 15 preimenovane u defekte `DEF-CON-<nn>`; reference na njih preusmjerene. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); uklonjen zalutali tekst „DOBAR PRIMJER" iz konteksta, dijagram stanja izdvojen u 10, alternativni tokovi i dijagram toka u 09, unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Dijagrami uskladeni s kodom: klasa (`state`, `operation`, `transfer_observers`), slučajevi uporabe i slijed (EV10 zamjena konekcije, EV05 `operation`), legenda u kontekstnom dijagramu. Poveznice na izvedbu ispravljene (putanja do `workflow/`). |
| v0.0.5 | 2026-10-03 | `hot_swap` preseljen iz `GenericConnection` u `ConnectionManager` (FRQ-MGR); uklonjen iz dijagrama, kriteriji 9 i 10 zadovoljeni. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: pretplatnik `IObserver` zamijenjen klasom `GenericDriver`, `StrategyWrite` umjesto „Strategy", poziv konstruktora s `**kwargs`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | Dodan dijagram stanja za `GenericConnection` (tablica prijelaza `TRANSITIONS`). |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
