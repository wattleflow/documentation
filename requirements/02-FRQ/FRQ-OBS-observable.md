<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-OBS — Promatrani objekt (nit-siguran)

> **Oznaka `OBS` je na osi FRQ-a po dokumentiranoj odluci.** Ista oznaka na NFRQ osi znači *observability*
> (`NFRQ-OBS-01…04`); nazivi datoteka se razlikuju prefiksom (`FRQ-` / `NFRQ-`), a dvoznačnost u
> prozi razrješava kontekst (D-12).

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `OBS` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-02`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `ThreadSafeObservable(Wattleflow, IObservableReactive)` — popis promatrača uz sigurno javljanje iz više dretvi |
| **Sestrinski** | [`FRQ-CON`](FRQ-CON-connection.md) (`ConnectionObserverInterface` je zaseban promatrani objekt konekcije) · [`FRQ-MGR`](FRQ-MGR-managers.md) (`IObserver`, ne `IObserverReactive`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/observable.py` (+ testovi `workflow/tests/test_observable.py`) |
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

Promatrani objekt drži popis promatrača i na `notify_observers` zove `update(self, *args, **kwargs)`
svakog. Sigurnost među dretvama postignuta je kopiranjem popisa pod bravom i **javljanjem izvan
brave**: promatrač koji se u `update` odjavi, ili javlja drugome, ne zaglavi objekt. Kvar jednog
promatrača bilježi se i ne prekida ostale.

| član | uloga |
|---|---|
| `_observers: list` | promatrači po redoslijedu dodavanja, bez duplikata |
| `_observers_lock: RLock` | štiti izmjenu popisa i izradu kopije; **nije** `_lock` jer slot tog imena zasjenjuje `Audit._lock`, koji `Audit.__init__` uzima prije nego podrazred išta dodijeli |

<div align="center">


</div>

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title ThreadSafeObservable

top to bottom direction

  interface IObservableReactive {
    {abstract} +add_observer(observer)
    {abstract} +remove_observer(observer)
    {abstract} +notify_observers(*args, **kwargs)
  }
  interface IObserverReactive {
    {abstract} +update(observable, *args, **kwargs)
  }
  class Wattleflow
  class ThreadSafeObservable {
    -_observers : list[IObserverReactive]
    -_observers_lock : RLock
    +add_observer(observer)
    +remove_observer(observer)
    +notify_observers(*args, **kwargs)
  }
Wattleflow <|-down- ThreadSafeObservable
IObservableReactive <|.down. ThreadSafeObservable
ThreadSafeObservable o-- "0..*" IObserverReactive
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title ThreadSafeObservable

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "ThreadSafeObservable", "Keeps observers and notifies them thread-safely")
System_Ext(pt, "Subscriber", "Code that subscribes")
System_Ext(pe, "Event producer", "Specialisation or owner")
System_Ext(ob, "IObserverReactive", "Observer")
Rel_L(pt, sys, "Subscribes and unsubscribes observer")
Rel_R(pe, sys, "Raises event")
Rel_U(sys, ob, "Calls update(observable, …)")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | pretplatnik (kod koji se pretplaćuje) | promatrač |
| **A2** | `IObserverReactive` | `update(subject, *args, **kwargs)` |
| **A3** | proizvođač događaja (specijalizacija ili vlasnik) | `notify_observers(...)` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 se pretplaćuje → `add_observer` |
| **EV02** | A1 se odjavljuje → `remove_observer` |
| **EV03** | A3 javlja → `notify_observers(*args, **kwargs)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title ThreadSafeObservable

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Subscriber" as A1
  actor "IObserverReactive" as A2
  actor "Event producer" as A3
    usecase "Subscribe observer" as EV01
    usecase "Unsubscribe observer" as EV02
    usecase "Notify observers" as EV03
  A1 --> EV01
  A1 --> EV02
  A2 --> EV03
  A3 --> EV03
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Promatrač je instanca `IObserverReactive`; `add_observer` to provjerava (`AttributeException`).
2. Promatrač se pretplaćuje kao objekt: jednakost je identitet (`is`), pa su dva jednaka, ali različita promatrača dva promatrača.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — pod bravom: promatrač se dodaje ako ga nema.
2. **EV02** — pod bravom: promatrač se uklanja ako ga ima; nepostojeći se tiho preskače.
3. **EV03** — pod bravom nastaje kopija popisa; izvan brave se redom zove `update(self, *args, **kwargs)`.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title ThreadSafeObservable

  participant "A3 Event producer" as P
  participant "A2 IObserverReactive" as O
  participant "ThreadSafeObservable" as T
  P -> T : notify_observers(*args, **kwargs)
  activate T
  T -> T : copy observer list (under lock)
  loop for each observer in copy
  T -> O : update(T, *args, **kwargs)
  activate O
  alt exception
  O --> T : Exception
  T -> T : exception(msg=Notify, reason, observer, error)
  end
  deactivate O
  end
  deactivate T
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| isti promatrač dodan dvaput | drugi poziv ne mijenja popis |
| odjava nepostojećeg | bez učinka i bez poruke |
| promatrač digne iznimku u `update` | `exception(msg=Notify, reason, observer, error)`; ostali promatrači dobivaju događaj |
| promatrač se odjavi tijekom javljanja | kopija se i dalje obilazi do kraja; sljedeće javljanje ga više ne uključuje |
| promatrač dodan tijekom javljanja | dobiva tek sljedeće javljanje |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title ThreadSafeObservable

start
:notify_observers(*args, **kwargs);
:copy observer list under lock;
while (more observers in copy?) is (yes)
:observer.update(self, *args, **kwargs);
if (exception?) then (yes)
  :exception(msg=Notify, reason, observer, error);
else (no)
endif
endwhile (no)
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | kvar promatrača je `exception` zapis s događajem `Notify` |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | kvar jednog promatrača ne doseže druge (blast radius jednog `update`) |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 4 |

## 12. Results

Proizvođač javlja događaj jednom, a svaki promatrač ga primi, bez obzira na kvar drugog i
istodobnu pretplatu ili odjavu iz drugih dretvi.

## 13. Acceptance Criteria

1. Popis nema duplikata. ✅
2. Javljanje ide izvan brave, nad kopijom popisa. ✅
3. Iznimka promatrača ne prekida javljanje ostalima i bilježi se s razlogom, promatračem i greškom. ✅
4. Modul deklarira `__all__`; klasa deklarira `__slots__` s `_observers` i `_observers_lock` (`BR-PTN-07`). ✅ — učinak ovisi o `__slots__` u `IObservableReactive` ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md))
5. Redoslijed javljanja jednak je redoslijedu pretplate. ✅
5a. Pretplata i odjava idu po identitetu: dva jednaka, ali različita promatrača oba primaju javljanje, a odjava jednog ne dira drugog. ✅
6. `add_observer` odbija objekt koji nije `IObserverReactive` s `AttributeException` i ništa ne dodaje. ✅
7. Kvar promatrača ne dospijeva do pozivatelja: bilježi se (`ERROR`, `Notify`), kako propisuje ugovor u `IObservableReactive`. ✅
8. Objekt se može izgraditi (slot brave ne zasjenjuje `Audit._lock`). ✅
9. Promatrač koji se odjavi, doda drugog ili javlja iz druge dretve tijekom javljanja ne blokira i ne mijenja tekući prolaz. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 5a | `IdentityTest` | dva `Equal` promatrača oba javljena; odjava uklanja točno taj objekt; jednak, ali nepretplaćen objekt ne uklanja ništa; isti objekt dvaput ostaje jednom; mutacije (`in`, `==` pri odjavi) ruše testove |
| 1–3, 5, 9 | `workflow/tests/test_observable.py` (`ListTest`, `DeliveryTest`) | bez duplikata, redoslijed pretplate, kopija popisa, javljanje izvan brave (druga dretva se pretplaćuje tijekom prolaza), kvar promatrača ne zaustavlja ostale |
| 4, 8 | `ConstructionTest` | `__slots__ == ("_observers", "_observers_lock")`, `ThreadSafeObservable._lock is Audit._lock`; prije ispravka konstruktor je padao s `AttributeError` |
| 6 | `TypeCheckTest` | `object()`, `None`, string i funkcija odbijeni, popis ostaje prazan |
| 7 | `DeliveryTest` | `ERROR` zapis nosi `Notify`, razlog i grešku; pozivatelj dobiva `None` |
| dretve | `ThreadsTest` | 8 dretvi pretplate/odjave i javljanje 200 puta: bez grešaka, popis ostaje `[stable]`, `stable` primi svih 200 |
| mutacije | ručno | uklonjena provjera tipa, slot `_lock`, popis bez kopije, `update` pod bravom, duplikati, grane `except` — svaka ruši barem jedan test |

**Trojka (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-OBS-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/observable.py`
- Testovi: `workflow/tests/test_observable.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (D-11 uklonjen): **konstruktor je padao** (`DEF-OBS-05`, zatvoren) jer je slot `_lock` zasjenjivao `Audit._lock` — slot preimenovan u `_observers_lock`. `DEF-OBS-01` zatvoren: `add_observer` provjerava tip s `Attribute.evaluate` (`AttributeException`). `DEF-OBS-02` zatvoren kao svojstvo: gutanje iznimki propisuje ugovor u `IObservableReactive`, pa je netočna tvrdnja da odluka nije zapisana. `DEF-OBS-04` zatvoren kao svojstvo: javljanje je izvan brave, a `RLock` samo štiti od povratnog poziva iz `__eq__` promatrača. Kriteriji 6–9, mutacije. `DEF-OBS-03` zatvoren: pretplata i odjava po identitetu (`is`), kriterij 5a, 22 testa. Svi defekti OBS zatvoreni. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: paket wattleflow.core.concurrent, paket wattleflow.concrete.observable, sudionik IObserverReactive, exception(msg=Notify, ...). |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |
