<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-ITR — Lijeni iteratori

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-02`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `LazyIterator(Wattleflow, IIterator[Element])` i `LazyAsyncIterator(Wattleflow, IAsyncIterator[Element])` — iterator koji pravi svoj izvor tek pri prvom dohvatu |
| **Sestrinski** | [`FRQ-PRC`](FRQ-PRC-processor.md) (generator stavki, `IProcessor`) · [`FRQ-PTN`](FRQ-PTN-root-base.md) (korijen) |
| **Izvedba** | `workflow/src/wattleflow/concrete/iterator.py` |

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

Sučelja `IIterator` i `IAsyncIterator` imaju jednu apstraktnu metodu, `create_iterator()`. Ova dva
razreda daju **politiku odgode**: izvor se ne gradi pri konstrukciji nego pri prvom `__next__`
odnosno `__anext__`, a zatim se zadržava. Specijalizacija piše samo `create_iterator`.

| klasa | protokol | izvor koji se gradi |
|---|---|---|
| `LazyIterator` | `__next__` (`__iter__` nasljeđuje od `Iterator`) | `Iterator[Element]` |
| `LazyAsyncIterator` | `__anext__` (`__aiter__` nasljeđuje od `AsyncIterator`) | `AsyncIterator[Element]` |

Jedini član je `_iterator`, `None` do prvog dohvata. Instanca je **jednokratan iterator** (`iter(x) is x`); novi prolaz daje agregat koji stvara novi iterator (`ISyncAggregate.create_iterator()`, kao `Catalog` s `ControlIterator`-om).

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title LazyIterator, LazyAsyncIterator

top to bottom direction

interface "IIterator<Element>" as IIterator {
  + create_iterator() : Iterator[Element]
}
interface "IAsyncIterator<Element>" as IAsyncIterator {
  + create_iterator() : AsyncIterator[Element]
}
class Wattleflow

abstract class "LazyIterator<Element>" as LazyIterator {
  # _iterator : Iterator[Element] | None
  # _build() : Iterator[Element]
  + __next__() : Element
}
abstract class "LazyAsyncIterator<Element>" as LazyAsyncIterator {
  # _iterator : AsyncIterator[Element] | None
  # _build() : AsyncIterator[Element]
  + __anext__() : Element
}
abstract class ThreadSafeLazyIterator {
  # _build_lock : Lock
  + __next__() : Element
}

IIterator <|.. LazyIterator
IAsyncIterator <|.. LazyAsyncIterator
Wattleflow <|-- LazyIterator
Wattleflow <|-- LazyAsyncIterator
LazyIterator <|-- ThreadSafeLazyIterator
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title LazyIterator, LazyAsyncIterator

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "LazyIterator, LazyAsyncIterator", "Iterator that builds its source on first fetch")
System_Ext(po, "Consumer", "for, next, async for")
System_Ext(sp, "LazyIterator subclass", "Implements create_iterator")
Rel_D(po, sys, "Requests next element")
Rel_D(sys, sp, "Builds source")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **Consumer** | potrošač (`for`, `next`, `async for`) | zahtjev za sljedeći element |
| **Specialisation** | specijalizacija | `create_iterator()` — gradi izvor |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 prvi put traži element → `__next__` / `__anext__` |
| **EV02** | A1 traži sljedeći element |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title LazyIterator, LazyAsyncIterator

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {

  actor Consumer
  actor Specialisation
  usecase "Fetch first element (builds source)" as EV01
  usecase "Fetch next element" as EV02
  Consumer --> EV01
  Specialisation --> EV01
  Consumer --> EV02
}
@enduml
```

## 07. Constraints and Preconditions

1. Specijalizacija implementira `create_iterator()` i vraća iterator odgovarajuće vrste (sinkroni ili asinkroni).
2. Konstruktor prosljeđuje cijeli `**kwargs` naviše ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §4 t.2).

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | konstruktor postavlja `_iterator = None`; izvor još ne postoji |
| 2 | **EV01** — `_iterator` je `None` → `create_iterator()`; rezultat se zadržava |
| 3 | `next(_iterator)` odnosno `await _iterator.__anext__()` |
| 4 | **EV02** — izvor se ne gradi ponovno; dohvaća se sljedeći element |
| 5 | kraj izvora — `StopIteration` odnosno `StopAsyncIteration` prolaze nepromijenjeni |

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title LazyIterator, LazyAsyncIterator

participant Consumer
participant LazyIterator
participant Specialisation

Consumer -> LazyIterator : next()
activate LazyIterator
alt _iterator is None
  LazyIterator -> LazyIterator : _build()
  LazyIterator -> Specialisation : create_iterator()
  activate Specialisation
  Specialisation --> LazyIterator : iterator
  deactivate Specialisation
end
LazyIterator -> LazyIterator : next(_iterator)
LazyIterator --> Consumer : element or StopIteration
deactivate LazyIterator
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `create_iterator()` digne iznimku | prolazi nepromijenjena; `_iterator` ostaje `None`, pa sljedeći dohvat ponovno pokušava graditi |
| `create_iterator()` vrati objekt koji nije iterator | `TypeError` iz `next(...)` pri istom dohvatu |
| izvor je iscrpljen | ostaje zadržan i iscrpljen; instanca se ne vraća na početak, novi prolaz traži novi iterator iz agregata |
| `LazyAsyncIterator` zovan izvan petlje događaja | ponašanje preuzima `await`, ne klasa |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title LazyIterator, LazyAsyncIterator

start
:next() or ~__anext~__();
if (_iterator is None?) then (yes)
  :audit Create, Started;
  :create_iterator();
  if (exception or source of the wrong type?) then (yes)
    :audit Create, Failed;
    :<b><color:red>FAILED: exception</color></b>;
    note right: _iterator stays None, so the next fetch builds again
    kill
  endif
  :audit Create, Completed;
  :_iterator = source;
  note right: ThreadSafeLazyIterator builds under _build_lock\nand checks _iterator again inside the lock
endif
:next(_iterator) or await _iterator.~__anext~__();
if (source exhausted?) then (yes)
  :StopIteration or StopAsyncIteration;
else (no)
  :return element;
endif
stop
@enduml
```

## 10. State Machine

Nije primjenjivo: `iterator.py` nema automat ni enum stanja; jedini član je `_iterator`, `None` do prvog dohvata (pretraga koda).

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-PRF-01`](../03-NFRQ/NFRQ-PRF-01-work-proportional-to-input.md) | odgoda sprječava rad nad cijelom kolekcijom prije potrebe |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | uvozi samo standardnu knjižnicu i `wattleflow` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`; kriterij 4 |

## 12. Results

Skup elemenata se ne materijalizira ni ne otvara resurs prije prvog potrošača, a specijalizacija
ostaje jedna metoda.

## 13. Acceptance Criteria

1. `create_iterator()` se ne zove pri konstrukciji. ✅
2. Izvor se gradi najviše jednom po instanci (uz uspjeh). ✅
3. Kraj izvora ne mijenja signal (`StopIteration` / `StopAsyncIteration`). ✅
4. Modul deklarira `__all__`; obje klase deklariraju `__slots__ = ("_iterator",)` (`BR-PTN-07`). ✅ — učinak ovisi o `__slots__` u `IIterator` i `IAsyncIterator` ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md))
5. Instanca je jednokratan iterator (`iter(x) is x`); novi prolaz daje agregat koji stvara novi iterator, a standardne konstrukcije (`zip(x, x)`, `islice`, nastavak nakon `break`) ponašaju se kao kod običnog iteratora. ✅
6. `ThreadSafeLazyIterator`: istodobni prvi dohvat iz više dretvi gradi izvor jednom; osnovni `LazyIterator` ostaje bez brave i nije nit-siguran. ✅
7. Pad `create_iterator()` širi se pozivatelju, a sljedeći dohvat ponovno pokušava graditi izvor (sinkrono i asinkrono). ✅
8. Gradnja izvora bilježi se u auditu (`Event.Create`: `Started`, `Completed`, pri padu `Failed` s razlogom), a izvor koji nije (asinkroni) iterator odbija se s `AttributeException` i ne pohranjuje (sinkrono i asinkrono). ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–4 | `workflow/tests/test_lazy_iterator.py` (21 test) | zadovoljeno; četiri mutacije uhvaćene (gradnja u konstruktoru, `__iter__` s novim izvorom, `__iter__` koji premotava, gradnja pri svakom dohvatu) |
| 5 | `test_lazy_iterator.py` (`SinglePassTest`, `AggregatePassTest`), `blackwattle/tests/oscal/test_control_iterator.py` (5 testova) | `iter(x) is x`; dva prolaza iz dva poziva `Catalog.create_iterator()` daju `['a', 'b', 'b1']` dvaput; usporedba izvedbi u analizi |
| 6 | `workflow/tests/test_lazy_iterator_threads.py` (7 testova) | 8 dretvi na barijeri: obični razred gradi izvor više puta, nit-sigurni jednom (10 krugova); mutacije (bez brave, brava bez ponovne provjere) uhvaćene |
| 7 | `test_lazy_iterator.py` (`FailureTest`, `AsyncLazyIteratorTest`) | pad se širi, sljedeći dohvat gradi ponovno |
| 8 | `workflow/tests/test_lazy_iterator_build.py` (10 testova) | audit i provjera tipa u oba razreda; mutacija (uklonjena provjera tipa) ruši 2 testa po razredu |
| trošak | `workflow/tests/test_lazy_iterator_cost.py` (4 testa) | svaka definirana metoda (`__init__`, `__next__`, `__anext__`) ima izmjeren trošak ispod proračuna od 200 000 ns; sljedeći dohvat 119 ns (sinkrono), 271 ns (asinkrono) |

**Trojka (D-10):** alat — `unittest`, čitanje koda · kriterij — odjeljak 13 · platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`. Usporedba izvedbi i trošak: odjeljak „Analiza” na kraju dokumenta.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-ITR-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/iterator.py`
- Analiza: `FRQ-ITR-lazy-iterators-ANL.md`
- Testovi: `workflow/tests/test_lazy_iterator.py`, `workflow/tests/test_lazy_iterator_build.py`, `workflow/tests/test_lazy_iterator_cost.py`, `workflow/tests/test_lazy_iterator_threads.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | DEF-ITR-02 zatvoren: nova `ThreadSafeLazyIterator(LazyIterator)` s bravom `_build_lock` (ne `_lock`, zbog zasjenjivanja `Audit._lock`) i dvostrukom provjerom, po uzoru na `ThreadSafeObservable`; kriterij 6, 7 testova, 2 mutacije. Svi defekti ITR zatvoreni. |
| v0.0.5 | 2026-10-04 | DEF-ITR-03 i DEF-ITR-04 zatvoreni: `_build()` u `LazyIterator` i `LazyAsyncIterator` bilježi `Event.Create` i provjerava tip izvora s `Attribute.evaluate`; kriterij 8, 10 testova u `test_lazy_iterator_build.py`, mutacije. Vraćen `async def __anext__` u `LazyAsyncIterator` (izmjena izvan sesije ga je zamijenila sinkronim `__next__`). |
| v0.0.5 | 2026-10-04 | DEF-ITR-01 zatvoren kao svojstvo, ne defekt: `LazyIterator` je jednokratan iterator, novi prolaz daje agregat (`Catalog.create_iterator()`); kriterij 5 preformuliran, kriterij 7 (ponovni pokušaj nakon pada izvora), 21 test u `workflow` i 5 u `blackwattle`, mutacije; analiza u `FRQ-ITR-lazy-iterators-ANL.md`. |
| v0.0.5 | 2026-10-04 | Otvorene stavke u 15 preimenovane u defekte `DEF-ITR-<nn>`; odjeljak 10 označen kao nije primjenjivo (nema automata) |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa izvan distribucije `workflow`; uloge iz drugih projekata zamijenjene općim nazivom. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: povratni tipovi `Iterator[Element]` / `AsyncIterator[Element]`; „Specialisation” → `ControlIterator`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |

## Analiza ponovne iteracije na dan 2026-10-04

Cjelovita analiza: `FRQ-ITR-lazy-iterators-ANL.md` (ponašanje četiriju izvedbi po scenarijima, trošak, utjecaj na postojeći kod i na izmjene, testovi i mutacije).
Skripta: `2026-10-04-iterators-analysis-compare.py`. Testovi: `test_lazy_iterator.py`, `test_lazy_iterator_cost.py`, `test_control_iterator.py`.

**Rezultat mjerenja:** konstrukcija oko 1,6 µs; sljedeći dohvat 119 ns (sinkrono) i 271 ns (asinkrono); u `for` petlji oko 70 ns po elementu naspram oko 12 ns za običan iterator, razlika je cijena odgode.
**Rezultat izmjena:** kod nije mijenjan; alternative koje mijenjaju `__iter__` lome protokol iteratora; ponašanje je zaključano testovima.
