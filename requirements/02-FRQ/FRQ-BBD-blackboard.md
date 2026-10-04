<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-BBD — GenericBlackboard

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01-GENERIC-LAYER`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — `BR-BBD-01`, `BR-PTN-03`, `BR-PRC-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericBlackboard(Wattleflow, IBlackboard, Generic[Item], ABC)` — dijeljeni radni prostor između pipelinea i spremišta; uz njega `BlackboardState`, `BlackboardAction` i tablica `TRANSITIONS` |
| **Sestrinski** | [`FRQ-PIP`](FRQ-PIP-pipeline.md) (piše na platno) · [`FRQ-REP`](FRQ-REP-repository.md) (prima flush) · [`FRQ-STR`](FRQ-STR-strategy.md) (`StrategyCreate`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/blackboard.py` |
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

`GenericBlackboard` je **platno za pohranu strukturiranih i nestrukturiranih dokumenata tijekom jednog procesa (ciklusa)**.

- Koriste ga `Processor` i `Pipeline` (cjevovod): na platno stavljaju dokumente nad kojima se obavljaju operacije.
- Dizajniran je kao temeljni element sloja trajne pohrane (`data persistence`).
- Neophodan je unutar svakog `workflow` (procesa) i služi dijeljenju radne memorije između pipelinea.
- Prima podatke u serijama i osigurava konzistentnost svih tokova podataka.
- Bez njega bi svaka transformacija sama odlučivala kada i kamo piše, što komplicira dizajn i održavanje koda te izmjene na spremištima svakog pipelinea.

**Javno sučelje** (ugovor `IBlackboard`; apstraktno u sedam točaka):

| član | uloga |
|---|---|
| `canvas` | nepromjenjiv pogled `Mapping[str, Item]` na platno |
| `state` | trenutno stanje automata (`BlackboardState`) |
| `repositories` | kopija popisa spremišta |
| `count`, `clean`, `create`, `delete`, `flush`, `read`, `write` | apstraktne točke; specijalizacija određuje politiku |

## 02. Class Diagrams

| član | uloga |
|---|---|
| `_canvas: Item` | radno platno; generički parametar — specijalizacija bira oblik |
| `_fsm: StateMachine` | automat nad `TRANSITIONS`, gradi se u stanju `IDLE` |
| `_repositories: list[Repository]` | odredišta na koja `flush` odašilje; izvana se vidi kao kopija (`repositories`) |
| `_strategy_create: StrategyCreate` | kako nastaje nova stavka na platnu |
| `_preset: PresetDecorator` | razrješavanje konfiguracijskih imena kroz `__getattr__` |

- Platno se izvana vidi **za čitanje** (`canvas` vraća `MappingProxyType` nad preslikavanjem identifikator → stavka, `Mapping[str, Item]`). 
- Klasa je **apstraktna u sedam točaka** (`count`, `clean`, `create`, `delete`, `flush`, `read`, `write`)
- te propisuje *ugovor i životni ciklus*, a ne *politiku*;
- Klasa drži **automat** nad tablicom prijelaza i odbija prijelaz koji tablica ne dopušta (`_transition`); 


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericBlackboard

top to bottom direction

interface IBlackboard <<core>>
interface IRepository <<core>>

class Wattleflow
class PresetDecorator
class StrategyCreate
class StateMachine

enum BlackboardState {
  IDLE
  READY
  DIRTY
  FAILED
  CLEARED
}
enum BlackboardAction {
  REGISTER
  WRITE
  FLUSH
  LOAD
  FAIL
  CLEAN
}

abstract class "GenericBlackboard<Item>" as GB {
  -_canvas : Item
  -_fsm : StateMachine
  +canvas : Mapping[str, Item] <<read-only>>
  +state : BlackboardState
  +repositories : list[IRepository]
  {abstract} +count : int
  {abstract} +clean()
  {abstract} +create(caller, **kwargs) : Item
  {abstract} +delete(identifier, **kwargs) : None
  {abstract} +flush(caller, **kwargs) : bool
  {abstract} +read(identifier, **kwargs) : Item
  {abstract} +write(pipeline, facade, **kwargs) : Any
  #_transition(action)
  #_try_transition(action) : bool
}

Wattleflow <|-- GB
IBlackboard <|.down. GB
IRepository <-down-o GB : "0..*"
GB o-- StrategyCreate
GB *-down- PresetDecorator
GB *-down- StateMachine : _fsm
BlackboardState .. BlackboardAction : TRANSITIONS
BlackboardAction -- StateMachine
@enduml
```
Tablica `TRANSITIONS` (stanje, akcija → stanje) definirana je u modulu; automat `_fsm` drži klasa.

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericBlackboard

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "GenericBlackboard", "wattleflow.concrete.blackboard: canvas between pipelines and repositories")

System_Ext(wf, "WorkflowFactory", "Builds from configuration")
System_Ext(pr, "GenericProcessor", "Cycle owner")
System_Ext(pi, "GenericPipeline", "Produces results")
System_Ext(rp, "GenericRepository", "Destination")

Rel_D(wf, sys, "Builds")
Rel_U(pr, sys, "Creates document")
Rel_U(pr, sys, "Flush, clean")
Rel_R(pi, sys, "Writes result")
Rel_R(sys, rp, "Writes on flush")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| `A1` | `WorkflowFactory` — gradi blackboard iz konfiguracije | `strategy_create`, `canvas`, preset ključevi |
| `A2` | `GenericProcessor` — vlasnik ciklusa; traži `flush` i `clean` | granica ciklusa |
| `A3` | `GenericPipeline` — piše rezultat transformacije | `facade` i pozivatelj |
| `A4` | `GenericRepository` — odredište | prima `write` pri flushu |

Spremište je akter jer ga `flush` poziva; **platno nije akter** — ono je stanje, ne sudionik.

## 05. Events

| oznaka | trigger | akcija automata |
|---|---|---|
| `EV01` | Factory instancira blackboard | — (stanje `IDLE`) |
| `EV02` | Repository se pridružuje | `REGISTER` |
| `EV03` | Procesor kreira document | `CREATE` |
| `EV04` | Pipeline piše rezultat | `WRITE` |
| `EV05` | Blackboard pise u repository | `WRITE` |
| `EV06` | Procesor zatvara ciklus | `FLUSH` |
| `EV07` | Procesor kraj životnog ciklusa | `CLEAN` |

Prijelazi `LOAD` (oporavak iz snimke) i `FAIL` (kvar) postoje u tablici `TRANSITIONS` (odjeljak 10), ali ih pokreću specijalizacije; vode se u njihovoj dokumentaciji.

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericBlackboard

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {

  actor WorkflowFactory as Factory
  actor GenericProcessor as Procesor
  actor GenericPipeline as Pipeline
  actor GenericRepository as Repository

  usecase "Instantiate" as EV01
  usecase "Attach repository" as EV02
  usecase "Create document" as EV03
  usecase "Write to canvas" as EV04
  usecase "Write to repository" as EV05
  usecase "Close cycle (flush)" as EV06
  usecase "End lifecycle (clean)" as EV07

  Factory --> EV01
  Factory --> EV02
  Procesor --> EV03
  Pipeline --> EV04
  Repository <-- EV05
  Procesor --> EV06
  Procesor --> EV07
}
@enduml
```

## 07. Constraints and Preconditions

1. `strategy_create` je instanca `StrategyCreate` — provjereno `assert`-om **prije** `super().__init__`.
2. `canvas` je predan konstruktoru; generički sloj mu ne propisuje tip.
3. Barem jedan `Repository` je registrirano prije prvog `flush`-a, inače flush nema odredište.
4. `defer_flush` je postavljen (zadano `True`, kad ga pozivatelj nije zadao).

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — konstruktor (alias `fmt` za `formatting` razrješava `Audit`, [`FRQ-PTN`](FRQ-PTN-root-base.md)) postavlja `defer_flush=True` ako
   ga pozivatelj nije zadao i prosljeđuje **cijeli** `**kwargs` naviše (`HLRQ-01-GENERIC-LAYER` §4 t.2).
2. `PresetDecorator` preuzima ostatak ključeva; od te točke `__getattr__` razrješava konfiguracijska
   imena kroz preset, a ne kroz `__dict__`.
3. Konstruktor prijavljuje `Constructor/Started` i `Completed` na `DEBUG` ([`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md)).
4. **EV02** — `register` primjenjuje `REGISTER` (nedopušten prijelaz diže `BlackboardException` i popis spremišta ostaje nepromijenjen), zatim spremište ulazi u `_repositories`.
5. **EV03** — `write(pipeline, facade)` stavlja rezultat na platno; platno je *prljavo* dok se ne isprazni.
6. **EV04** — `flush(caller, **kwargs)` odašilje platno svakom spremištu i vraća platno u čisto stanje.
   Potpis vraća `bool`: `True` samo ako je na platnu bio dokument i svako spremište vratilo `True`.
   Ishod čita procesor.
7. **EV05** — `__del__` odustaje ako `_preset` nije postavljen; inače `Delete/Started` → `clean()` →
   oslobađanje strategije i preseta → `Delete/Completed`.

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericBlackboard

participant "GenericProcessor" as GenericProcessor
participant "GenericPipeline" as GenericPipeline
participant "GenericBlackboard" as GenericBlackboard
participant "GenericRepository" as Repository

  GenericProcessor -> GenericProcessor : start()
  activate GenericProcessor #e8b924

  loop for each pipeline (1..n)
    GenericProcessor -> GenericPipeline : process(processor, facade)
    activate GenericPipeline #e8b924
    GenericPipeline -> GenericPipeline : transform(processor, facade, **kwargs)
    GenericPipeline -> GenericBlackboard : write(pipeline, facade)
    activate GenericBlackboard #e8b924
    GenericBlackboard -> GenericBlackboard : facade to canvas, DIRTY
    note right of GenericBlackboard : no repository write,\nfacade waits on canvas
    GenericBlackboard --> GenericPipeline
    deactivate GenericBlackboard
    GenericPipeline --> GenericProcessor
    deactivate GenericPipeline
  end

  GenericProcessor -> GenericBlackboard : flush(caller=self, **write_context)
  activate GenericBlackboard #e8b924
  loop each repository
    GenericBlackboard -> Repository : write(caller=self, facade=facade)
    activate Repository #e8b924
    Repository --> GenericBlackboard : bool
    deactivate Repository
  end
  GenericBlackboard -> GenericBlackboard : READY, canvas empty
  GenericBlackboard --> GenericProcessor : bool
  deactivate GenericBlackboard

  GenericProcessor -> GenericBlackboard : clean()
  activate GenericBlackboard #e8b924
  GenericBlackboard -> GenericBlackboard : CLEARED
  deactivate GenericBlackboard
  deactivate GenericProcessor
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `strategy_create` nije `StrategyCreate` | `AssertionError` prije konstrukcije | objekt ne nastaje s neispravnom strategijom |
| `__init__` padne prije `_preset` | `__del__` odustaje, `__getattr__` diže `AttributeError` | prava iznimka nije maskirana |
| `write` ili `flush` bez registriranog spremišta | propisuje specijalizacija; generička klasa ga ne propisuje | — |
| iznimka unutar `__del__` | `error(...)` u `try`, a i taj poziv je u `try/except Exception: pass` | destruktor nikad ne diže (`HLRQ-01-GENERIC-LAYER` §4 t.4) |
| akcija koju tablica ne dopušta (`REGISTER`, `WRITE`, `FLUSH`, `LOAD`) | `_transition` diže `BlackboardException`, stanje i platno ostaju | nedopuštena promjena ne prolazi |
| `FAIL` ili `CLEAN` koje tablica ne dopušta | `_try_transition` vraća `False`, ne diže | destruktor i ponovljen `clean` su sigurni |

### Dijagram aktivnosti: Processor, Pipeline, Blackboard

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
title Activity Diagram: GenericProcessor, GenericBlackboard

|GenericProcessor|
start
:start();
while (more pipelines?) is (yes)
  :process(processor, facade);
  |GenericPipeline|
  :transform(processor, facade, **kwargs);
  :write(pipeline, facade);
  |GenericBlackboard|
  :facade to canvas;
  if (state allows WRITE?) then (yes)
    :state DIRTY;
  endif
  if (not defer_flush?) then (yes)
    |GenericRepository|
    :write(caller=self, facade=facade);
  endif
  |GenericProcessor|
  :next pipeline;
endwhile (no)
:flush(caller=self, **write_context);
|GenericBlackboard|
if (facade on canvas and FLUSH allowed?) then (yes)
  |GenericRepository|
  :write(caller=self, facade=facade) to each;
  |GenericBlackboard|
  if (no write raised an exception?) then (yes)
    :canvas empty; state READY;
  else (no)
    :<b><color:red>state FAILED: BlackboardException</color></b>;
    kill
  endif
endif
|GenericProcessor|
:clean(); state CLEARED;
stop
@enduml
```

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericBlackboard

start
:write(pipeline, facade);
if (state allows WRITE?) then (no)
:BlackboardException; state and canvas unchanged;
stop
else (yes)
:facade to canvas; state DIRTY;
endif
if (defer_flush?) then (yes)
:canvas waits for flush;
else (no)
:straight to repositories;
endif
:flush(caller=self, **write_context);
if (facade on canvas and FLUSH allowed?) then (no)
:nothing to write; state unchanged; returns False;
else (yes)
if (no repository.write raised an exception?) then (no)
  :FAIL; state FAILED; BlackboardException;
  stop
else (yes)
  :state READY; canvas empty; returns confirmed;
endif
endif
:clean(): state CLEARED;
stop
@enduml
```

## 10. State Machine

Tablica `TRANSITIONS` u modulu. Akcija bez prijelaza iz trenutnog stanja nije dopuštena
(`StateMachine.apply` diže `ValueError`, `can` vraća `False`). Klasa prijelaze primjenjuje kroz `_transition(action)` (nedopušten diže `BlackboardException` i ne mijenja stanje; koriste ga `REGISTER`, `WRITE`, `FLUSH`, `LOAD`) i `_try_transition(action)` (vraća `bool`, ne diže; koriste ga `FAIL` i `CLEAN`).

| Akcija | `REGISTER` | `WRITE` | `FLUSH` | `LOAD` | `FAIL` | `CLEAN` |
|---|---|---|---|---|---|---|
| `IDLE` | `READY` | — | — | `READY` | `FAILED` | `CLEARED` |
| `READY` | `READY` | `DIRTY` | `READY` | — | `FAILED` | `CLEARED` |
| `DIRTY` | — | `DIRTY` | `READY` | — | `FAILED` | `CLEARED` |
| `FAILED` | — | — | — | `READY` | — | `CLEARED` |
| `CLEARED` | — | — | — | — | — | — |


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption State Diagram
title GenericBlackboard

[*] -down-> IDLE
IDLE -right-> READY : REGISTER
IDLE -right-> READY : LOAD
READY -right-> READY : REGISTER
READY -down-> DIRTY : WRITE
READY -right-> READY : FLUSH
DIRTY -right-> DIRTY : WRITE
DIRTY -up-> READY : FLUSH
FAILED -up-> READY : LOAD
IDLE -down[#$WF_ERROR,thickness=2]-> FAILED : FAIL
READY -left[#$WF_ERROR,thickness=2]-> FAILED : FAIL
DIRTY -left[#$WF_ERROR,thickness=2]-> FAILED : FAIL
IDLE -down-> CLEARED : CLEAN
READY -down-> CLEARED : CLEAN
DIRTY -down-> CLEARED : CLEAN
FAILED -down-> CLEARED : CLEAN
CLEARED -down-> [*]
note right of FAILED : recovery only by LOAD,\nending only by CLEAN
@enduml
```

## 11. Non-Functional Requirements

Registar i obveze skupine: [`HLRQ-01-GENERIC-LAYER` §6](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md#6-nefunkcionalni-zahtjevi).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Blackboard` je rezervirani primitiv; platno nije „mali repozitorij" i ne preuzima njegovu ulogu |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | platno ne poznaje spremište osim kroz `Repository`; kvar drivera ne doseže platno |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `__all__` izlaže četiri imena, među njima `TRANSITIONS` koji specijalizacije uvoze |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | modul je u clean core tieru; nijedan third-party uvoz |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | konstrukcija i destrukcija su `DEBUG`; jedinica posla je flush, i nju prijavljuje specijalizacija |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | životni ciklus i preset se ne prepisuju po specijalizacijama — žive ovdje |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: `GenericBlackboard` deklarira `__slots__` s pet članova; bez `__slots__ = ()` u `IBlackboard` deklaracija nema učinka ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) §5) |

## 12. Results

Rezultati transformacije skupljaju se na jednom mjestu i odlaze u spremišta u serijama, na granici
ciklusa koju određuje procesor. Pipeline ne zna koliko spremišta postoji ni kojeg su tipa; spremište
ne zna koji ga je pipeline proizveo. Zamjena spremišta je izmjena konfiguracije (`BR-WFL-01`).

## 13. Acceptance Criteria

1. `Wattleflow` prethodi `Generic[Item]` u popisu baza (MRO za `GenericBlackboard + IOriginator`). ✅
2. `canvas` vraća **nepromjenjiv** pogled `Mapping[str, Item]`. ✅
3. `strategy_create` se provjerava prije `super().__init__`. ✅
4. `__del__` preživi neuspjelu konstrukciju bez maskiranja izvorne iznimke. ✅
5. Modul deklarira `__all__`; `__slots__` pokriva svih pet članova. ✅
6. Import closure modula je `stdlib ∪ wattleflow` ([`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md)). ✅
7. Tablica prijelaza pokriva `FAIL` iz svakog neterminalnog stanja i `CLEAN` iz svakog stanja. ✅
8. Generička klasa drži automat i odbija prijelaz koji tablica ne dopušta. ✅
9. Ugovor `flush` definira što vraćeni `bool` znači: `True` samo ako je na platnu bio barem jedan dokument i svako spremište je potvrdilo svaki. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–6 | pregled `blackboard.py` | zadovoljeno; `__all__ = ["BlackboardAction", "BlackboardState", "GenericBlackboard", "TRANSITIONS"]`, `__slots__` = 5 imena |
| 7 | prebrojavanje `TRANSITIONS` | `FAIL` iz `IDLE`/`READY`/`DIRTY`; `CLEAN` iz sva četiri |
| 2, 5, 8 | `workflow/tests/test_blackboard.py` (`AutomatonTest`, `CanvasContractTest`, `ModuleContractTest`) | prolaze; mutacije (bez `REGISTER` u `register`, `_transition` koji ne diže) ruše testove |
| 9 | docstring apstraktne metode `flush`, `FlushContractTest` | definicija ishoda zapisana; `-> bool` |

**Trojka (D-10):** alat — čitanje koda i `unittest` (`workflow/tests`, 65 testova); kriterij — odjeljak 13;
platforma — radno stablo `workflow`, CPython 3.12 (Linux/WSL2).

## 15. Open issues

Nema otvorenih stavki u opsegu `workflow`.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/blackboard.py`
- Testovi: `workflow/tests/test_blackboard.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); tekst normalnog toka, dijagram slijeda i dijagrami toka razvrstani u 08 i 09; dodan popis javnog sučelja u 01. |
| v0.0.5 | 2026-10-03 | Otvorene stavke riješene: automat i `_transition`/`_try_transition` u generičkoj klasi, `READ` uklonjen, `canvas` je `Mapping[str, Item]`, ishod `flush` definiran, zastarjeli `defer_flush` uklonjen iz procesora. Poveznice na izvedbu ispravljene (putanja do `workflow/`). |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: `IRepository`, `GenericPipeline`/`GenericProcessor`/`GenericRepository` i specijalizacije umjesto uloga, StrategyCreate kao agregacija, pozivi `start`/`process`/`transform`/`write(caller=self, ...)`/`flush(caller=self, **write_context)`, uvjet flusha i neuspjeha. |
| v0.0.5 | 2026-10-02 | Prva verzija dokumenta sa uml diagramima. |
| v0.0.5 | 2026-10-03 | Izmjene na diagramima i nazivima |
