<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-MGR — Upravitelji konekcija, drivera i procesora

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `MGR` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-02`, `BR-PTN-05`, `BR-PTN-07` (`__slots__`), `BR-DRV-01` |
| **Predmet** | `ConnectionManager`, `DriverManager`, `ProcessorManager` (sve `Wattleflow, IObserver`) — registar objekata jedne vrste po imenu, s operacijom nad njima i raspremanjem pri uništenju |
| **Sestrinski** | [`FRQ-CON`](FRQ-CON-connection.md) · [`FRQ-DRV`](FRQ-DRV-driver.md) · [`FRQ-PRC`](FRQ-PRC-processor.md) (upravljani) · [`FRQ-WFL`](FRQ-WFL-workflow.md) (vlasnik) · [`FRQ-ORC`](FRQ-ORC-orchestrator.md) |
| **Izvedba** | [`concrete/manager.py`](../../../workflow/src/wattleflow/concrete/manager.py) |
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

Tri klase dijele isti oblik: rječnik `ime → objekt`, `register_*`, `unregister_*`, `get_*`,
`operation(name, action)` i `__del__` koji upravljane objekte raspreme. Razlikuju se po tome što
je upravljano i koja je operacija dopuštena.

| upravitelj | drži | `operation` prosljeđuje | pri uništenju |
|---|---|---|---|
| `ConnectionManager` | konekcije (`_connections`) | bilo koju `Operation` | `request(Disconnect)` za svaku koja je povezana (isto i pri `unregister_connection`) |
| `DriverManager` | driveri (`_drivers`) | bilo koju `Operation` | `ensure_unloaded()` za svaki (isto i pri `unregister_driver`) |
| `ProcessorManager` | procesori (`_processors`) | samo `Start`; ostalo je upozorenje i `False` | `unregister_processor` dok rječnik ne ostane prazan |

`ConnectionManager.hot_swap(name, new_connection)` zamjenjuje registriranu konekciju: prvo spaja novu, zatim prebacuje promatrače na nju (`transfer_observers`), upisuje je u registar, javlja `Event.Swap` s imenom i novom konekcijom, i tek na kraju odspaja staru. Driver koji je promatrač te konekcije na `Event.Swap` s njezinim imenom ispražnjuje stanje (`GenericDriver` poziva `ensure_unloaded()`, `LazyDriverProxy` poziva `release()`), pa sljedeća uporaba ponovno razrješava konekciju.

**`unregister_*` oslobađa ono što bi `__del__` oslobodio:** konekciju odspaja ako je povezana, driver ispražnjuje (`ensure_unloaded`), a unos nestaje i kad oslobađanje padne (kvar se bilježi). Procesor nema što osloboditi. Sva tri su `IObserver`; `update(*args, **kwargs)` je u sva tri namjerno prazan (bilježi na `DEBUG`).

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title ConnectionManager, DriverManager, ProcessorManager

top to bottom direction

interface IObserver {
  + update(event : Any, **kwargs)
}
class Wattleflow

class ConnectionManager {
  # _connections : dict[str, IObserver]
  + connect(name, **kwargs) : object
  + disconnect(name, **kwargs) : bool
  + get_connection(name) : Connection
  + register_connection(connection, **kwargs)
  + unregister_connection(name)
  + hot_swap(name, new_connection)
  + operation(name, action, **kwargs) : bool
}
class DriverManager {
  # _drivers : dict[str, IDriver]
  + all : dict[str, IDriver]
  + load(name, **kwargs) : object
  + get_driver(name) : IDriver
  + register_driver(driver, **kwargs)
  + unregister_driver(driver)
  + operation(name, action, **kwargs) : bool
}
class ProcessorManager {
  # _processors : dict[str, IProcessor]
  + all : dict[str, IProcessor]
  + load(name, **kwargs) : IProcessor
  + get_processor(name) : IProcessor
  + register_processor(processor, **kwargs)
  + unregister_processor(processor)
  + operation(name, action, **kwargs) : bool
}
class ManagerException

IObserver <|.. ConnectionManager
IObserver <|.. DriverManager
IObserver <|.. ProcessorManager
Wattleflow <|-- ConnectionManager
Wattleflow <|-- DriverManager
Wattleflow <|-- ProcessorManager
ConnectionManager ..> ManagerException : raises
DriverManager ..> ManagerException : raises
ProcessorManager ..> ManagerException : raises
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title ConnectionManager, DriverManager, ProcessorManager

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Manager", "wattleflow.concrete.manager: registers connections, drivers or processors by name and forwards operations")

System_Ext(cl, "Caller", "WorkflowFactory or workflow")
System_Ext(mo, "Managed object", "GenericConnection, GenericDriver or GenericProcessor")

Rel_D(cl, sys, "Registers, looks up, runs")
Rel_D(sys, mo, "Forwards operation, releases")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces |
|---|---|---|
| **A1** | `GenericWorkflow` / `WorkflowFactory` | objekt za registraciju, ime |
| **A2** | upravljani objekt (`GenericConnection`, `GenericDriver`, `GenericProcessor`) | `operation(action, **kwargs)`, `ensure_unloaded()` |
| **A3** | pozivatelj koji dohvaća po imenu | `name` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 registrira → `register_connection` / `register_driver` / `register_processor` |
| **EV02** | A3 dohvaća → `get_*` |
| **EV03** | A1 pokreće operaciju → `operation` (ili `connect`, `disconnect`, `load`) |
| **EV04** | A1 odjavljuje → `unregister_*` |
| **EV05** | uništenje upravitelja → `__del__` |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title ConnectionManager, DriverManager, ProcessorManager

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor Caller as CALLER
  actor "Managed object" as OBJECT

  usecase "Register object" as EV01
  usecase "Get by name" as EV02
  usecase "Run operation" as EV03
  usecase "Unregister object" as EV04
  usecase "Release on discard" as EV05

  CALLER --> EV01
  CALLER --> EV02
  CALLER --> EV03
  CALLER --> EV04
  OBJECT --> EV03
  OBJECT --> EV05
}
@enduml
```

## 07. Constraints and Preconditions

1. Ime je jednoznačno: iz `kwargs` (`connection_name`, `name`) ili iz objekta (`connection_name`, `name`); objekt bez svojstva registrira se ako je ime dano u `kwargs`.
2. Za `ConnectionManager` objekt izlaže `connection_name`; za druge `name`.
3. `update` se ne smije očekivati kao mehanizam javljanja: u sva tri je prazan.
4. Upravitelj nema vlastiti `__hash__`: vrijedi zadani (identitet), pa je upravitelj ispravan ključ rječnika.

## 08. Sequence Diagrams

### Normalan tok

Slijedi. Zatečeni dokument nema ovog odjeljka.

### Dijagram slijeda

| korak | ponašanje |
|---|---|
| 1 | **EV01** — ime se razrješava; ako je već registrirano, `warning` i povratak bez promjene |
| 2 | **EV02** — ime nije registrirano → `ManagerException`; inače se vraća objekt |
| 3 | **EV03** — ime nije registrirano → `ManagerException` s imenom akcije; inače se proslijedi `operation(action, **kwargs)` upravljanog objekta i vrati njegov rezultat |
| 4 | **EV04** — vidi §6; razlikuje se po upravitelju |
| 5 | **EV05** — upravljani objekti se raspreme, pogreške se skupe u jedan `error` zapis, rječnik se isprazni |


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title ConnectionManager, DriverManager, ProcessorManager

actor Caller
participant Manager
participant "Managed object" as Object

Caller -> Manager : register_*(object)
alt name already registered
  Manager --> Caller : warning, no change
else
  Manager -> Manager : registry[name] = object
end
Caller -> Manager : operation(name, action)
alt name not registered
  Manager --> Caller : ManagerException
else
  Manager -> Object : operation(action)
  Object --> Manager : result
  Manager --> Caller : result
end
Caller -> Manager : unregister_*(name)
Manager -> Object : release (disconnect or unload)
Manager -> Manager : remove the entry
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `ConnectionManager.disconnect` padne | iznimka se guta, `error`, vraća `False` |
| `ConnectionManager.register_connection` bez imena | `ManagerException` |
| `unregister_*` s nepoznatim imenom | `warning` (processor: bez poruke) |
| `unregister_connection` | unos nestaje; povezana konekcija se odspaja, a kvar odspajanja bilježi se (`error`) i unos je ipak uklonjen |
| `ConnectionManager.hot_swap` s nepoznatim imenom | `ManagerException` |
| `ConnectionManager.hot_swap`: nova konekcija se ne spoji (iznimka ili `False`) | `ManagerException`; stara ostaje registrirana i aktivna, promatrači se ne prebacuju |
| `ConnectionManager.hot_swap`: odspajanje stare padne | `warning`; zamjena je provedena |
| `unregister_processor` | izbacuje unos; procesor nema što osloboditi |
| `unregister_driver` | unos nestaje i driver se ispražnjuje (`ensure_unloaded`); kvar se bilježi (`error`), unos je ipak uklonjen |
| `ProcessorManager.operation` s akcijom osim `Start` | `warning`, vraća `False`, procesor se ne zove |
| pogreška pri raspremanju u `__del__` | skupi se, `error`; ostali objekti se i dalje raspremaju |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title ConnectionManager, DriverManager, ProcessorManager

start
:register_*(object, **kwargs);
:name from kwargs or from the object;
if (name missing?) then (yes)
  :<b><color:red>FAILED: ManagerException</color></b>;
  kill
endif
if (name already registered?) then (yes)
  :warning, no change;
else (no)
  :registry[name] = object;
endif
:operation(name, action, **kwargs);
if (name not registered?) then (yes)
  :<b><color:red>FAILED: ManagerException</color></b>;
  kill
endif
if (ProcessorManager and action is not Start?) then (yes)
  :warning, return False;
  stop
endif
:managed.operation(action);
:return result;
:unregister_*(name);
if (name registered?) then (yes)
  :remove the entry;
  :release the object (a failure is reported);
else (no)
  :warning;
endif
stop
@enduml
```

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | operacije su `DEBUG`; pogreške raspremanja `ERROR` |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | upravitelj ne poznaje protokol konekcije ni drivera, samo `operation` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 5 |

## 12. Results

Workflow dohvaća konekciju, driver ili procesor po imenu, a pri kraju života upravitelja nijedan
upravljani objekt ne ostaje otvoren zbog pogreške drugog.

## 13. Acceptance Criteria

1. Registracija istog imena ne prepisuje postojeći unos. ✅
2. Nepoznato ime u `get_*` i `operation` diže `ManagerException`. ✅
3. Pogreška pri raspremanju jednog objekta ne prekida raspremanje ostalih. ✅
4. Modul deklarira `__all__`. ✅
5. Svaka klasa deklarira `__slots__` (`BR-PTN-07`), uključujući `ProcessorManager`. ✅
6. `hash(upravitelj)` radi i vraća `int`. ✅
7. `unregister_*` vraća upravitelja u stanje bez registriranog objekta, a objekt oslobađa kao `__del__` (konekcija se odspaja, driver ispražnjuje). ✅
8. `__del__` ne diže iznimku ni kad konstruktor nije dovršen ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §4 t.4) i ne bilježi isti završetak dvaput. ✅
9. `ConnectionManager.hot_swap` ne gubi promatrače, a pri neuspjelom spajanju stara konekcija ostaje aktivna. ✅
10. Ime se razrješava bez čitanja svojstva koje ne treba: `connection_name` iz `kwargs` ima prednost. ✅
11. `update` prihvaća događaj u sva tri upravitelja i ne mijenja ništa. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–3 | `workflow/tests/test_manager.py` (`ConnectionManagerTest`, `DriverManagerTest`, `ProcessorManagerTest`) | bez prepisivanja, `ManagerException` za nepoznata imena, raspremanje svih unatoč kvaru jednog |
| 4 | pregled modula | `__all__` s tri imena |
| 5, 6, 11 | `CommonTest` | `__slots__` u sva tri; `hash` je `int` i upravitelj je ključ rječnika; `update(event, name=…)` ne diže |
| 7 | `ConnectionManagerTest`, `DriverManagerTest`, `ProcessorManagerTest` | odspajanje/ispražnjavanje pri `unregister_*`, kvar oslobađanja ne ostavlja unos; nepovezana konekcija se ne dira |
| 8 | `CommonTest`, `DriverManagerTest` | `sys.unraisablehook` ne dobiva ništa za neizgrađene upravitelje; `Delete/Completed` jednom |
| 9 | `workflow/tests/test_hot_swap.py` (7 testova) | prolaze; mutacije (bez prijenosa promatrača, bez upisa u registar, bez `release`) ruše testove |
| 10 | `ConnectionManagerTest` | objekt bez `connection_name` s imenom u `kwargs` se registrira; bez ikakvog imena `ManagerException` |
| mutacije | ručno | tipfeler `__slots_`, bez odspajanja i bez ispražnjavanja pri `unregister_*`, svojstvo čitano uvijek, nezaštićen `__del__` — svaka ruši test |

**Trojka (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-MGR-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: [`concrete/manager.py`](../../../workflow/src/wattleflow/concrete/manager.py)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; nepravilno numerirani odjeljci izvornog dokumenta (3, 6, 8.1) složeni u strukturu 01–17. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno u cjelini (`test_manager.py`, 32 testa, uz 7 u `test_hot_swap.py`), svi defekti zatvoreni: `DEF-MGR-01` `__slots__` u `ProcessorManager` (tipfeler); `-02` `hash(ConnectionManager)` je dizao `AttributeError` (`_random`), svi `__hash__` uklonjeni i vrijedi zadani; `-03` `unregister_driver` nije uklanjao driver (dvaput isti `update`, poruka o „konekciji") — sada uklanja i ispražnjuje; `-04` `unregister_*` oslobađa kao `__del__` (konekcija odspaja, driver ispražnjuje; procesor nema što osloboditi); `-05` `__del__` zaštićen od neizgrađenog objekta, a `DriverManager` ne bilježi završetak dvaput; `-06` `update` je namjerno prazan, ujednačen na `DEBUG` (`ProcessorManager.update(**kwargs)` nije primao događaj pa bi `update(event)` pao); `-07` tvrdnja o `ConnectionManagerException`/`DriverManagerException`/`ProcessorManagerException` u modulu **nije točna** (ne postoje) i uklonjena; `-08` `register_connection` je uvijek čitao `connection.connection_name` pa objekt bez svojstva nije mogao dobiti ime iz `kwargs` — ispravljeno; `pop` naspram `get` nad `kwargs` nema učinka (lokalna kopija). Kriteriji 5–8, 10–11, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Dodan `ConnectionManager.hot_swap` (dijagram, tok, alternativni tokovi, kriterij 9) i reakcija drivera na `Event.Swap`. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: uloge zamijenjene klasama (WorkflowFactory, GenericDriver, GenericConnection, GenericProcessor), tipovi svojstva all, paket manager, operation(name, action, **kwargs). |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — s defektima (odjeljak 15 t.1–4) |
