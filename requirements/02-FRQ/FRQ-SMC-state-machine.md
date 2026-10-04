<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SMC — Automat stanja

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — `BR-PTN-03` (prijelaz koji tablica ne dopušta ne izvodi se), `BR-PRC-01`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `StateMachine(IStateMachine, Generic[State, Action], ABC)` i `GuardedStateMachine(IStateMachine, ABC)` — tablica prijelaza, trenutno stanje i jednokratna zaštita prije prvog prijelaza |
| **Sestrinski** | [`FRQ-CON`](FRQ-CON-connection.md) · [`FRQ-DRV`](FRQ-DRV-driver.md) · [`FRQ-PRC`](FRQ-PRC-processor.md) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) (drže ili izvoze tablicu prijelaza) |
| **Izvedba** | `workflow/src/wattleflow/concrete/state_machine.py` (123 linije) |
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

Automat je **najmanja** potpora životnom ciklusu: ne zna što su stanja ni što akcije znače. Prima
tablicu `(stanje, akcija) → stanje` i početno stanje, i dopušta isključivo prijelaze iz tablice.
Nije `Wattleflow`: nema audita, loggera ni preseta (laka razina). Vlasnik automata (konekcija,
driver, procesor, specijalizacija blackboarda) bilježi što se događa.

| član | uloga |
|---|---|
| `_transitions` | preslikavanje `(stanje, akcija) → stanje`; **samo za čitanje kopija** (`MappingProxyType`) tablice koju vlasnik preda: kasnija izmjena vlasnikova rječnika ne mijenja automat |
| `_state` | trenutno stanje; mijenja se samo kroz `apply` |
| `_name` | ime za zapise; zadano ime klase |
| `_lock` | provjera i upis stanja su jedan korak; dvije dretve ne mogu obje uzeti isti prijelaz |

| metoda | ponašanje |
|---|---|
| `can(action)` | `True` ako tablica sadrži `(stanje, akcija)`; ne mijenja stanje |
| `apply(action)` | ako ključa nema, `ValueError`; inače stanje postaje vrijednost iz tablice (atomično) |
| `try_apply(action)` | `can` i `apply` kao jedan atomični korak: `True` ako je prijelaz uzet, `False` ako nije dopušten |
| `state` | svojstvo samo za čitanje |

`GuardedStateMachine` omata automat: pri **prvom** `apply` (ili `try_apply`) zove `guard(inner)` pod vlastitom bravom, pa i kad istodobno stigne više dretvi zaštita se izvede jednom; ako zaštita digne
iznimku, prijelaz se ne događa, a zaštita se ponavlja pri sljedećem pozivu; nakon uspjeha delegira bez daljnje provjere. `can`, `state` i `inner`
samo proslijeđuju.

<div align="center">


</div>

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title StateMachine, GuardedStateMachine

top to bottom direction

interface IStateMachine {
  + can(action) : bool
  + apply(action)
}

class "StateMachine<State, Action>" as StateMachine {
  # _transitions : Mapping[tuple[State, Action], State]
  # _state : State
  # _name : str | None
  # _lock : Lock
  + name : str
  + state : State
  + can(action) : bool
  + apply(action)
  + try_apply(action) : bool
}

class "GuardedStateMachine<State, Action>" as GuardedStateMachine {
  # _inner : StateMachine
  # _guard : Callable
  # _consumed : bool
  # _guard_lock : Lock
  + name : str
  + inner : StateMachine
  + state : State
  + can(action) : bool
  + apply(action)
  + try_apply(action) : bool
}

IStateMachine <|.. StateMachine
IStateMachine <|.. GuardedStateMachine
GuardedStateMachine "1" o-- "1" StateMachine : wraps
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title StateMachine, GuardedStateMachine

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "StateMachine, GuardedStateMachine", "Transition table and current state")
System_Ext(vl, "GenericConnection, GenericDriver, GenericProcessor", "Owners of the state machine")
System_Ext(gu, "Guard", "Function over the state machine")
Rel_L(vl, sys, "Builds the state machine; checks and applies actions")
Rel_R(sys, gu, "Runs it before the first transition")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | vlasnik automata (`GenericConnection`, `GenericDriver`, `GenericProcessor`, specijalizacije blackboarda) | tablica prijelaza, početno stanje, ime |
| **A2** | tablica prijelaza | `(stanje, akcija) → stanje` |
| **A3** | zaštita (`guard`) | funkcija nad automatom, jednokratna |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 gradi automat → `StateMachine(transitions, initial, name)` |
| **EV02** | A1 provjerava → `can(action)` |
| **EV03** | A1 mijenja stanje → `apply(action)` |
| **EV04** | A1 omata automat zaštitom → `GuardedStateMachine(inner, guard)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title StateMachine, GuardedStateMachine

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "GenericConnection,\nGenericDriver,\nGenericProcessor" as A1
  actor "Guard" as A3
    usecase "Build state machine" as EV01
    usecase "Check action (can)" as EV02
    usecase "Apply action (apply)" as EV03
    usecase "Wrap with guard" as EV04
  A1 --> EV01
  A1 --> EV02
  A1 --> EV03
  A1 --> EV04
  A3 --> EV04
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Stanja i akcije su vrijednosti iste vrste kao ključevi tablice (obično `Enum`). Početno stanje mora se pojaviti u tablici (kao izvor ili odredište): inače `ValueError` pri izgradnji. Vrsta ključeva se dalje ne provjerava: neodgovarajući ključ je samo „akcija nije dopuštena”.
2. Automat drži vlastitu kopiju tablice; vlasnikova izmjena kasnije ne vrijedi.
3. Vlasnik koji bi poziv `can` pa `apply` izveo iz više dretvi koristi `try_apply`; inače je spreman na `ValueError`.

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | **EV01** — `_state = initial`, `_transitions` je predana tablica |
| 2 | **EV02** — `(stanje, akcija) in tablica` |
| 3 | **EV03** — ako je ključ u tablici, `_state = tablica[ključ]`; inače `ValueError("<akcija> not allowed in state <stanje>")` |
| 4 | **EV04** — `apply` pri prvom pozivu zove `guard(inner)`, označi zaštitu potrošenom, pa `inner.apply` |

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title StateMachine, GuardedStateMachine

participant "Owner" as Owner
participant GuardedStateMachine as Guarded
participant Guard
participant StateMachine as Machine

Owner -> Guarded : apply(action)
activate Guarded
alt guard not consumed
  Guarded -> Guard : guard(inner)
  alt guard raises
    Guard --> Guarded : exception
    Guarded --> Owner : exception, no transition
  else
    Guard --> Guarded : done
    Guarded -> Guarded : _consumed = True
  end
end
Guarded -> Machine : apply(action)
activate Machine
alt (state, action) not in the table
  Machine --> Owner : ValueError
else
  Machine -> Machine : _state = table[(state, action)]
end
deactivate Machine
deactivate Guarded
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| akcija nije dopuštena u stanju | `can` je `False`; `apply` diže `ValueError`; stanje ostaje |
| zaštita padne | iznimka prolazi; `_consumed` ostaje `False`, pa se zaštita ponavlja pri sljedećem pozivu (namjerno: zaštita je uvjet prvog prijelaza, a pad nije odobrenje) |
| zaštita istodobno iz više dretvi | izvodi se jednom; ostale dretve čekaju |
| dvije dretve istim prijelazom iz istog stanja | jedna uspije, druga dobije `ValueError` (`apply`) odnosno `False` (`try_apply`) |
| prijelaz u isto stanje | dopušten ako je u tablici |
| pristup `state` | samo čitanje; dodjela `automat.state = …` diže `AttributeError` |
| vraćanje stanja | nema `reset` ni settera; oporavak je akcija `LOAD` iz tablice |
| vlasnik izravno čita `_state` | moguće, ali izvan ugovora |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title StateMachine, GuardedStateMachine

start
:apply(action);
if (guarded and guard not consumed?) then (yes)
  :take the guard lock;
  if (still not consumed?) then (yes)
    :guard(inner);
    if (guard raised?) then (yes)
      :<b><color:red>FAILED: exception, no transition, guard tried again next time</color></b>;
      kill
    endif
    :_consumed = True;
  endif
endif
:take the machine lock;
if ((state, action) in the table?) then (yes)
  :_state = table[(state, action)];
else (no)
  :<b><color:red>FAILED: ValueError, state unchanged</color></b>;
  kill
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
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | uvozi samo standardnu knjižnicu i `wattleflow.core` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 5 |

## 12. Results

Stanje vlasnika mijenja se isključivo primjenom akcije nad tablicom (`BR-PTN-03`); nedopušten
prijelaz nikad ne ostavlja automat u neodređenom stanju, jer se stanje dodjeljuje tek nakon provjere.

## 13. Acceptance Criteria

1. `apply` mijenja stanje samo za ključ iz tablice. ✅
2. Nedopuštena akcija diže `ValueError`; stanje se ne mijenja. ✅
3. `can` ne mijenja stanje. ✅
4. Zaštita se izvodi najviše jednom uspješno, prije prvog prijelaza, i njezin pad sprječava prijelaz. ✅
5. Modul deklarira `__all__`; obje klase deklariraju `__slots__` (`BR-PTN-07`). ✅ — učinak ovisi o `__slots__` u `IStateMachine` ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md))
6. Tablica je zaštićena od vanjske izmjene: automat drži vlastitu kopiju samo za čitanje. ✅
7. Vlasnik vraća automat u zadano stanje akcijom iz tablice (`LOAD`); nema posebne operacije vraćanja. ✅
8. Provjera i upis prijelaza su atomični: dvije dretve ne uzimaju isti prijelaz; `try_apply` je atomičan par `can` i `apply`. ✅
9. Početno stanje koje ne pripada tablici odbija se pri izgradnji. ✅
10. Zaštita se izvodi najviše jednom uspješno i pod istodobnim pozivima; pad zaštite se ponavlja. ✅
11. Obje klase su generičke (`StateMachine[State, Action]`, `GuardedStateMachine[State, Action]`). ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–3 | `workflow/tests/test_state_machine.py` (`TransitionTest`) | prijelaz po tablici, ista-stanje-prijelaz, nedopuštena akcija `ValueError` i stanje ostaje, `can` ne mijenja, `state` samo za čitanje, ime i `repr` |
| 4 | `GuardTest` | zaštita jednom prije prvog prijelaza; pad blokira prijelaz i ponavlja se; jednom i pod 8 dretvi; `can`, `state`, `inner`, ime prosljeđuju |
| 5 | `GuardTest` | `__slots__` u obje klase; `__all__` u modulu |
| 6, 9 | `TableTest` | izmjena izvornog rječnika ne dira automat; upis u vlastitu tablicu daje `TypeError`; početno stanje izvan tablice odbijeno, odredište-samo je valjano |
| 7 | `TransitionTest` | `try_apply` bez settera; oporavak akcijom iz tablice (primjeri vlasnika: `FRQ-PRC`, `FRQ-DRV`, `FRQ-CON`) |
| 8 | `ThreadsTest` | 8 dretvi, tablica sa sporom provjerom: točno jedna uspije; isto za `try_apply` |
| 11 | `GuardTest` | `StateMachine[S, A]` i `GuardedStateMachine[S, A]` se mogu parametrizirati |
| mutacije | ručno | tablica po referenci, bez provjere početnog stanja, `apply` bez brave, zaštita bez brave, zaštita potrošena prije uspjeha — svaka ruši test |

**Trojka (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-SMC-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/state_machine.py`
- Testovi: `workflow/tests/test_state_machine.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram stanja nije potreban (automati su u dokumentima vlasnika). |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_state_machine.py`, 21 test), svi defekti zatvoreni: `DEF-SMC-01` tablica se kopira kao `MappingProxyType`; `-02` nema vraćanja stanja zapisano kao svojstvo (oporavak je `LOAD`); `-03` `apply` je atomičan (`_lock`) i dodan `try_apply` kao atomični par `can`/`apply` — (test s 8 dretvi i sporom provjerom padao je prije ispravka: provjera i upis nisu bili jedan korak); `-04` početno stanje mora pripadati tablici; `-05` `GuardedStateMachine` je generička; `-06` ponavljanje zaštite nakon pada zapisano kao namjera. **Novo:** zaštita se mogla izvesti više puta pod istodobnim pozivima — sada pod vlastitom bravom. Pozivatelji `can`+`apply` (procesor, driver, blackboard) nisu mijenjani; `try_apply` je dostupan za njih. Kriteriji 6–11, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa izvan distribucije `workflow`; uloge iz drugih projekata zamijenjene općim nazivom. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: uloge Owner i Guard zamijenjene klasama GenericConnection, GenericDriver, GenericProcessor i OSCALGate. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |
