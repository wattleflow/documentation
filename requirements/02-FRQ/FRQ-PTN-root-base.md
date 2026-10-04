<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-PTN — Korijenska baza `Wattleflow`

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | t.3 — kategorija `PTN` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — `BR-PTN-01`, `BR-PTN-02`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | [`Wattleflow(Audit, IWattleflow)`](../../../workflow/src/wattleflow/concrete/base.py) — kanonski korijen svakog objekta frameworka |
| **Sestrinski** | svi zapisi `FRQ-*-15.*` — svaki od njih nasljeđuje ovu klasu prvu |
| **Izvedba** | `workflow/src/wattleflow/concrete/base.py` (69 linija) |
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

Klasa ima **jednu naredbu u tijelu** i najveći domet u sloju. Njezino tijelo je
`super().__init__(**kwargs)`; sve ostalo je ono što njezino postojanje **jamči**:

1. **Identitet** se izvodi iz konkretnog tipa pri pristupu, pa je nepromjenjiv: nijedan put u kodu
   ne može objekt preimenovati nakon konstrukcije. Time audit zapisi temeljeni na `__str__`
   ostaju otporni na krivotvorenje (`BR-PTN-01`).
2. **Auditabilnost nije opcija.** `Audit` se nasljeđuje **ovdje**, na jednom deklariranom mjestu,
   pa je svaki potomak nosi **po ugovoru** — a ne tako što je usput pokupi kao nuspojavu neke
   druge baze (`BR-PTN-02`).
3. **Podjela `**kwargs` je na jednom mjestu.** `Wattleflow.__init__` samo otvara kooperativni lanac
   (`super().__init__(**kwargs)`); logging ključeve (`formatting`, `propagate`, `logger`, `level`,
   `handler`; uz njih alias `fmt` za `formatting`) izdvaja jedino `Audit.__init__`, a ostatak ključeva ondje stane. Podklasa prosljeđuje
   **cijeli** `**kwargs` nepromijenjen i ostatak zadržava za sebe.

Treća točka je razlog zašto klasa uopće postoji kao zaseban sloj iznad `Audit`: podklasa nasljeđuje jedan korijen, a ne `Audit` i sučelje zasebno. Podklasa koja bi
sama izdvajala logging ključeve duplicira podjelu — i razilazi se **istog trena** kad se doda novi
logging ključ, jer njezina kopija popisa ostaje stara.


>>NOTE:
    Wattleflow - canonical root of every framework object: identity plus audit.

    Identity is derived on access from the concrete type and is therefore
    immutable: no code path can rename an object after construction, which
    keeps __str__-based audit records forgery-resistant.

    Auditability is not optional. Audit is inherited here, at the single
    declared point, so every descendant carries it by contract rather than by
    picking it up as a side effect of some other base.

    Concrete framework bases (Generic*, managers, ...) inherit this root first,
    ahead of their pattern interfaces, and never name Audit themselves:
        class GenericProcessor(Wattleflow, IProcessor[Item], ABC): ...

    __init__ opens the cooperative chain and is the single place that splits a
    constructor's keywords: the logging ones go to Audit, the rest stop
    here. Subclasses therefore forward their whole **kwargs unchanged and keep
    the remainder for their own use (PresetDecorator, strategies, ...):

        def __init__(self, **kwargs):
            super().__init__(**kwargs)
            self._preset = PresetDecorator(self, **kwargs)

    No subclass pops the logging keywords itself — doing so duplicates the
    split and drifts the moment a logging keyword is added.

    Interface (inherited from Audit, which owns identity):
        name -> str      (concrete type name)
        __str__ -> str   (name)
        __repr__ -> str  (TypeName())

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Wattleflow

top to bottom direction

  interface IWattleflow
  interface ILogger
  interface IObserver
  class Audit {
    +name : str
    +levelname : str
    +debug(msg : str, *args, **kwargs) : None
    +info(msg : str, *args, **kwargs) : None
    +warning(msg : str, *args, **kwargs) : None
    +error(msg : str, *args, **kwargs) : None
    +set_level(level : int | str) : None
  }
  class Wattleflow {
    __slots__ = ()
  }
  class GenericProcessor
  class PresetDecorator
ILogger <|.down. Audit
IObserver <|.down. Audit
Audit <|-down- Wattleflow
IWattleflow <|.down. Wattleflow
Wattleflow <|-down- GenericProcessor
GenericProcessor *-down- PresetDecorator : _preset
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Wattleflow

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "Wattleflow", "Root base of the generic classes")
System_Ext(gk, "GenericProcessor", "Descendant (any generic class)")
System_Ext(au, "Audit", "Identity and logging")
System_Ext(pd, "PresetDecorator", "Configuration")
Rel_L(gk, sys, "Is built through super().~__init~__")
Rel_R(sys, au, "Provides name and logging")
Rel_U(sys, pd, "Takes over the remaining keys")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Bilo koja generička klasa sloja | `**kwargs` iz tvornice |
| **A2** | `Audit` — vlasnik identiteta i zapisa | logging ključevi |
| **A3** | `PresetDecorator` — u podklasi, nad ostatkom ključeva | konfiguracijski ključevi |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | podklasa se konstruira → `super().__init__(**kwargs)` |
| **EV02** | bilo koji poziv `debug` / `info` / `warning` / `error` |
| **EV03** | objekt ulazi u zapis → `__str__` / `__repr__` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Wattleflow

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "GenericProcessor" as A1
  actor "Audit" as A2
  actor "PresetDecorator" as A3
    usecase "Construct subclass" as EV01
    usecase "Record log entry" as EV02
    usecase "Render object in log" as EV03
  A1 --> EV01
  A3 --> EV01
  A1 --> EV02
  A2 --> EV02
  A2 --> EV03
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Podklasa navodi `Wattleflow` **prvi** u popisu baza, ispred svojeg pattern sučelja.
2. Podklasa **ne navodi** `Audit` među bazama.
3. Podklasa ne izdvaja logging ključeve iz `**kwargs` (ni alias `fmt`: razrješava ga `Audit`).
4. Podklasa u sloju `concrete` (i `schedulers`, `decorators`) deklarira vlastiti `__slots__` i ne ponavlja slotove baza (`BR-PTN-07`); prazan `()` je ispravna izjava da klasa ne dodaje stanje.
5. Opseg: klase u `helpers` (`Config`, `Monitor`, `ClassLoader`, `ResourceManager`) leže **ispod** korijena (korijen uvozi `helpers.audit`), pa imenuju `Audit` izravno i ovim pravilima nisu vezane.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — podklasa poziva `super().__init__(**kwargs)`; lanac vodi kroz `Wattleflow` u `Audit`,
   koji uzme logging ključeve.
2. Podklasa zatim gradi vlastito stanje iz preostalih ključeva, tipično kroz `PresetDecorator`.
3. **EV02/EV03** — identitet i zapis dolaze iz `Audit`: `name` (ime konkretnog tipa), `__str__`
   (ime), `__repr__` (`TypeName()`).

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Wattleflow

  participant "GenericProcessor" as S
  participant "Wattleflow" as W
  participant "Audit" as A
  participant "PresetDecorator" as P
  activate S
  S -> W : super().~__init~__(**kwargs)
  activate W
  W -> A : ~__init~__(logging keys)
  activate A
  A --> W
  deactivate A
  W --> S
  deactivate W
  S -> P : PresetDecorator(self, **remainder)
  activate P
  deactivate P
  S -> A : debug(msg=Constructor, ...)
  activate A
  deactivate A
  deactivate S
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| podklasa navede `Wattleflow` iza `Generic[…]` | MRO se ne linearizira kad se u miks doda `IOriginator` | konstrukcija pada pri definiciji klase ([`FRQ-BBD`](FRQ-BBD-blackboard.md) §8 k.1) |
| podklasa sama navede `Audit` kao bazu | audit stiže dvaput u MRO | prekršaj  nije mjereno lintom |
| podklasa sama pop-a logging ključ | podjela je duplicirana | razilazi se pri sljedećem novom ključu |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Wattleflow

start
:subclass: super().~__init~__(**kwargs);
if (MRO linearises?) then (no)
:TypeError at class definition;
stop
else (yes)
endif
:Audit takes the logging keys;
if (subclass names Audit itself or extracts a logging key?) then (yes)
:audit twice in MRO or duplicated split;
note right: checked by tests/test_wattleflow_base.py
else (no)
endif
:subclass builds state (PresetDecorator);
:name, ~__str~__, ~__repr~__ from Audit;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | redakcija je odgovornost `Audit` obitelji; podklasa je ne može zaobići jer je ne nasljeđuje zaobilazno |
| `NFRQ-OBS-01/02/03` | jedno mjesto nasljeđivanja znači da lint mjeri zapis nad **svim** objektima, ne nad uzorkom |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | korijen uvozi samo `wattleflow.core` i `wattleflow.helpers.audit` — clean core |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | podjela `**kwargs` postoji jednom, u `Audit`; korijen je jamči tako što je jedini put do nje |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Svaki objekt frameworka ima ime koje se ne može promijeniti i zapis koji se ne može izostaviti, a
konfiguracijski ključevi stižu tamo gdje pripadaju bez ijedne kopije popisa u podklasama.

## 13. Acceptance Criteria

1. Klasa nasljeđuje `Audit` i `IWattleflow`, tim redom. ✅
2. `__slots__ = ()` — korijen ne dodaje stanje. ✅
3. Tijelo `__init__` je isključivo `super().__init__(**kwargs)`. ✅
4. Modul deklarira `__all__`; import closure je `wattleflow.*`. ✅
5. Docstring imenuje ugovor koji klasa jamči. ✅
6. Nijedna podklasa u sloju ne navodi `Audit` kao bazu niti izdvaja logging ključeve. ✅ — alias `fmt` preseljen u `Audit`
7. Nijedna podklasa u sloju ne ponavlja slotove baze. ✅
8. Svaka klasa sloja deklarira vlastiti `__slots__` (`BR-PTN-07`). ✅
9. `Wattleflow` stoji prvi među bazama, ispred svakog drugog pattern sučelja. ✅
10. Identitet je nepromjenjiv: `name` se ne može dodijeliti, a `__str__`/`__repr__` ne može nadjačati instanca. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–3 | pregled `base.py`; `workflow/tests/test_wattleflow_base.py` (`ContractTest`) | `class Wattleflow(Audit, IWattleflow)`, `__slots__ = ()`, jedna naredba u `__init__`; `Audit` prije sučelja u MRO |
| 4–5 | pregled modula, docstringa | `__all__ = ["Wattleflow"]`; ugovor `name` / `__str__` / `__repr__` naveden izrijekom |
| 6–9 | `StructureTest` (AST-ni obilazak svih klasa u `concrete`, `schedulers`, `decorators`, više od 30) | nijedna ne navodi `Audit`; korijen prvi; svaka ima vlastiti `__slots__`; nijedna ne ponavlja slot baze; prije ispravka: 6 klasa bez slotova, 7 s ponovljenim slotovima |
| 10, 3 (podjela) | `IdentityTest`, `KeywordSplitTest` | `name = …` diže `AttributeError`; instanca ne nadjačava `str`; `fmt` postaje `formatting`, a izričiti `formatting` ima prednost; ostatak ključeva ne stiže do `object.__init__` |
| mutacije | ručno | ponovljen slot, klasa bez slotova, bez aliasa `fmt` — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest` (AST/MRO obilazak); kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-PTN-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/base.py`; podjela ključeva `workflow/src/wattleflow/helpers/audit.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; `__init__` u tekstu dijagrama zaštićen od podcrtavanja. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_wattleflow_base.py`, 16 testova), svi defekti zatvoreni: `DEF-PTN-01` ponovljeni slotovi uklonjeni (`GenericWorkflow._level/_handler`, `WorkflowFactoryLogger._logger`, `_strict` u četiri razreda strategija); `-02` pravilo „korijen prvi” i zabrana `Audit` kao baze sada provjerava AST-ni test (a ne samo pad MRO-a); `-03` `BR-PTN-07`: `__slots__` dodan u `DummyReadDocument`, `RepositoryWithDriver`, `SchedulerCronJob` (`ProcessorManager` i `Scheduler._preset` ispravljeni ranije), a tvrdnje o `WorkflowFactoryLogger` i `DriverNotFound` bile su zastarjele; `-04` alias `fmt` preseljen iz `GenericBlackboard` u `Audit` (i u popis ključeva koje preset ne prijavljuje). Opseg zapisan: `helpers` leži ispod korijena i imenuje `Audit` izravno. Kriteriji 6–10, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: uloga „Generic class/Subclass” zamijenjena klasom GenericProcessor, potpisi Audit (`*args`, povratni tipovi), kutije paketa. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — obrnuti inženjering zatečenog |
