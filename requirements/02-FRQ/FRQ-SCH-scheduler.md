<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SCH — Raspoređivač

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `SCH` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-02`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `Scheduler(Wattleflow, IScheduler, ABC)` — izvor događaja koji vodi jedan orkestrator: postavlja ga, pokreće, zaustavlja i javlja slušateljima |
| **Sestrinski** | [`FRQ-ORC`](FRQ-ORC-orchestrator.md) (kojeg vodi) · [`FRQ-PTN`](FRQ-PTN-root-base.md) (korijen) |
| **Izvedba** | `workflow/src/wattleflow/concrete/scheduler.py`; potrošač `workflow/src/wattleflow/schedulers/cron_job.py` (`SchedulerCronJob`) |
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

Raspoređivač je ugovor `IScheduler` (`setup_orchestrator`, `start_orchestration`,
`stop_orchestration`) uz `IEventSource` (`register_listener`, `emit_event`). Generička klasa nudi
životni ciklus i emitiranje; **orkestratora ne stvara**: `setup_orchestrator` samo bilježi
konfiguriranje, a `_orchestrator` ostaje `None` dok ga specijalizacija ne postavi (to je ugovor, ne kvar).
Jedini potrošač u distribuciji, `SchedulerCronJob`, orkestratora uopće ne koristi: izvodi vlastitu petlju
`run_once` / `run` i klasu koristi zbog slušatelja i događaja.

| član | uloga |
|---|---|
| `_lock: RLock` | štiti stanje (slušatelje, `_running`, `_counter`), **ne** rad orkestratora; stvara se prije `super().__init__` jer slot zasjenjuje `Audit._lock` |
| `_listeners: list` | slušatelji događaja, bez duplikata |
| `_orchestrator` | vođeni orkestrator; postavlja specijalizacija |
| `_initialised` | štiti od ponovnog početnog stanja |
| `_preset: PresetDecorator` | konfiguracijska imena kroz `__getattr__` |
| `_running`, `_counter` | `running` je istina dok orkestrator radi; `count` broji dovršene orkestracije (neuspjela se ne broji) |

<div align="center">


</div>

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Scheduler

top to bottom direction

interface IEventSource {
  + register_listener(listener : IEventListener)
  + emit_event(event : Event, **kwargs)
}
interface IScheduler {
  + setup_orchestrator()
  + start_orchestration(parallel : bool)
  + stop_orchestration()
}
interface IEventListener {
  + on_event(event : IEvent)
}

abstract class Scheduler {
  # _lock : RLock
  # _listeners : list[IEventListener]
  # _orchestrator : Orchestrator | None
  + count : int
  + running : bool
  + setup_orchestrator()
  + start_orchestration(parallel : bool = False)
  + stop_orchestration()
  + register_listener(listener : IEventListener)
  + emit_event(event : Event, **kwargs)
}

class SchedulerCronJob {
  + heartbeat : int
  + passes : int
  + failures : int
  + run_once() : bool
  + run(cycles : int | None)
}

class Orchestrator {
  + start(parallel : bool)
  + stop()
}
class PresetDecorator

IEventSource <|-- IScheduler
IScheduler <|.. Scheduler
Scheduler <|-- SchedulerCronJob
Scheduler "1" o-- "0..1" Orchestrator
Scheduler "1" o-- "0..*" IEventListener
Scheduler "1" *-- "1" PresetDecorator
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Scheduler

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Scheduler", "wattleflow.concrete.scheduler: event source that drives one orchestrator")

System_Ext(cj, "SchedulerCronJob", "Specialisation; builds and runs a workflow on a heartbeat")
System_Ext(cl, "Caller", "Workflow or specialisation that starts and stops")
System_Ext(or, "Orchestrator", "Runs the processors")
System_Ext(ls, "IEventListener", "Receives events")

Rel_D(cj, sys, "Specialises")
Rel_D(cl, sys, "Starts, stops")
Rel_D(sys, or, "Starts, stops")
Rel_D(sys, ls, "Emits events")

Lay_R(cj, cl)
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | pozivatelj (workflow, specijalizacija) | `level`, `handler`, preset ključevi; `start_orchestration(parallel)` |
| **A2** | `Orchestrator` | `start(parallel)`, `stop()` |
| **A3** | `IEventListener` | `on_event(event, **kwargs)` |
| **A4** | specijalizacija | postavlja `_orchestrator` (nadjačava `setup_orchestrator`) |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 gradi raspoređivač → `__init__` → `setup_orchestrator()` |
| **EV02** | A1 registrira slušatelja |
| **EV03** | A1 pokreće → `start_orchestration(parallel)` |
| **EV04** | A1 zaustavlja → `stop_orchestration()` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Scheduler

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor Caller as CALLER
  actor Specialisation as SPECIALISATION
  actor Orchestrator as ORCHESTRATOR
  actor IEventListener as LISTENER

  usecase "Set up orchestrator" as EV01
  usecase "Register listener" as EV02
  usecase "Start orchestration" as EV03
  usecase "Stop orchestration" as EV04
  usecase "Receive event" as EV05

  SPECIALISATION --> EV01
  CALLER --> EV02
  CALLER --> EV03
  CALLER --> EV04
  ORCHESTRATOR --> EV03
  ORCHESTRATOR --> EV04
  LISTENER --> EV05
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. `level` je predan (obvezan); `handler` je izboran.
2. Specijalizacija je postavila `_orchestrator`, inače su `start`/`stop` bez učinka.
3. Slušatelj smije dizati iznimke: bilježe se i ne prekidaju emitiranje; slušatelj koji prima samo `on_event(event)` bez `**kwargs` svaki put pada (bilježi se). Slušatelj se prijavljuje po identitetu.
4. Specijalizacija koja ne postavi `_orchestrator` ne dobiva ni događaje ni brojilo od `start`/`stop`; `SchedulerCronJob` zato emitira vlastite događaje.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — `_lock` se stvara ako ga nema; `super().__init__` dobiva `level`, `handler` i ostatak ključeva;
   `PresetDecorator` preuzima ključeve; pri prvom pozivu postavlja se početno stanje i zove `setup_orchestrator`.
2. **EV03** — ako orkestrator postoji i ne radi: pod bravom `_running = True`; izvan brave `emit_event(Started)`,
   `orchestrator.start(parallel)`; zatim `_running = False`, `_counter += 1`, `emit_event(Completed)`. Pad orkestratora
   zatvara `Started` s `emit_event(Failed, error=…)` i prolazi dalje.
3. **EV04** — `emit_event(Stopped)`, pa `orchestrator.stop()`; ne čeka završetak `start` koji teče u drugoj dretvi.
4. `emit_event` uzme kopiju popisa pod bravom, a `on_event(event, **kwargs)` zove izvan nje, redom; slušatelj koji padne
   bilježi se (`exception`, `Notify`) i ne prekida ostale.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Scheduler

actor Caller
participant Scheduler
participant Orchestrator
participant "IEventListener" as Listener

Caller -> Scheduler : start_orchestration(parallel)
activate Scheduler
alt orchestrator not set or already running
  Scheduler --> Caller : no effect
else
  Scheduler -> Listener : on_event(Started)
  Scheduler -> Orchestrator : start(parallel)
  activate Orchestrator
  alt completed
    Orchestrator --> Scheduler : done
    Scheduler -> Listener : on_event(Completed)
    Scheduler --> Caller : done
  else raises
    Orchestrator --> Scheduler : exception
    Scheduler -> Listener : on_event(Failed)
    Scheduler --> Caller : exception
  end
  deactivate Orchestrator
end
deactivate Scheduler
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `_orchestrator` je `None` | `start_orchestration` i `stop_orchestration` ne rade ništa, bez poruke |
| orkestrator digne iznimku pri `start` | emitira se `Failed` s `error`; `Completed` ne; `running` se vraća na `False`, `count` se ne mijenja; iznimka prolazi |
| `start_orchestration` dok orkestrator radi | zanemaruje se (`_running`) |
| `stop_orchestration` dok `start` radi u drugoj dretvi | izvodi se odmah; brava se ne drži za vrijeme rada orkestratora |
| slušatelj digne iznimku | `exception(msg=Notify, reason, listener, error)`; sljedeći slušatelji dobivaju događaj, emiter ne vidi iznimku |
| slušatelj se prijavi tijekom isporuke | dobiva tek sljedeći događaj |
| `__init__` pozvan ponovno | početno stanje se ne vraća (`_initialised`) |
| pristup imenu s `_` prefiksom koje nije postavljeno | `AttributeError` |
| ostala nepostojeća javna imena | razrješava ih preset |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
title Activity Diagram: Scheduler, Orchestrator

|Caller|
start
:start_orchestration(parallel);
|Scheduler|
if (orchestrator set and not running?) then (yes)
  :_running = True;
  :emit_event(Started);
  |Orchestrator|
  :start(parallel);
  |Scheduler|
  :_running = False;
  if (start raised?) then (yes)
    :emit_event(Failed);
    :<b><color:red>FAILED: exception</color></b>;
    kill
  endif
  :_counter += 1;
  :emit_event(Completed);
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
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | konstruktor je `DEBUG`; događaji idu slušateljima, ne auditu |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | uvozi samo `threading` i `wattleflow` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 4 |

## 12. Results

Slušatelji vide `Started`/`Completed`/`Stopped` oko rada orkestratora, a specijalizacija ne
mora ponavljati bravu ni popis slušatelja.

## 13. Acceptance Criteria

1. `register_listener` ne dodaje isti objekt dvaput, a dva jednaka, ali različita slušatelja oba primaju događaje (identitet). ✅
2. Brava štiti stanje, ne rad orkestratora: `stop_orchestration` se izvodi dok `start` radi. ✅
3. Modul deklarira `__all__`. ✅
4. Svi atributi koje klasa postavlja nalaze se u `__slots__` (`BR-PTN-07`); neiskorišteni slotovi su uklonjeni. ✅
5. `setup_orchestrator` ne stvara orkestrator: zadano ponašanje je bilježenje, a `_orchestrator` postavlja specijalizacija. ✅
6. `count` vraća broj dovršenih orkestracija, `running` je istina dok orkestrator radi. ✅
7. `Started` je uvijek zatvoren: `Completed` kad uspije, `Failed` kad padne; iznimka prolazi. ✅
8. Slušatelj koji padne ne prekida emitiranje i ne dolazi do emitera; isporuka je izvan brave. ✅
9. Neuspjeli prolaz `SchedulerCronJob` ne zaustavlja petlju, ni kad slušatelj njegova događaja padne. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 8 | `workflow/tests/test_scheduler.py` (`ListenerTest`) | identitet, redoslijed, izolacija kvara, isporuka izvan brave (druga dretva se prijavljuje tijekom isporuke), prijava tijekom isporuke |
| 2, 7 | `LifecycleTest`, `StopWhileRunningTest`, `CounterTest` | `Started`→`Completed` / `Failed`, ponovni start nakon pada, `stop` ne čeka `start`, istodobni drugi `start` se zanemaruje; prije ispravka `stop` je čekao kraj `start` |
| 3–5 | `ConstructionTest`, `SlotsTest` | `_preset` je slot, `_tasks` i `_config` nisu; bez orkestratora `start`/`stop` ne rade ništa; drugi `__init__` ne vraća početno stanje |
| 6 | `CounterTest` | broji dovršene, ne neuspjele; `running` točno tijekom i nakon rada, i nakon pada |
| 9 | `workflow/tests/test_scheduler_cron.py` | 3 prolaza s padom i slušateljem koji pada: 3 prolaza, 3 pada, 2 spavanja |
| mutacije | ručno | kvar slušatelja ne izoliran, bez `Failed`, `running` se ne resetira, bez brojila, bez zaštite od dvostrukog starta, jednakost umjesto identiteta — svaka ruši test |

**Trojka (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-SCH-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/scheduler.py`; potrošač `workflow/src/wattleflow/schedulers/cron_job.py` (`SchedulerCronJob`)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (class, context, use case, sequence, activity): bez `frame`, `package` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu, raspored `LAYOUT_TOP_DOWN`; sequence s uravnoteženim aktivacijama i povratkom nakon ishoda; svi renderirani s PlantUML 1.2026.8. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (D-11 uklonjen), svi defekti zatvoreni: `DEF-SCH-01` kostur zatvoren kao ugovor (specijalizacija postavlja orkestrator; `SchedulerCronJob` ga ne koristi); `DEF-SCH-02` `count` je sada broj dovršenih orkestracija, dodan `running`, uklonjeni mrtvi slotovi `_tasks` i `_config`; `DEF-SCH-03` `Started` se zatvara s `Failed`; `DEF-SCH-04` slušatelj koji padne bilježi se i ne prekida emitiranje, isporuka izvan brave (netočna tvrdnja o razlici prema `Orchestrator.emit_event` uklonjena: tada ni on nije hvatao iznimke; od [`FRQ-ORC`](FRQ-ORC-orchestrator.md) isto izolira slušatelje); `DEF-SCH-05` `_preset` u slotovima; `DEF-SCH-06` `ABC` bez apstraktnih metoda zatvoren kao svojstvo (`SchedulerCronJob` ne nadjačava `setup_orchestrator`). **Novo:** `DEF-SCH-07` `stop_orchestration` čekao je kraj `start` (brava za cijeli rad) — zatvoren; `DEF-SCH-08` pad slušatelja mogao je ugasiti petlju `SchedulerCronJob` (emitira iz vlastitog `except`) — zatvoren; slušatelji po identitetu (kao `FRQ-OBS`). Kriteriji 1–9, 27 testova, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: potpisi IScheduler i Scheduler, sudionik IEventListener. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — kostur: izvođenje ovisi o specijalizaciji (odjeljak 15 t.1–3) |
