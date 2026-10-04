<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-PRC — Generički procesor

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericProcessor(Wattleflow, IProcessor, IOriginator, ABC)` — vlasnik prolaza nad skupom stavki; uz njega `ProcessorState`, `ProcessorAction`, `TRANSITIONS` |
| **Sestrinski** | [`FRQ-PIP`](FRQ-PIP-pipeline.md) (poziva se po stavci) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) (flush na granici ciklusa) · [`FRQ-MEM`](FRQ-MEM-memento.md) (snimka stanja) |
| **Izvedba** | `workflow/src/wattleflow/concrete/processor.py` |
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

Procesor je **vlasnik prolaza**: on zna koliko stavki ima, dovodi ih jednu po jednu, provlači
svaku kroz sve pipelinee i odlučuje kada se platno prazni. On je i **jedina** klasa ovog sloja
koja je istodobno `IProcessor` i `IOriginator` — dakle jedina koja svoje stanje zna spremiti u
snimku i iz nje se nastaviti.

| član | uloga |
|---|---|
| `_generator` | izvor stavki; gradi ga apstraktni `create_generator()` |
| `_pipelines` | redoslijed transformacija kroz koje svaka stavka prolazi |
| `_blackboard` | platno na koje pipelinei pišu |
| `_cycle` | broj dovršenih stavki — uz stanje automata **jedina** vrijednost koja ulazi u snimku |
| `_fsm` | automat stanja; ovdje je u **generičkoj** klasi, ne u specijalizaciji |
| `_flush_per_cycle` | prazni li se platno nakon svake stavke ili jednom, kad su sve stavke gotove |
| `_current` | stavka koja je trenutno u obradi |
| `_flush_outcome` | ishod flusha za tekući dokument: `bool` ili `None` (nepoznato); svojstvo `flush_outcome` |
| `_memento_store`, `_memento_key`, `_checkpoint_every` | neobavezna kontrolna točka ([`FRQ-MEM`](FRQ-MEM-memento.md)): pohrana, ključ snimke i razmak u ciklusima |

Specijalizacija piše **jednu** metodu: `create_generator()`. Sve ostalo — petlja, automat, audit,
granice kvara, snimka — nasljeđuje.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericProcessor

top to bottom direction

interface IProcessor
interface IOriginator
class Wattleflow

abstract class GenericProcessor {
  # _cycle : int
  # _flush_per_cycle : bool
  # _memento_store : MementoStore | None
  # _memento_key : str
  # _checkpoint_every : int
  + blackboard : IBlackboard
  + cycle : int
  + flush_per_cycle : bool
  + flush_outcome : bool | None
  + write_context : dict[str, Any]
  + operation(action : Operation, **kwargs) : bool
  + start()
  + register_blackboard(blackboard : IBlackboard)
  + register_pipeline(pipeline : IPipeline)
  + save_state() : GenericMemento
  + restore_state(memento : GenericMemento)
  + {abstract} create_generator() : Generator[ITarget, None, None]
}

enum ProcessorState {
  IDLE
  STATE_LOADED
  RUNNING
  COMPLETED
  FAILED
}
enum ProcessorAction {
  LOAD
  START
  NEXT_ITEM
  CYCLE_COMPLETED
  RECORDS_PROCESSED
  FAIL
  STORE
}
class StateMachine
class GenericMemento
class MementoStore
interface IBlackboard
interface IPipeline

IProcessor <|.. GenericProcessor
IOriginator <|.. GenericProcessor
Wattleflow <|-- GenericProcessor
GenericProcessor "1" *-- "1" StateMachine : _fsm
GenericProcessor "1" o-- "1" IBlackboard
GenericProcessor "1" o-- "1..*" IPipeline
GenericProcessor "1" o-- "0..1" MementoStore
GenericProcessor ..> GenericMemento : creates
GenericProcessor ..> ProcessorState
GenericProcessor ..> ProcessorAction
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericProcessor

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "GenericProcessor", "Owns the pass over a set of items")
System_Ext(wf, "WorkflowFactory", "Builds the processor")
System_Ext(or, "ProcessorManager / Orchestrator", "Starts the pass")
System_Ext(pi, "GenericPipeline", "Processes an item")
System_Ext(bb, "GenericBlackboard", "Canvas")
System_Ext(me, "GenericMemento", "Snapshot")
Rel_L(wf, sys, "Builds the processor")
Rel_R(or, sys, "Requests start (Start)")
Rel_U(sys, pi, "Receives each item")
Rel_D(sys, bb, "Receives flush at the cycle boundary")
Rel_L(sys, me, "Carries the state snapshot")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi procesor iz konfiguracije | `blackboard`, `pipelines`, `flush_per_cycle` |
| **A2** | `GenericWorkflow` / `Orchestrator` — pokreće prolaz | `Operation.Start` |
| **A3** | `GenericProcessor` — predmet ovog zahtjeva | ciklus i stanje |
| **A4** | `GenericPipeline` — prima svaku stavku | `process(processor, facade)` |
| **A5** | `GenericBlackboard` — prima `flush` | granica ciklusa |
| **A6** | `GenericMemento` — nosi snimku | `cycle` + stanje automata |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira procesor → `__init__` |
| **EV02** | pipeline ili blackboard se pridružuje → `register_pipeline` / `register_blackboard` |
| **EV03** | A2 traži pokretanje → `operation(Operation.Start)` → `start()` |
| **EV04** | generator daje sljedeću stavku → `NEXT_ITEM` |
| **EV05** | stavka prošla sve pipelinee → `CYCLE_COMPLETED` |
| **EV06** | generator iscrpljen → `RECORDS_PROCESSED` |
| **EV07** | kvar u prolazu → `FAIL` |
| **EV08** | nastavak iz snimke → `restore_state(memento)` → `LOAD`; uz `memento_store` sam `start()` čita zadnju snimku (`_resume`), nakon svakog ciklusa je zapisuje (`_checkpoint`), a nakon uspjeha briše (`_release`) |
| **EV09** | kraj životnog ciklusa → `__del__` |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericProcessor

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "WorkflowFactory" as A1
  actor "ProcessorManager / Orchestrator" as A2
  actor "GenericPipeline" as A4
  actor "GenericBlackboard" as A5
  actor "GenericMemento" as A6
    usecase "Instantiate processor" as EV01
    usecase "Register pipeline or blackboard" as EV02
    usecase "Start pass" as EV03
    usecase "Process items" as EV04
    usecase "Report fault in pass" as EV05
    usecase "Resume from snapshot" as EV06
    usecase "End lifecycle" as EV07
  A1 --> EV01
  A1 --> EV02
  A2 --> EV03
  A4 --> EV04
  A5 --> EV04
  A2 --> EV05
  A6 --> EV06
  A1 --> EV07
}
@enduml
```

## 07. Constraints and Preconditions

1. Blackboard je registriran — inače `start()` odbija raditi (`BR-WFL-02`).
2. Najmanje jedan pipeline je registriran — isto.
3. Specijalizacija je implementirala `create_generator()`.
4. Za nastavak: snimka nosi stanje iz kojeg je `LOAD` dopušten (`IDLE` ili `FAILED`).
5. Nastavak pretpostavlja **isti skup stavki istim redoslijedom** (aspiracija, D-05): `restore_state` premotava generator pozivom `next()` točno `cycle` puta, a snimka nosi broj, ne identitet zadnje stavke. Za izvor bez zajamčenog redoslijeda nastavak bi obrađivao druge stavke. Premotavanje je linearno: izmjereno 90 ns po stavci za trivijalan generator (1 milijun stavki 90 ms); stvarni trošak je trošak generatora specijalizacije.
6. `Operation` podržava samo `Start`; zaustavljanje, pauza i nastavak nisu operacije, nego posljedice iznimke i snimke.
7. `__del__` zove `gc.collect()`: izmjereno 1 do 11 ms po uništenju procesora (11 ms uz milijun živih objekata); zahtjev ga ne traži, a trošak je zanemariv pri jednom uništenju po prolazu.
8. `write_context` je točka proširenja: generička izvedba vraća prazan rječnik, ključeve koje specijalizacija objavi dobiva svaki `flush`. Nijedna produkcijska specijalizacija ga danas ne nadjačava; ispitan je testom s nadjačanom izvedbom.
9. `Processed` potvrđuje transformaciju stavke; da je i pohranjena (flush) kaže `flush_outcome`. Uz `flush_per_cycle=False` zapis ne tvrdi pohranu.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — konstruktor izdvaja `blackboard`, `pipelines`, `flush_per_cycle` iz `**kwargs`, pa
   ostatak prosljeđuje naviše. `defer_flush` nije ključ procesora: `PresetGate` ga prijavi kao
   nepoznat i odbaci, a `flush_per_cycle` ostaje kakav je bio.
2. Automat se gradi ovdje, u generičkoj klasi: `StateMachine(TRANSITIONS, ProcessorState.IDLE,
   name="ProcessorFSM")`.
3. **EV03** — `operation(Operation.Start)` je jedina podržana operacija; svaka druga daje
   `warning` i `False` umjesto iznimke.
4. `start()` prvo provjerava blackboard pa pipelinee. Tek **nakon** provjera piše otvarajući
   `INFO` — zapis tako imenuje ono što će stvarno raditi, a ne ono što je zatraženo. Konfiguracija
   (tipovi blackboarda, pipelinea i spremišta) ide u **otvarajući** zapis, jer je operateru
   korisna dok još može djelovati; imena idu kao spojeni string, ne lista, jer audit renderer
   kolekciju na `INFO` sažima u `<list: N>` i sakrio bi upravo ta imena.
5. Generator se gradi ako ne postoji; `START` prevodi automat u `RUNNING`.
6. **EV04–EV05** — za svaku stavku: `NEXT_ITEM`, pa redom svi pipelinei, pa `_cycle += 1`,
   `CYCLE_COMPLETED`, i — ako je `flush_per_cycle` — `blackboard.flush(caller=self,
   **self.write_context)`. Ključeve koje `write_context` objavljuje čitaju write strategije;
   generička implementacija vraća prazan rječnik, pa procesor koji ništa ne deklarira ne mijenja
   nijedan poziv (vidi odjeljak 15 t.5).
7. **Granica dokumenta** — tek **iza** flusha procesor piše `INFO` s `msg=Event.Processed`, bez
   `step`-a, s poljima `cycle`, `source` i `document`. Dokument je jedinica posla, a procesor
   njezin vlasnik: on dovodi stavku, provlači je kroz **sve** pipelinee i određuje granicu flusha
   (t.3). Zapis stoji iza flusha da uz `flush_per_cycle=True` potvrdi i
   transformaciju i pohranu; uz `flush_per_cycle=False` potvrđuje samo transformaciju — deklarirano
   ograničenje, vidi odjeljak 15 t.6.
   Procesor uz to pamti **ishod flusha** (`_flush_outcome`): briše ga na početku svake
   stavke, a upisuje ga samo ako je `flush` vratio `bool`; inače i bez flusha ostaje `None`
   („nepoznato”). Čita ga specijalizacija koja uklanja izvor (`FRQ-PRC-25.6`).
8. **EV06** — kad su sve stavke gotove, a `flush_per_cycle` je isključen i bilo je stavki, platno se prazni jednom (`flush`); zatim `RECORDS_PROCESSED` prevodi automat u `COMPLETED`.
9. Zatvarajući `INFO` nosi **samo ishod** (`cycles`), pod `msg=Event.Completed`. Otvarajući i
   zatvarajući zapis razlikuju se **imenom zapisa**, ne poljem `step` — inače operater mora
   čitati polje da bi znao koji od dva zapisa gleda.

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericProcessor

  participant "ProcessorManager" as O
  participant "GenericProcessor" as P
  participant "GenericPipeline" as L
  participant "GenericBlackboard" as B
  activate O
  O -> P : operation(Operation.Start)
  activate P
  P -> P : START, RUNNING
  loop each item of the generator
    P -> P : NEXT_ITEM
    loop each pipeline
      P -> L : process(processor, facade)
      activate L
      deactivate L
    end
    P -> P : CYCLE_COMPLETED
    opt flush_per_cycle
      P -> B : flush(caller, **write_context)
      activate B
      deactivate B
    end
    opt memento_store set
    P -> P : checkpoint
  end
  P -> P : INFO Processed
  end
  opt not flush_per_cycle and items were processed
    P -> B : flush(caller, **write_context)
  end
  P -> P : RECORDS_PROCESSED, COMPLETED
  P -> P : INFO Completed (cycles)
  P --> O : True
  deactivate P
  deactivate O
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| nema blackboarda ili pipelinea | `ProcessorException` prije otvarajućeg zapisa | prolaz ne počinje (`BR-WFL-02`) |
| bilo što padne unutar obrade stavke (pipeline, `flush`, zapis `Processed`) | `FAIL` → `FAILED`, pa `PipelineException(caller=self, …) from e` | prolaz staje; uzrok očuvan; stanje je `FAILED`, dakle nastavljivo (`BR-PRC-01`) |
| `PipelineException` u vanjskom `except` | `FAIL` (ako ga automat dopušta), pa se **ponovno diže nepromijenjen** | ne omata se dvaput |
| kvar izvan obrade stavke (generator, `START`, `NEXT_ITEM`, `RECORDS_PROCESSED`) | `FAIL` (ako ga automat dopušta) → `ProcessorException(caller=self, error=…) from e` | stanje je `FAILED`, dakle nastavljivo (`BR-PRC-01`) |
| `flush_per_cycle` isključen | jedan `flush` nakon zadnje stavke (samo ako je bilo stavki i samo na uspjehu); kvar tog flusha je `ProcessorException` i `FAILED` | platno se ne gubi neispražnjeno |
| nepodržana operacija | `warning` + `False` | pogrešna konfiguracija ne ruši prolaz |
| snimka iz stanja iz kojeg `LOAD` nije dopušten | `ProcessorException` **prije** izmjene stanja | automat se ne kvari polovičnim vraćanjem |
| snimka traži više ciklusa nego skup ima | `StopIteration` → `ProcessorException("dataset shorter than saved cycle")` | nastavak nad promijenjenim skupom pada glasno |
| iznimka u `__del__` | `error(...)`, pa `finally: gc.collect()`; konstrukcija koja nije završila (članovi nepostavljeni) vraća se odmah | destruktor ne diže i ne ostavlja šum ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §4 t.4) |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericProcessor

start
:start();
if (blackboard and pipelines registered?) then (no)
  :<b><color:red>FAILED: ProcessorException</color></b>;
  kill
endif
:INFO opening record;
if (memento_store set and no generator yet?) then (yes)
  :read checkpoint, restore_state;
endif
:create generator, START -> RUNNING;
while (generator yields an item?) is (yes)
  :NEXT_ITEM, run through all pipelines;
  if (item raised?) then (yes)
    :FAIL -> FAILED;
    :<b><color:red>FAILED: PipelineException</color></b>;
    kill
  endif
  :_cycle += 1, CYCLE_COMPLETED;
  if (flush_per_cycle?) then (yes)
    :blackboard.flush(caller, **write_context);
  endif
  if (memento_store set and cycle due?) then (yes)
    :write checkpoint (cycle, FAILED);
  endif
  :INFO Processed;
endwhile (no)
if (not flush_per_cycle and items processed?) then (yes)
  :blackboard.flush(caller, **write_context);
endif
:RECORDS_PROCESSED -> COMPLETED;
if (memento_store set?) then (yes)
  :clear checkpoint;
endif
:INFO Completed (cycles);
stop
@enduml
```

## 10. State Machine

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption State Diagram
title GenericProcessor

  [*] -down-> IDLE
  IDLE -down-> STATE_LOADED : LOAD
  IDLE -right-> RUNNING : START
  STATE_LOADED -right-> RUNNING : START
  RUNNING -down-> RUNNING : NEXT_ITEM
  RUNNING -up-> RUNNING : CYCLE_COMPLETED
  RUNNING -right-> COMPLETED : RECORDS_PROCESSED
  RUNNING -down[#$WF_ERROR,thickness=2]-> FAILED : FAIL
  FAILED -left-> STATE_LOADED : LOAD
  COMPLETED -down-> COMPLETED : STORE
  FAILED -down-> FAILED : STORE
@enduml
```

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Processor` je rezervirani primitiv; on posjeduje ciklus, a pipeline transformaciju |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | tri `INFO` točke: dvije granice prolaza i jedna granica dokumenta; sve ostalo `DEBUG`, osim deprecation `warning` |
| [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) | otvarajući zapis nosi `blackboard`, `pipelines`, `repositories`; po-dokumentu `cycle`, `source`, `document`; zatvarajući samo `cycles` |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | volumen je `2 + N` — procesor posjeduje **dvije ugniježđene** jedinice, prolaz i dokument |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | procesor ne poznaje spremišta osim kroz platno; `_repository_names` je samo za zapis i nikad ne diže |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; uvozi samo `gc`, `abc`, `enum`, `typing`, `collections.abc` + `wattleflow.*` |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | petlja, automat i granice kvara žive ovdje, ne prepisane po specijalizacijama |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Skup stavki je prošao kroz sve pipelinee, platno je ispražnjeno na granici koju određuje
`flush_per_cycle`, a audit tok nosi **`2 + N`** `INFO` zapisa: otvarajući s konfiguracijom, po
jedan za svaki od *N* dovršenih dokumenata, i zatvarajući s brojem ciklusa. Broj konfiguriranih
pipelinea u toj formuli ne sudjeluje. Prolaz prekinut bilo gdje nakon `START` ostaje u stanju `FAILED`, iz kojeg vodi
nastavak preko snimke. Dokument na kojem je stao **nema** svoj zapis, pa se neuspjeh čita po
odsutnosti.

## 13. Acceptance Criteria

1. `create_generator()` je jedina apstraktna metoda. ✅
2. Automat se gradi u generičkoj klasi i primjenjuje na svaki prijelaz. ✅
3. `start()` provjerava preduvjete **prije** otvarajućeg zapisa. ✅
4. `2 + N` `INFO` zapisa po prolazu (`N` = dovršeni dokumenti); sva tri oblika razlikuju se
   `msg`-om, nijedan ne nosi `step` ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) k.1). ✅
5. `restore_state` provjeri cijelu snimku (`state` je `ProcessorState` s dopuštenim `LOAD`-om, `cycle` je nenegativan `int`) prije nego dira stanje, a stanje automata obnavlja novim `StateMachine`. ✅ — [`FRQ-MEM`](FRQ-MEM-memento.md) kriterij 10
6. `PipelineException` se ne omata dvaput. ✅
7. `defer_flush` nije ključ procesora: prijavljuje se kao nepoznat i ne mijenja `flush_per_cycle`. ✅
8. `__del__` ne diže iznimku ni u jednoj grani. ✅
9. Snimka nosi dovoljno za nastavak nad **istim** skupom; uz konfiguriranu pohranu procesor sam sprema nakon ciklusa (`memento_store`, `checkpoint_every`) i nastavlja pri `start()`. ✅ — uz pretpostavku iz odjeljka 07 t.5
10. Raspored flusha: uz `flush_per_cycle` jedan po stavci; bez njega točno jedan na kraju, samo ako je bilo stavki i samo na uspjehu. Kvar tog flusha je `ProcessorException` i `FAILED`. ✅
11. Kvar u obradi stavke ostavlja automat u `FAILED` (ne u `RUNNING`), pa je prolaz nastavljiv. ✅
12. `__del__` ne ostavlja šum ni nakon konstrukcije koja nije završila. ✅
13. `write_context` svake specijalizacije stiže do svakog `flush`. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | `workflow/tests/test_processor.py` (`PreconditionTest`) | `__abstractmethods__ == {create_generator}` |
| 2 | `PassTest` | `IDLE` → `COMPLETED` nakon prolaza; svaka stavka kroz sve pipelinee redom |
| 3 | `PreconditionTest` | `assertNoLogs(INFO)`: bez blackboarda i bez pipelinea `ProcessorException` prije otvarajućeg zapisa |
| 4 | `PassTest` | točno `2 + N` `INFO` zapisa |
| 5 | `test_processor_restore.py` (9 testova) | odbijanje bez `state`/`cycle`, pogrešnog tipa i bez dopuštenog `LOAD`-a; odbijena snimka ništa ne mijenja; ispravna nastavlja iza spremljenog ciklusa |
| 6 | `FailureTest` | uzrok nije `PipelineException` |
| 7 | `test_processor_flush_key.py` | `defer_flush` se prijavljuje kao odbačen ključ |
| 8, 12 | `DestructorTest` | ponovljen `__del__` i `__del__` neizgrađenog objekta ne dižu |
| 9 | `test_processor_checkpoint.py` (16 testova) | pad i nastavak (memorija i datoteka), brisanje nakon uspjeha, kvarovi pohrane — [`FRQ-MEM`](FRQ-MEM-memento.md) |
| 10 | `FlushScheduleTest` | `ITEMS` flushova po zadanom; jedan bez `flush_per_cycle`; nijedan bez stavki i na padu; kvar završnog flusha je `FAILED` |
| 11 | `FailureTest` | `FAILED` nakon kvara pipelinea |
| 13 | `FlushScheduleTest` | `skip="x"` i `caller` u svakom pozivu; zadano `{}`; ishod flusha (`True`, `False`, drugo → `None`) |
| mutacije | ručno | bez završnog flusha, završni flush i bez stavki, kvar stavke ostaje `RUNNING`, nezaštićen `__del__` — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-PRC-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/processor.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka obuhvaća kontrolnu točku, završni flush i `FAILED` pri kvaru stavke. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno u cjelini (`test_processor.py`, 23 testa), svi defekti zatvoreni: **`flush_per_cycle=False` nikad nije ispraznio platno** (dokument je obećavao „tek na kraju”, a koda za to nije bilo; uz zadani `defer_flush` blackboarda podaci se nisu zapisivali) — dodan jedan završni `flush` na uspjehu; `DEF-PRC-07` kvar u obradi stavke sada vodi u `FAILED`; `__del__` neizgrađenog objekta ne ostavlja šum. Ostale stavke zatvorene kao zapisana svojstva ili izmjereno: `DEF-PRC-01` (isti skup, odjeljak 07 t.5), `-02` (premotavanje 90 ns po stavci), `-03` (samo `Start`), `-04` (`gc.collect` 1–11 ms), `-05` (`write_context` ispitan nadjačavanjem), `-06` (`Processed` ne tvrdi pohranu). Kriteriji 10–13, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-04 | Kontrolna točka i nastavak (`memento_store`, `memento_key`, `checkpoint_every`; odbija pohranu bez `flush_per_cycle`), `restore_state` ispravljen i validira snimku; kriteriji 5 i 9, EV08, odjeljak 15 t.7 usklađeni s kodom ([`FRQ-MEM`](FRQ-MEM-memento.md)). |
| v0.0.5 | 2026-10-03 | Zastarjeli `defer_flush` uklonjen iz procesora (EV01, kriterij 7); odjeljak 15 t.3 i t.8 riješene odnosno izvan opsega, ostale renumerirane. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa i poveznice izvan distribucije `workflow`. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: pokretač prolaza je ProcessorManager (umjesto GenericWorkflow/Orchestrator), potpisi operation/start/register_*/restore_state/create_generator, write_context : dict[str, Any]. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | Dodan dijagram stanja za `GenericProcessor` (tablica prijelaza `TRANSITIONS`). |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — obrnuto inženjerstvo zatečenog. Procesor piše i granicu **po dokumentu** |
