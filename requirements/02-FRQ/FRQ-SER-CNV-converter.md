<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SER-CNV — Generički konverter

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | Converter je uloga ontologije ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §3) |
| **Nadređeni zahtjev** | [`HLRQ-10-SERIALISATION`](../01-HLRQ/HLRQ-10-SERIALISATION.md) — `BR-CNV-01`, `BR-CNV-02`, `BR-SER-01`, `BR-SER-03`, `BR-SER-04` · [`HLRQ-01-GENERIC-LAYER`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §3 — `BR-DRV-01`, `BR-PTN-05`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericConverter` — kontekst strategije konverzije: drži jednu strategiju i pokreće je |
| **Sestrinski** | [`FRQ-SER-PAR`](FRQ-SER-PAR-parser.md) · [`FRQ-SER-FMT`](FRQ-SER-FMT-formatter.md) (dijelovi strategije) · [`FRQ-STR`](FRQ-STR-strategy.md) |
| **Izvedba** | `workflow/src/wattleflow/concrete/serialisation.py` (`GenericConverter`, `ConverterError`) |
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

Konverter pretvara izvor jednog formata u teret drugog **pokretanjem strategije koju drži**.
Parser i formater su dijelovi strategije; vlastitu klasu strategije opravdava ono što se događa
između `parse` i `render` (transformacija), ne izbor formatera. Konverter nije primitiv sa
životnim ciklusom: nema uređaja, resursa ni pohrane — uloga je nad Strategy i `IStrategyContext`.
Lagana razina: bez audita i logiranja; `__slots__ = ()`.

Konstruktor nastavlja kooperativni lanac (`super().__init__()`), pa se klasa može kombinirati s drugim bazama. `_strategy` živi u rječniku instance jer su `__slots__` prazni: dvije baze s neispražnjenim slotovima ne bi se mogle složiti (npr. s `Wattleflow`).

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericConverter

top to bottom direction

  interface IStrategyContext
  interface IStrategy {
    +execute(caller : IWattleflow, **kwargs) : Any
  }
  abstract class GenericConverter {
    {static} STRATEGY : type[IStrategy] = IStrategy
    {static} ERROR : type[Exception] = ConverterError
    {static} ERRORS : tuple[type[BaseException], ...] = (ConverterError,)
    -_strategy : IStrategy | None
    +name : str
    +strategy : IStrategy | None
    +__init__(strategy : IStrategy | None = None)
    +set_strategy(strategy : IStrategy) : None
    +execute_strategy(caller : IWattleflow, **kwargs) : Any
    +convert(source : Any, **kwargs) : Any
  }
  class ConverterError {
    +caller : object | None
    +error : str
  }
IStrategyContext <|.down. GenericConverter
GenericConverter o-- IStrategy
GenericConverter .right.> ConverterError : ERROR
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericConverter

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

Title Generic Converter
System(sys, "GenericConverter", "Conversion strategy context")
System_Ext(po, "Caller", "Library, pipeline")
System_Ext(sp, "GenericConverter subclass", "Sets STRATEGY, ERROR, ERRORS")
System_Ext(st, "IStrategy", "Conversion strategy")

Rel_L(po, sys, "Calls convert")
Rel_R(sp, sys, "Sets STRATEGY, ERROR")
Rel_U(sys, st, "Invokes execute")
@enduml
```

## 04. User Diagram

| oznaka | tko | ulazi s |
|---|---|---|
| **A1** | pozivatelj (knjižnica, pipeline, strategija) | izvor i opcije |
| **A2** | strategija konverzije | `execute(caller, source=…, **opcije)` → teret |
| **A3** | specijalizacija (podklasa `GenericConverter`) | `STRATEGY`, `ERROR`, `ERRORS` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | konstrukcija sa strategijom ili `set_strategy` |
| **EV02** | A1 zove `convert(source, **opcije)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericConverter

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Caller" as A1
  actor "IStrategy" as A2
  actor "GenericConverter subclass" as A3
    usecase "Set strategy" as EV01
    usecase "Convert source (convert)" as EV02
  A1 --> EV01
  A3 --> EV01
  A1 --> EV02
  A2 --> EV02
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Strategija je instanca `STRATEGY` (zadano `IStrategy`); specijalizacija suzuje `STRATEGY`. Pogrešna daje `ERROR`, a stara strategija ostaje.
2. Strategija se poziva kao `execute(caller, source=…, **opcije)`; pozivatelj je konverter.
3. `ERRORS` zadano sadrži samo `ConverterError`; svaku drugu obitelj specijalizacija dodaje izričito, inače se omata u `ERROR`.
4. `__init__` zove `super().__init__()`; `_strategy` je u `__dict__` instance (`__slots__ = ()` zbog slaganja s `Wattleflow`).

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | `convert` zove `execute_strategy(self, source=source, **opcije)` — konverter je pozivatelj strategije |
| 2 | provjera da je strategija postavljena |
| 3 | `strategy.execute(caller, **opcije)` |
| 4 | teret se vraća nepromijenjen |

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericConverter

  participant "A1 Caller" as A1
  participant "A2 IStrategy" as S
  participant "GenericConverter" as C
  activate A1
  A1 -> C : convert(source, **kwargs)
  activate C
  C -> C : execute_strategy(self, source=source, **kwargs)
  C -> C : _strategy is None?
  C -> S : execute(self, source=source, **kwargs)
  activate S
  S --> C : payload
  deactivate S
  C --> A1 : payload
  deactivate C
  deactivate A1
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| strategija nije postavljena | `ERROR` („no conversion strategy set”) |
| strategija pogrešnog tipa | `ERROR`; postojeća strategija ostaje |
| strategija digne iznimku | `ERROR` s uzrokom (`BR-PTN-05`); iznimka iz `ERRORS` prolazi nepromijenjena |
| strategija vrati teret | vraća se bez preoblikovanja |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericConverter

start
:set_strategy(strategy);
if (strategy is an instance of STRATEGY?) then (no)
:ERROR; old strategy is kept;
stop
else (yes)
endif
:convert(source, **kwargs);
if (strategy set?) then (no)
:ERROR "no conversion strategy set";
stop
else (yes)
endif
:strategy.execute(caller, source=..., **kwargs);
if (exception?) then (yes)
if (exception in ERRORS?) then (yes)
  :propagates unchanged;
else (no)
  :ERROR from e (BR-PTN-05);
endif
stop
else (no)
:return payload unchanged;
stop
endif
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | opći sloj ne uvozi treću stranu i ne poznaje motor |
| [`NFRQ-ORG-13`](../03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) | javna površina je sučelje (`BR-24-03`) |
| [`NFRQ-FUN-01`](../03-NFRQ/NFRQ-FUN-01-conversion-fidelity.md) | vjernost pretvorbe mjeri se u strategijama |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Pozivatelj mijenja ponašanje konverzije zamjenom strategije, ne konverterom; sve što pođe krivo
stiže kao jedna klasa iznimke.

## 13. Acceptance Criteria

1. `convert` pokreće strategiju s konverterom kao pozivateljem i izvorom kao `source`. ✅
2. Bez strategije i s pogrešnom strategijom: `ERROR`. ✅
3. Kvar strategije stiže kao `ERROR` s uzrokom; iznimka iz `ERRORS` prolazi nepromijenjena. ✅
4. Konstruktor nastavlja kooperativni lanac. ✅
5. `STRATEGY` se može suziti na specijalizaciji. ✅
6. Ne nasljeđuje `Wattleflow`; nema audita; `__slots__ = ()`. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | `workflow/tests/test_serialisation.py` (`ConverterTest`) | pozivatelj je konverter; `{"source": 1, "step": 2}` |
| 2, 5 | `ConverterTest` | bez strategije, pogrešan tip, suženi `STRATEGY` |
| 3 | `ConverterTest` | uzrok i poruka; isti objekt za iznimku iz `ERRORS` |
| 4 | `ConverterTest` | repni miks dobiva svoj `__init__` |
| 6 | `SlotsTest` | prazni slotovi |
| mutacije | ručno | bez `super().__init__()` ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-CNV-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/serialisation.py` (`GenericConverter`, `ConverterError`)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_serialisation.py`, 40 testova za sva tri dokumenta), svi defekti zatvoreni: `DEF-CNV-01` `__init__` sada zove `super().__init__()` (docstring više ne spominje nepostojeću audit razinu); `-02` `_strategy` u `__dict__` zapisano uz razlog (slaganje slotova s `Wattleflow`); `-03` široki zadani `STRATEGY` zapisano kao svojstvo (suzuje specijalizacija); `-04` `ERRORS` zadano zapisano kao svojstvo. Kriterij 4, mutacija. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa izvan distribucije `workflow`; uloge iz drugih projekata zamijenjene općim nazivom. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: potpisi i tipovi, oznaka grupe FRQ-SER-CNV, self umjesto C u porukama. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
