<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SGT — Singleton

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — `BR-PTN-01` (identitet izveden iz vlastitog tipa) |
| **Predmet** | `Singleton(IWattleflow)` — jedna instanca po konkretnoj podklasi, `__init__` se izvodi jednom |
| **Sestrinski** | [`FRQ-PTN`](FRQ-PTN-root-base.md) (korijen; `Singleton` ga ne nasljeđuje) |
| **Izvedba** | `workflow/src/wattleflow/concrete/singleton.py` (83 linije) |
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

Singleton je **temeljna klasa, ne primitiv**: podklasa koja ga nasljeđuje dobiva po jednu instancu po
procesu. Ne koristi metaklasu — zaštita je u `__new__` i u omotu `__init__` koji podklasi nameće
`__init_subclass__`. Omot drži bravu klase tijekom cijelog `__init__`, pa druga dretva čeka potpuno
izgrađen objekt, a `__init__` koji padne ponavlja se pri sljedećem pozivu. Nije `Wattleflow`: nema audita ni loggera; nasljeđuje samo `IWattleflow`
(`name`, `__str__`, `__repr__`).

| član | uloga |
|---|---|
| `_instances: dict` | **dijeljen na razini procesa**: `klasa → instanca` |
| `_lock` | zasebna brava po podklasi za izgradnju instance, stvara je `__init_subclass__` |
| `_init_lock` | zasebna `RLock` po podklasi, drži se tijekom cijelog `__init__` |
| `_init_calls: dict` | `klasa → (args, kwargs)` poziva koji je inicijalizirao instancu; služi za prijavu različitih argumenata |
| `_wf_initialized` | zastavica na instanci: `__init__` je već izveden; **slot deklarira sama baza** |

| pravilo | ponašanje |
|---|---|
| apstraktna klasa | nikad se ne sprema; svaki poziv gradi novi objekt |
| konkretna klasa | sprema se pri prvom pozivu, uz dvostruku provjeru pod bravom klase |
| `__init__` | omotava se jednom po podklasi; izvodi se pod `_init_lock` samo dok `_wf_initialized` nije postavljen; roditeljski `__init__` kroz `super()` dio je iste inicijalizacije, a zastavicu postavlja samo najvanjskiji poziv |
| ponovni poziv s drugim argumentima | vraća se ista instanca; `RuntimeWarning` kaže da se argumenti ne primjenjuju |
| `__init__` padne | iznimka prolazi; zastavica nije postavljena, pa sljedeći poziv ponovno izvodi cijeli `__init__` (instanca ostaje spremljena) |
| podklasa s `__slots__` | ne treba dodatni slot; ako ga i navede, ostaje ispravno |

<div align="center">


</div>

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Singleton

top to bottom direction

interface IWattleflow

class Singleton {
  - {static} _instances : dict
  - {static} _lock : Lock
  - {static} _init_lock : RLock
  - {static} _init_calls : dict
  # _wf_initialized : bool
  + name : str
  + __init_subclass__(**kwargs)
  + __new__(*args, **kwargs)
  # {static} _note_repeat(args, kwargs)
}

class Subclass

IWattleflow <|.. Singleton
Singleton <|-- Subclass

note right of Singleton
  not a Wattleflow:
  no audit, no logger
end note
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Singleton

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "Singleton", "One instance per concrete subclass")
System_Ext(pk, "Subclass", "class X(Singleton)")
System_Ext(pz, "Caller", "X(...)")
Rel_L(pk, sys, "Inherits Singleton")
Rel_R(pz, sys, "Obtains the unique instance")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | podklasa (`class X(Singleton)`) | vlastiti `__init__` i `__slots__` |
| **A2** | pozivatelj `X(...)` | argumenti konstruktora (vrijede samo pri prvom pozivu) |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | definicija podklase → `__init_subclass__` |
| **EV02** | A2 zove `X(...)` → `__new__`, zatim (omotani) `__init__` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Singleton

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Subclass" as A1
  actor "Caller" as A2
    usecase "Define subclass" as EV01
    usecase "Obtain unique instance" as EV02
  A1 --> EV01
  A2 --> EV02
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Podklasa s `__slots__` ne mora uključiti `_wf_initialized`: slot deklarira baza.
2. Argumenti konstruktora vrijede samo za prvi (uspješan) poziv; kasniji različiti argumenti zanemaruju se uz `RuntimeWarning`.
3. `__init__` je definiran u samoj podklasi da bi se omotao; naslijeđeni `__init__` već je omotan u roditelju.

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | **EV01** — podklasa dobiva vlastitu bravu; ako ima vlastiti `__init__`, zamjenjuje se omotom |
| 2 | **EV02** — `__new__`: apstraktna klasa → običan objekt; inače, ako klase nema u `_instances`, pod bravom (dvostruka provjera) gradi se i sprema instanca |
| 3 | `__new__` vraća spremljenu instancu |
| 4 | omotani `__init__`: ako je `_wf_initialized` postavljen, vraća se; inače se izvodi izvorni `__init__` i zastavica se postavlja |



### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Singleton

actor Caller
participant "X (Singleton)" as X
participant "Singleton" as Registry

Caller -> X : X(...)
activate X
X -> Registry : instance of X?
alt first call
  X -> X : create the instance under the class lock
  X -> Registry : _instances[X] = instance
  X -> X : ~__init~__ under the init lock
  X -> X : _wf_initialized = True
else later call
  Registry --> X : existing instance
  X -> X : skip ~__init~__, warn if the arguments differ
end
X --> Caller : the same instance
deactivate X
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| drugi poziv s drugim argumentima | vraća se ista instanca; `RuntimeWarning` (argumenti jednaki prvom pozivu ili bez argumenata ne prijavljuju se; argumenti koji se ne mogu usporediti prijavljuju se) |
| istodobni prvi pozivi iz više dretvi | jedna instanca, `__init__` jednom; ostale dretve čekaju potpuno izgrađen objekt |
| `__init__` digne iznimku | instanca ostaje spremljena, zastavica nije postavljena; sljedeći poziv ponovno izvodi `__init__` (i roditeljske kroz `super()`) |
| podklasa podklase | zasebna instanca i zasebne brave (ključ je klasa) |
| podklasa bez vlastitog `__init__` | omot se ne stvara; vrijedi omot roditelja |
| singleton gradi drugi singleton u svom `__init__` | svaki se inicijalizira jednom (praćenje je po instanci, ne po dretvi) |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Singleton

start
:X(...);
if (class is abstract?) then (yes)
  :new plain object, not cached;
  stop
endif
if (class not yet in _instances?) then (yes)
  :acquire the class lock;
  if (class still not in _instances?) then (yes)
    :create instance, _instances[class] = instance;
  endif
endif
:acquire the init lock of the class;
if (_wf_initialized?) then (yes)
  if (arguments differ from the first call?) then (yes)
    :RuntimeWarning, arguments are not applied;
  endif
  :return the instance;
  stop
endif
:run ~__init~__ (parents through super() run inside it);
if (~__init~__ raised?) then (yes)
  :<b><color:red>FAILED: exception, flag not set, next call retries</color></b>;
  kill
endif
:_wf_initialized = True, remember the arguments;
:return the instance;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | `_instances` je procesno globalno stanje (ambient authority), kako to navodi i docstring |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | uvozi samo standardnu knjižnicu i `wattleflow.core` |

## 12. Results

Za svaku konkretnu podklasu postoji najviše jedna instanca, a njezino stanje se postavlja jednom.

## 13. Acceptance Criteria

1. Dva poziva iste konkretne podklase vraćaju isti objekt. ✅
2. Apstraktna klasa se ne sprema. ✅
3. Svaka podklasa ima vlastitu bravu i vlastitu instancu. ✅
4. Modul deklarira `__all__`. ✅
5. `__init__` se izvodi najviše jednom i uz istodobne pozive; druga dretva ne vidi napola izgrađen objekt. ✅
6. Argumenti kasnijih poziva se ne gube nečujno: različiti argumenti daju `RuntimeWarning`. ✅
7. Neuspjeli `__init__` ostavlja instancu koja se ponovno inicijalizira pri sljedećem pozivu, uključujući lanac roditeljskih `__init__`. ✅
8. Zastavica `_wf_initialized` je slot baze, pa podklasa s `__slots__` radi bez ceremonije. ✅
9. `_instances` je procesno promjenjivo stanje (zapisano svojstvo, [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md)): ključ je klasa, nijedan javni put ga ne mijenja. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–3 | `workflow/tests/test_singleton.py` (`IdentityTest`) | isti objekt; zasebna instanca i brava po podklasi; podklasa podklase zasebna; apstraktna se ne sprema, konkretna iz apstraktne da |
| 4 | pregled modula | `__all__ = ["Singleton"]` |
| 5 | `InitOnceTest` | 8 dretvi na barijeri, spor `__init__`: jedno izvođenje i sve dretve vide `ready`; lanac `__init__` jednom; singleton u singletonu jednom |
| 6 | `ArgumentsTest` | različiti argumenti → `RuntimeWarning`; isti ili nikakvi → ništa; neusporedivi argumenti ne ruše poziv |
| 7 | `FailureTest` | pad se širi i ponavlja; pad djeteta nakon roditeljskog `__init__` ponavlja cijeli lanac |
| 8 | `SlotsTest` | podklasa s `__slots__` radi bez dodatnog slota i s njim |
| 9 | pregled | `_instances` je atribut klase |
| mutacije | ručno | bez brave inicijalizacije, bez prijave argumenata, bez praćenja ugniježđenog poziva, praćenje se ne briše, slot nije deklariran — svaka ruši test |

**Trojka (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-SGT-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/singleton.py`
- Testovi: `workflow/tests/test_singleton.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka iz odjeljka 08 uklonjen (nosi ga odjeljak 09). |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_singleton.py`, 18 testova), svi defekti zatvoreni: `DEF-SGT-01` `__init__` je sada pod bravom klase (`_init_lock`) — (test s 8 dretvama i sporim `__init__` padao je prije ispravka: bez brave provjera zastavice i izvođenje nisu atomični); `-02` različiti argumenti kasnijih poziva daju `RuntimeWarning`; `-03` `_instances` zapisan kao svojstvo (procesno stanje, ključ je klasa); `-04` pad `__init__` zapisan kao ponavljanje cijelog lanca, i ispravljeno: roditeljski omot je postavljao zastavicu usred djetetova `__init__`, pa se pad djeteta nije ponavljao; `-05` zastavica `_wf_initialized` je slot baze (`Singleton.__slots__`); tvrdnja da je slot obvezan u podklasi nije vrijedila jer instance nose `__dict__` iz `IWattleflow`. **Novo:** singleton koji gradi drugi singleton u svom `__init__` ne smije se pogrešno tretirati kao ugniježđeni poziv istog objekta — praćenje je po instanci. Kriteriji 5–9, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: potpis `__init_subclass__(**kwargs)`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — s ograničenjima (odjeljak 15 t.1–4) |
