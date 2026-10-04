<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-ORC — Orkestrator

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `ORC` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-02`, `BR-PTN-05`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `Orchestrator(Wattleflow, IEventSource, IFacade)` — pokreće skup procesora, sekvencijski ili paralelno, i o tijeku obavještava slušatelje |
| **Sestrinski** | [`FRQ-SCH`](FRQ-SCH-scheduler.md) (zove `start`/`stop`) · [`FRQ-PRC`](FRQ-PRC-processor.md) (pokretani) · [`FRQ-MGR`](FRQ-MGR-managers.md) (primljeni `ConnectionManager`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/orchestrator.py` (212 linija) |
| **Dijagrami** | inline (02, 03, 06, 08, 09) — pogledi izvedeni iz koda, ne izvor istine (D-13) |

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

Orkestrator drži popis procesora i popis slušatelja. `start` pokreće svaki procesor kroz njegov
`start()` (ili, za procesor koji izlaže samo fasadu, kroz `operation(Start)`), mjeri trajanje i
emitira `Processed` ili `Failed`. Sučelje `IFacade` svodi se na `operation(Start | Stop)`.
Događaji idu **slušateljima** (`emit_event`), a trag ide u **audit** (`info`/`debug`): dva toka s
različitim čitateljima.

| član | uloga |
|---|---|
| `_processors: list[IProcessor]` | što se pokreće, po redoslijedu registracije |
| `_listeners: list[IEventListener]` | tko prima događaje; bez duplikata |
| `_running: bool` | zastavica izvođenja |
| `_emit_lock: Lock` | štiti popis slušatelja (registracija i izrada kopije za isporuku) |
| `_state_lock: Lock` | štiti `_running` i popis procesora: postavljanje i brisanje zastavice te registracija su atomični |
| `_connection_manager`, `_strategy_execute` | primljeni u konstruktoru i provjereni (`AttributeException`), izloženi svojstvima `connection_manager` i `strategy_execute` za specijalizacije; **orkestrator ih sam ne koristi** |

<div align="center">


</div>

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Orchestrator

top to bottom direction

interface IEventSource {
  + register_listener(listener : IEventListener)
  + emit_event(event : Event, **kwargs)
}
interface IFacade {
  + operation(action : Any)
}
interface IProcessor
interface IEventListener {
  + on_event(event : IEvent)
}
interface IStrategy

class Wattleflow

class Orchestrator {
  # _processors : list[IProcessor]
  # _listeners : list[IEventListener]
  # _running : bool
  # _emit_lock : Lock
  # _state_lock : Lock
  + connection_manager : ConnectionManager
  + strategy_execute : IStrategy | None
  + running : bool
  + add_processor(processor : IProcessor)
  + register_listener(listener : IEventListener)
  + emit_event(event : Event, **kwargs)
  + operation(action : Operation, **kwargs)
  + start(parallel : bool = False)
  + stop()
}

class ConnectionManager
class OrchestratorException

IEventSource <|.. Orchestrator
IFacade <|.. Orchestrator
Wattleflow <|-- Orchestrator
Orchestrator "1" o-- "0..*" IProcessor
Orchestrator "1" o-- "0..*" IEventListener
Orchestrator "1" o-- "1" ConnectionManager
Orchestrator "1" o-- "0..1" IStrategy
Orchestrator ..> OrchestratorException : raises
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Orchestrator

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Orchestrator", "wattleflow.concrete.orchestrator: starts a set of processors in order or in parallel and reports each step")

System_Ext(cl, "Caller", "Scheduler or workflow")
System_Ext(pr, "IProcessor", "Started processor")
System_Ext(ls, "IEventListener", "Receives events")

Rel_D(cl, sys, "Registers, starts, stops")
Rel_D(sys, pr, "Calls start")
Rel_D(sys, ls, "Emits events")

Lay_R(pr, ls)
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | pozivatelj (`Scheduler`, workflow) | `start(parallel)`, `stop()`, `operation(action)` |
| **A2** | `IProcessor` | `start()` ili `operation(Start)` |
| **A3** | `IEventListener` | `on_event(event, **kwargs)` |
| **A4** | `ConnectionManager` | predan konstruktoru i izložen svojstvom; ne sudjeluje u toku |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 registrira procesor → `add_processor` |
| **EV02** | A1 registrira slušatelja → `register_listener` |
| **EV03** | A1 pokreće → `start(parallel)` ili `operation(Operation.Start)` |
| **EV04** | A1 zaustavlja → `stop()` ili `operation(Operation.Stop)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Orchestrator

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor Caller as CALLER
  actor IProcessor as PROCESSOR
  actor IEventListener as LISTENER

  usecase "Register processor" as EV01
  usecase "Register listener" as EV02
  usecase "Start processors" as EV03
  usecase "Stop" as EV04
  usecase "Receive event" as EV05

  CALLER --> EV01
  CALLER --> EV02
  CALLER --> EV03
  CALLER --> EV04
  PROCESSOR --> EV03
  LISTENER --> EV05
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. `connection_manager` je predan konstruktoru (obvezan, mora biti `ConnectionManager`; inače `AttributeException`).
2. Svaki procesor ima pozivljiv `start()` — provjerava se pri registraciji (`OrchestratorException`); isti procesor se registrira jednom (identitet).
3. `strategy_execute` je izborni (`IStrategy`); orkestrator ga drži za specijalizacije i ne koristi.
4. Slušatelj prima `on_event(event, **metadata)` ako to potpis dopušta, inače samo `on_event(event)`; o tome odlučuje potpis, ne iznimka.

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | `start`: pod `_state_lock` provjera i postavljanje `_running`; ako je već postavljen, vraća se bez učinka |
| 2 | `info` i `emit_event(OrchestrationStarted)` |
| 3 | sekvencijski: redom svaki procesor, uz provjeru `_running` prije svakog; paralelno: jedna dretva po procesoru, pa `join` |
| 4 | po procesoru: `start()` (ili `operation(Start)`), trajanje u sekundama, `emit_event(Processed, processor, duration)` |
| 5 | `finally`: pod `_state_lock` `_running = False`; `info` i `emit_event(OrchestrationCompleted)` |



### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Orchestrator

actor Caller
participant Orchestrator
participant IProcessor as Processor
participant IEventListener as Listener

Caller -> Orchestrator : start(parallel)
activate Orchestrator
alt already running
  Orchestrator --> Caller : no effect
else
  Orchestrator -> Listener : on_event(OrchestrationStarted)
  loop each processor
    Orchestrator -> Processor : start()
    alt completed
      Processor --> Orchestrator : done
      Orchestrator -> Listener : on_event(Processed, processor, duration)
    else raises
      Processor --> Orchestrator : exception
      Orchestrator -> Listener : on_event(Failed, processor, error)
    end
  end
  Orchestrator -> Listener : on_event(OrchestrationCompleted)
  Orchestrator --> Caller : done or OrchestratorException
end
deactivate Orchestrator
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| procesor bez `start()` | `add_processor` diže `OrchestratorException` |
| isti procesor ili slušatelj dvaput | drugi poziv ne mijenja popis (identitet) |
| procesor padne | `emit_event(Failed)`, `debug`, `OrchestratorException ... from e` (`BR-PTN-05`); sekvencijski: ostali procesori se ne pokreću |
| procesori padnu paralelno | sve dretve dovrše; jedan pad podiže se kakav jest, više padova jednim `OrchestratorException` koji imenuje svaki |
| `stop()` za vrijeme izvođenja | `_running = False`, `emit_event(OrchestrationStopped)`; sekvencijski petlja staje prije sljedećeg procesora; procesor koji već radi, i paralelne dretve, nisu prekidivi |
| slušatelj diže iznimku | bilježi se (`exception`, `Notify`); ostali slušatelji čuju događaj, a procesor koji je radio ne postaje neuspjeh |
| slušatelj prima samo `on_event(event)` | dobiva događaj bez metapodataka; `TypeError` iznutra se ne ponavlja |
| nepoznata operacija | `OrchestratorException` |
| `operation(Start, parallel=True)` | prosljeđuje se `start(parallel=True)` |
| procesor izlaže ni `start()` ni `operation()` | `OrchestratorException` pri pokretanju |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Orchestrator

start
:start(parallel);
if (not running?) then (yes)
  :_running = True;
  :emit OrchestrationStarted;
  if (parallel?) then (yes)
    :start every processor in its own thread;
    :wait for all threads;
  else (no)
    while (more processors and running?) is (yes)
      :start next processor;
      if (processor raised?) then (yes)
        :emit Failed;
        :<b><color:red>FAILED: OrchestratorException</color></b>;
        break
      endif
      :emit Processed;
    endwhile (no)
  endif
  :_running = False;
  :emit OrchestrationCompleted;
  if (any processor failed?) then (yes)
    :raise OrchestratorException naming every failure;
  endif
endif
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | `info` za početak i kraj orkestracije, `debug` za neuspjeh procesora |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | uvozi samo `threading` i `wattleflow` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 5 |

## 12. Results

Svi registrirani procesori pokrenuti su po redoslijedu (ili istodobno), a slušatelji su dobili
`Started`, po jedan `Processed` ili `Failed` po procesoru te `Completed`, i kad je izvođenje palo.

## 13. Acceptance Criteria

1. `add_processor` odbija objekt bez pozivljivog `start()` (`OrchestratorException`) i ne dodaje isti procesor dvaput. ✅
2. `register_listener` ne dodaje isti objekt dvaput; dva jednaka, ali različita slušatelja oba primaju događaje (identitet). ✅
3. `OrchestrationCompleted` se emitira uvijek (`finally`), a `running` se vraća na `False` i nakon pada. ✅
4. Kvar procesora izlazi kao `OrchestratorException` s uzrokom (`BR-PTN-05`); paralelno se čekaju sve dretve, a svi padovi su imenovani. ✅
5. Modul deklarira `__all__`; klasa deklarira `__slots__` sa svim članovima (`BR-PTN-07`). ✅ — učinak ovisi o `__slots__` u sučeljima `core` ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md))
6. Dva istodobna `start` ne pokreću procesore dvaput: provjera i postavljanje `_running` je atomično. ✅
7. `connection_manager` i `strategy_execute` se provjeravaju i izlažu, a orkestrator ih ne koristi. ✅
8. `operation(Start, parallel=True)` dostupno kroz fasadu. ✅
9. `stop` zaustavlja sekvencijski niz prije sljedećeg procesora; procesori koji već rade (i paralelne dretve) nisu prekidivi. ✅
10. Slušatelj koji padne ne dira tok: ne pretvara uspjeh procesora u neuspjeh, ne skriva izvornu grešku i ne sprječava ostale slušatelje; potpis odlučuje o metapodacima. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2 | `workflow/tests/test_orchestrator.py` (`RegistrationTest`) | odbijanje bez `start()`, identitet procesora i slušatelja |
| 3, 4 | `SequentialTest`, `ParallelTest` | redoslijed i događaji, pad zaustavlja ostale, `Completed` nakon pada, ponovni start; paralelno svi dovrše, svi padovi imenovani |
| 5 | pregled modula | `__all__`, `__slots__` sa svim članovima |
| 6 | `StartGuardTest`, `StateLockTest` | drugi `start` za vrijeme rada se zanemaruje; zahtjev i oslobađanje pod `_state_lock` (utrka se ne može izazvati pa je zaštita provjerena izravno) |
| 7 | `ConstructionTest` | tipovi, svojstva samo za čitanje |
| 8, 9 | `ParallelTest`, `SequentialTest` | `operation(..., parallel=True)`, `stop` između procesora, procesor u radu nije prekinut |
| 10 | `ListenerDeliveryTest` | izolacija, potpis, bez ponovnog poziva nakon `TypeError`, isporuka izvan brave |
| mutacije | ručno | slušatelji neizolirani, bez provjere potpisa, duplikati, jednakost slušatelja, samo prva paralelna greška, `operation` bez `parallel`, bez provjere managera, `_running` bez brave — svaka ruši test |

**Trojka (D-10):** alat — čitanje koda; kriterij — §8; platforma — `workflow` radno stablo. Kod nije
izvršavan (D-11).

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-ORC-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/orchestrator.py` (212 linija)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka koji je stajao u odjeljku 08 uklonjen (nosi ga odjeljak 09), sequence prikazuje i pad procesora. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (D-11 uklonjen), svi defekti zatvoreni: `DEF-ORC-01` primljeni suradnici provjereni i izloženi, docstring modula više ne obećava ono što ne radi (zatvoreno kao svojstvo: drže se za specijalizacije); `DEF-ORC-02` `operation(Start, parallel=…)`; `DEF-ORC-03` `stop` ne prekida procesor u radu (svojstvo, kriterij 9); `DEF-ORC-04` `_running` atomičan (`_state_lock`); `DEF-ORC-05` paralelni padovi su svi imenovani; `DEF-ORC-06` potpis odlučuje o metapodacima, `TypeError` iznutra se ne ponavlja; `DEF-ORC-07` procesor i slušatelj registriraju se po identitetu. **Novo:** slušatelj koji padne pretvarao je uspjeh procesora u neuspjeh (`Processed` je bio u istom `try`) i mogao sakriti izvornu grešku (`Completed` u `finally`) — zatvoreno (izolacija kao u `Scheduler`); `add_processor` diže `OrchestratorException` umjesto `TypeError`. Kriteriji 1–10, 29 testova, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: potpisi i tipovi članova Orchestrator, paket wattleflow.concrete.orchestrator, Event.* u porukama. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — dio obećanja modula nije proveden (odjeljak 15 t.1–3) |
