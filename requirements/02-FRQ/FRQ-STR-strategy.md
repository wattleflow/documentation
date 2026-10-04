<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-STR — Obitelj strategija

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `Strategy(Wattleflow, IStrategy, ABC)`, četiri obitelji — `StrategyGenerate`, `StrategyCreate`, `StrategyRead`, `StrategyWrite` — i zamjenska `StrategyReadDummy` |
| **Sestrinski** | [`FRQ-BBD`](FRQ-BBD-blackboard.md) (traži `StrategyCreate`) · [`FRQ-REP`](FRQ-REP-repository.md) (čitanje/pisanje) · [`FRQ-DRV`](FRQ-DRV-driver.md) (strategija piše kroz driver) |
| **Izvedba** | `workflow/src/wattleflow/concrete/strategy.py` (156 linija) |
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

Strategija je **zamjenjivi algoritam jedne operacije**. Generički sloj ovdje ne donosi ponašanje
nego **rječnik**: četiri obiteljske podklase imaju gotovo identično tijelo — proslijedi na
`execute` — a razlikuju se **imenom metode** (`write` još pretvara rezultat u `bool`):

| obitelj | metoda | potpis iznad `execute` | vraća |
|---|---|---|---|
| `StrategyGenerate` | `generate(caller, **kwargs)` | — | `ITarget \| None` |
| `StrategyCreate` | `create(caller, **kwargs)` | — | `ITarget \| None` |
| `StrategyRead` | `read(caller, identifier, **kwargs)` | dodaje `identifier` | `ITarget \| None` |
| `StrategyWrite` | `write(caller, facade, **kwargs)` | dodaje `facade: ITarget` | **`bool`** |

**Zašto uopće četiri klase kad je tijelo isto.** Zato što ime metode nosi namjeru na mjestu
poziva: blackboard traži `StrategyCreate` i zove `create`, spremište traži `StrategyWrite` i zove
`write`. Da postoji samo `execute`, pozivatelj bi morao znati koju je vrstu strategije dobio, a
provjera tipa (`assert isinstance(strategy_create, StrategyCreate)` u blackboardu) ne bi imala
što provjeriti. Obitelj je dakle **tipska tvrdnja**, ne kod.

Jedina apstraktna metoda je `execute`; specijalizacija piše nju i ništa drugo. Obiteljske metode zovu `execute` kroz `Strategy._run`, koji **svaki kvar nosi kao `StrategyException` s uzrokom** (`BR-PTN-05`); `StrategyException` koji je specijalizacija već podigla prolazi nepromijenjen. Specijalizaciji zato ne treba vlastiti `try/except` za to.

`StrategyReadDummy(StrategyRead)` je jedina konkretna strategija u modulu: čitanje koje nitko nije
napisao. `execute` na svaki poziv upozori (`warning`) i vrati `DocumentFacade` nad dokumentom koji
sam sebe označava kao zamjenu (metapodaci `implemented=False`, `placeholder`, `declared_by`,
`expected_type`, `identifier`). Tip dokumenta zadaje `document_type`; ako ne prihvaća obavijest
kao `content`, koristi se `DummyReadDocument`. S `strict=True` umjesto vraćanja diže
`StrategyException` koji nosi isti dokument (`failure.document`).

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Strategy

top to bottom direction

interface IStrategy
class Wattleflow

abstract class Strategy {
  + {abstract} execute(caller, **kwargs) : ITarget | None
  # _run(operation, **kwargs)
}
abstract class StrategyGenerate {
  + generate(caller, **kwargs) : ITarget | None
}
abstract class StrategyCreate {
  + create(caller, **kwargs) : ITarget | None
}
abstract class StrategyRead {
  + read(caller, identifier, **kwargs) : ITarget | None
}
abstract class StrategyWrite {
  + write(caller, facade, **kwargs) : bool
}
class StrategyReadDummy {
  # _document_type : type | None
  # _strict : bool
  + strict : bool
  + expected : str
  + execute(caller, identifier, **kwargs) : ITarget
}
class StrategyException

IStrategy <|.. Strategy
Wattleflow <|-- Strategy
Strategy <|-- StrategyGenerate
Strategy <|-- StrategyCreate
Strategy <|-- StrategyRead
Strategy <|-- StrategyWrite
StrategyRead <|-- StrategyReadDummy
Strategy ..> StrategyException : raises
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Strategy

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "Strategy", "Interchangeable algorithm of a single operation")
System_Ext(wf, "WorkflowFactory", "Builds the strategy")
System_Ext(pp, "GenericBlackboard, GenericRepository", "Primitive callers")
System_Ext(dr, "GenericDriver", "External system")
Rel_L(wf, sys, "Builds the strategy")
Rel_R(pp, sys, "Calls create, read, write, generate")
Rel_U(sys, dr, "Calls from execute")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi strategiju iz konfiguracije | preset ključevi, `repository` |
| **A2** | Pozivatelj primitiva (`IWattleflow`) — blackboard, spremište, pipeline | `caller` + operacijski argumenti |
| **A3** | `Strategy` — predmet ovog zahtjeva | algoritam |
| **A4** | `GenericDriver` — vanjski sustav kad ga operacija treba | prima poziv iz `execute` |

`caller` je **obvezni** prvi argument svake operacije: strategija po njemu zna tko je traži, i to
je ono što ulazi u audit trag.

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira strategiju → `__init__` |
| **EV02** | blackboard traži novu stavku → `create(caller, **kwargs)` |
| **EV03** | spremište čita → `read(caller, identifier, **kwargs)` |
| **EV04** | spremište piše → `write(caller, facade, **kwargs)` |
| **EV05** | pozivatelj traži izvedeni sadržaj → `generate(caller, **kwargs)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Strategy

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "WorkflowFactory" as A1
  actor "GenericBlackboard,\nGenericRepository" as A2
  actor "GenericDriver" as A4
    usecase "Instantiate strategy" as EV01
    usecase "Create item (create)" as EV02
    usecase "Read (read)" as EV03
    usecase "Write (write)" as EV04
    usecase "Generate (generate)" as EV05
  A1 --> EV01
  A2 --> EV02
  A2 --> EV03
  A4 --> EV03
  A2 --> EV04
  A4 --> EV04
  A2 --> EV05
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Specijalizacija je implementirala `execute` — inače je klasa apstraktna.
2. `caller` je `IWattleflow`.
3. Strategija koja treba spremište ili driver dobiva ga kroz `kwargs` koje joj predaje spremište (`_strategy_context`, [`FRQ-REP`](FRQ-REP-repository.md)); to je jedini ugovoreni kanal, a generički sloj o driveru ne zna ništa.
4. `execute` u `write` obitelji vraća `bool`: izmjereno nad `blackwattle/src` 57 povrata je doslovno `True`/`False`, a jedan je logički izraz. `write` zato pretvara rezultat s `bool(...)`; `None` je neuspjeh.
5. `None` iz `generate`/`create`/`read` znači „nisam ništa proizveo"; kvar je uvijek iznimka, nikad `None`.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — konstruktor samo otvara kooperativni lanac (`super().__init__(**kwargs)`); ništa se
   ne izdvaja ni ne pamti. Sav `**kwargs` ide naviše ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §4 t.2).
2. **EV02–EV05** — pozivatelj zove metodu **svoje** obitelji; ona kroz `_run` proslijedi na `execute` s
   `caller=` i pripadnim argumentom (`identifier` odnosno `facade`) kao imenovanim; kvar `execute` izlazi kao `StrategyException`.
3. `execute` izvede algoritam i vrati `ITarget` ili `None`.
4. Kod `write` se rezultat prevodi u `bool` istinitošću (`bool(...)` nad rezultatom `_run`).

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Strategy

participant GenericRepository as Repository
participant StrategyWrite as Strategy
participant GenericDriver as Driver

Repository -> Strategy : write(caller, facade, **kwargs)
activate Strategy
Strategy -> Strategy : _run("write", ...)
Strategy -> Strategy : execute(caller, facade, ...)
Strategy -> Driver : operation on the external system
alt execute completed
  Strategy --> Repository : bool(result)
else execute raised
  Strategy --> Repository : StrategyException from the cause
end
deactivate Strategy
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `execute` nije implementiran | `TypeError` pri instanciranju | apstraktna klasa se ne gradi |
| algoritam ne nađe traženo | `execute` vraća `None` | `read`/`create`/`generate` vraćaju `None`, `write` vraća `False` |
| `execute` digne iznimku | **`StrategyException` s uzrokom** (`from e`), poruka `Klasa.operacija error: Tip: tekst`; trag `Executing/Failed` na `DEBUG` | jedan sloj omata, specijalizacije ne moraju |
| `execute` digne `StrategyException` | prolazi nepromijenjen (isti objekt) | ne omata se dvaput |
| `execute` u `write` obitelji vrati `None` ili neistinitu vrijednost | `write` vraća `False` | neuspjeh, ne iznimka |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Strategy

start
:family method (create, read, write or generate);
:_run(operation, arguments);
:execute(caller, arguments);
if (execute raised?) then (yes)
  if (it is a StrategyException already?) then (yes)
    :<b><color:red>FAILED: StrategyException, unchanged</color></b>;
    kill
  endif
  :trace Executing, Failed;
  :<b><color:red>FAILED: StrategyException from the cause</color></b>;
  kill
endif
if (method is write?) then (yes)
  :return bool(result);
else (no)
  :return result as it is;
endif
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Strategy` je rezervirani primitiv; *cross-cutting* sposobnost (rutiranje, digest) je **helper**, ne strategija |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `__all__` izlaže šest imena; obitelj je javna, `execute` je ugovor |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; third-party (OCR, PDF) živi **isključivo** u specijalizacijama u `blackwattle` (`ARCHITECTURE.md` §7.4) |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | ručno prepisano omatanje `try/except → StrategyException` (114 pojava) je otvoreni DRY klaster (`TODO.md` D3) — kandidat za `@strategy_guard` |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | strategija ne otvara vlastitu jedinicu posla; trag joj daje pozivatelj preko `caller` |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Pozivatelj je dobio ishod operacije ne znajući kako je izvedena, a tip strategije koju drži
jamči da je pozvao operaciju koja za nju ima smisla. Zamjena algoritma je izmjena konfiguracije
(`BR-WFL-01`) i ne dira nijednog pozivatelja.

## 13. Acceptance Criteria

1. `execute` je jedina apstraktna metoda cijele obitelji. ✅
2. Svaka obitelj dodaje **samo** ime i potpis, bez vlastite logike. ✅
3. Sve četiri prosljeđuju `caller` kao imenovani argument. ✅
4. Modul deklarira `__all__` sa šest imena; import closure je `wattleflow.*` + `abc` + `__future__`. ✅
5. `StrategyWrite.write` vraća `bool`: `False` kad `execute` vrati `None` ili neistinu. ✅
6. Kvar strategije izlazi kao `StrategyException` s uzrokom u sve četiri obitelji (`BR-PTN-05`); već podignuti `StrategyException` prolazi nepromijenjen. ✅
7. Klase deklariraju `__slots__` i ne ponavljaju slotove baze (`BR-PTN-07`): stanje drži samo `StrategyReadDummy`. ✅
8. Zamjensko čitanje upozorava na svaki poziv, vraća fasadu nad dokumentom koji se sam označava (`implemented=False`) i uz `strict=True` diže `StrategyException` s dokumentom. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2, 3 | `workflow/tests/test_strategy.py` (`ContractTest`) | `__abstractmethods__ == {execute}` za svih pet baza; svaka obitelj prosljeđuje imenovane argumente i vraća rezultat |
| 4 | pregled modula | `__all__` sa šest imena |
| 5 | `WriteResultTest` | `True`, `False`, `None` → `bool` |
| 6 | `FailureTest` | sva četiri puta `StrategyException` s uzrokom i imenom klase, operacije i poruke; vlastiti `StrategyException` je isti objekt; trag samo na `DEBUG` |
| 7 | `SlotsTest`; `test_wattleflow_base.py` | `Strategy.__slots__ == ()`, stanje samo u zamjenskom čitanju |
| 8 | `StandInReadTest` | upozorenje po pozivu, metapodaci, `strict`, tip dokumenta i povratak na zamjenski dokument |
| mjerenje | AST nad `blackwattle/src` | 57 doslovnih `True`/`False` povrata u `execute` write-strategija, jedan izraz (`not outcome['failed']`) |
| mutacije | ručno | iznimka se omata ponovno, bez uzroka, `write` bez `bool`, `read` bez omota — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`, AST pretraga; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-STR-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

Napomena izvan opsega `workflow`: 114 pojava `StrategyException(` u `blackwattle/src/wattleflow/strategies/documents` (klaster D3) sada su većinom suvišne; njihovo uklanjanje je zasebna odluka u `blackwattle`.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/strategy.py`
- Testovi: `workflow/tests/test_strategy.py`, `workflow/tests/test_wattleflow_base.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka prikazuje omatanje kvara. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_strategy.py`, 13 testova), svi defekti zatvoreni: `DEF-STR-01` obitelji sada omataju kvar kroz `Strategy._run` (`StrategyException` s uzrokom, izvorni tip u poruci; vlastiti `StrategyException` prolazi nepromijenjen) — promjena ponašanja: pozivatelji vide `StrategyException`, ne izvorni tip (`GenericRepository` je prilagođen: uzrok mu je `StrategyException`); `-02` kanal prema driveru zapisan (kontekst spremišta); `-03` izmjereno: sve write-strategije vraćaju `bool` (57 + 1 izraz); `-04` `_strict` više nije u bazi i obiteljima, samo u `StrategyReadDummy`; `-05` `None` naspram kvara razgraničen. `blackwattle` suite: bez novih padova. Kriteriji 5–8, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: uloga Primitive caller zamijenjena klasama GenericBlackboard i GenericRepository; zadana vrijednost `identifier : str = ""` u StrategyReadDummy.execute. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
