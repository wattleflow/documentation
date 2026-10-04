<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-REP — Generičko spremište i varijanta s driverom

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | kategorija `REP` |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericRepository(Wattleflow, IRepository, ABC)` i `RepositoryWithDriver(GenericRepository)` |
| **Sestrinski** | [`FRQ-BBD`](FRQ-BBD-blackboard.md) (jedini pozivatelj `write`) · [`FRQ-STR`](FRQ-STR-strategy.md) (čita i piše) · [`FRQ-DRV`](FRQ-DRV-driver.md) (kanal prema vanjskom sustavu) · [analiza čitanja](../06-ANALYSIS/2026-09-14-read-path.md) |
| **Izvedba** | `workflow/src/wattleflow/concrete/repository.py` |
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

Spremište je **odredište** stavke: ono zna gdje se piše i odakle se čita, ali **ne zna kako**. Kako
je posao strategije ([`FRQ-STR`](FRQ-STR-strategy.md)), a kroz što — drivera ([`FRQ-DRV`](FRQ-DRV-driver.md)). Spremište je mjesto
na kojem se te tri stvari sastaju, i jedino koje broji koliko je zapisa prošlo.

| klasa | ima driver | razlika |
|---|---|---|
| `GenericRepository` | ne | cijeli tok `read` i `write`; kontekst strategije je prazan |
| `RepositoryWithDriver` | da (`ALLOWED = ["driver"]`) | nadjačava **samo** kontekst strategije: dodaje `driver` |

Strategija ne dohvaća driver, nego ga **prima** kroz kontekst — to je jedini kanal kojim strategija
dolazi do vanjskog sustava. Spremište strategiji predaje **sebe** kao `caller`, pa pozivni kontekst
ploče ne curi u sloj strategije.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericRepository, RepositoryWithDriver

top to bottom direction

  interface IRepository
  interface IBlackboard
  class Wattleflow
  class StrategyWrite
  class StrategyRead
  class GenericDriver
  abstract class GenericRepository {
    +count : int
    +write(caller : IBlackboard, facade : ITarget, **kwargs) : bool
    +read(identifier : str, **kwargs) : ITarget | None
    +clear() : None
    #_strategy_context() : dict[str, Any]
  }
  class RepositoryWithDriver {
    {static} ALLOWED : list[str] = ["driver"]
    #_strategy_context() : dict[str, Any]
  }
Wattleflow <|-down- GenericRepository
IRepository <|.down. GenericRepository
GenericRepository <|-down- RepositoryWithDriver
GenericRepository o-- "1" StrategyWrite : _strategy_write
GenericRepository o-- "0..1" StrategyRead : _strategy_read
RepositoryWithDriver o-- "1" GenericDriver
IBlackboard .right.> GenericRepository : write
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericRepository, RepositoryWithDriver

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "GenericRepository, RepositoryWithDriver", "Persistent store of items")
System_Ext(wf, "WorkflowFactory", "Builds the repository")
System_Ext(bb, "GenericBlackboard", "Sole caller of write")
System_Ext(st, "StrategyWrite / StrategyRead", "Perform writing and reading")
System_Ext(dr, "GenericDriver", "Only with a driver")
Rel_L(wf, sys, "Builds the repository")
Rel_R(bb, sys, "Writes on flush")
Rel_U(sys, st, "Delegates the operation")
Rel_D(sys, dr, "Accesses the external system")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi spremište iz konfiguracije | `strategy_write`, `strategy_read`, `driver` |
| **A2** | `GenericBlackboard` — jedini pozivatelj `write` | `caller`, `facade` |
| **A3** | pozivatelj `read` — podklasa blackboarda (generička klasa `read` ne zove) | `identifier` |
| **A4** | `StrategyWrite` / `StrategyRead` | operacija |
| **A5** | `GenericDriver` — samo u varijanti s driverom | vanjski sustav |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira spremište → `__init__` |
| **EV02** | A2 piše pri pražnjenju → `write(caller, facade)` |
| **EV03** | A3 čita → `read(identifier)` |
| **EV04** | brojanje se resetira → `clear()` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericRepository, RepositoryWithDriver

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "WorkflowFactory" as A1
  actor "GenericBlackboard" as A2
  actor "GenericBlackboard subclass" as A3
  actor "StrategyWrite / StrategyRead" as A4
  actor "GenericDriver" as A5
    usecase "Instantiate repository" as EV01
    usecase "Write on flush" as EV02
    usecase "Read" as EV03
    usecase "Reset count" as EV04
  A1 --> EV01
  A2 --> EV02
  A4 --> EV02
  A5 --> EV02
  A3 --> EV03
  A4 --> EV03
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. `strategy_write` je `StrategyWrite`; `strategy_read`, ako je zadan, je `StrategyRead` — provjera prije konstrukcije (`RepositoryException`, ostaje i pod `python -O`). Bez `strategy_read` čitanje je onemogućeno.
2. Za `RepositoryWithDriver`: `driver` je instanca `GenericDriver` — ista provjera.
3. U `write`: `caller` je `IBlackboard`, `facade` je `ITarget` — provjera prije čitanja `caller.name` i `facade.identifier`.
4. Jednakost i hash spremišta su zadani (identitet): spremište je objekt koji drži brojač i strategije.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — strategije i driver se provjere izričito (`RepositoryException`, ne `assert`), preset preuzme konfiguraciju, brojač krene od nule.
2. **EV02** — `write` najprije provjeri tipove (`caller` je `IBlackboard`, `facade` je `ITarget`; inače `RepositoryException` prije bilo čega drugog), zatim zapisuje `DEBUG` s pozivateljem i brojačem i pozove strategiju
   s `caller=self`, `facade`, `repository=self` i kontekstom (`driver` u varijanti s driverom).
3. Brojač se poveća **samo kad strategija vrati uspjeh**; rezultat se vraća pozivatelju.
4. **EV03** — `read` zapisuje `DEBUG` i pozove strategiju čitanja s `caller=self`, `identifier` i
   kontekstom. Tko danas čita i kojim putem: [analiza čitanja](../06-ANALYSIS/2026-09-14-read-path.md) §3.
5. **EV04** — `clear()` zapisuje `INFO` s brojem zapisa i vraća brojač na nulu.

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericRepository, RepositoryWithDriver

  participant "GenericBlackboard" as B
  participant "GenericRepository" as R
  participant "StrategyWrite" as W
  participant "GenericDriver" as D
  activate B
  B -> R : write(caller, facade)
  activate R
  R -> R : type checks
  R -> W : write(caller=self, facade, repository=self, driver)
  activate W
  W -> D : write(...) (only with a driver)
  activate D
  deactivate D
  W --> R : True or False
  deactivate W
  R -> R : _write_counter += 1 (only on True)
  R --> B : result
  deactivate R
  deactivate B
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| strategija ili driver pogrešnog tipa | `RepositoryException` prije konstrukcije — spremište ne nastaje |
| čitanje bez strategije čitanja | `warning` i `None` — odsutnost čitanja je konfiguracija, ne kvar |
| `caller` ili `facade` pogrešnog tipa u `write` | `RepositoryException` prije zapisa i prije poziva strategije; strategija se ne zove |
| strategija pisanja ili čitanja padne | `RepositoryException` s uzrokom (`BR-PTN-05`), u obje klase; uzrok je `StrategyException` ([`FRQ-STR`](FRQ-STR-strategy.md)), a njegov uzrok izvorni kvar |
| kvar pisanja | poruka nosi strategiju, tipove iz konteksta (driver), identifikator dokumenta, pozivatelja i tip iznimke |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericRepository, RepositoryWithDriver

start
if (operation?) then (read)
if (read strategy exists?) then (no)
  :warning; return None;
  stop
else (yes)
  :_strategy_read.read(caller=self, identifier);
  if (exception?) then (yes)
    :RepositoryException with cause;
    stop
  else (no)
    :return result;
    stop
  endif
endif
else (write)
endif
:write(caller, facade);
if (caller and facade of correct type?) then (no)
:RepositoryException before the strategy is called;
stop
else (yes)
endif
:_strategy_write.write(caller=self, facade, repository=self);
if (strategy raises an exception?) then (yes)
:RepositoryException with cause (BR-PTN-05);
stop
else (no)
endif
if (strategy returned success?) then (yes)
:_write_counter += 1;
else (no)
:_write_counter unchanged;
endif
:return result;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Repository` je rezervirani primitiv; ono je odredište, ne platno ([`FRQ-BBD`](FRQ-BBD-blackboard.md)) |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | spremište poznaje jedan driver; kompromitacija jednog odredišta ne doseže druga |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `ALLOWED = ["driver"]` je cijela konfiguracijska površina varijante s driverom |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | `read` i `write` na `DEBUG`, `clear` na `INFO`, kvar jednom |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | jedan tok u roditelju; dijete nadjačava samo kontekst strategije |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Stavka je trajno zapisana kroz strategiju koja zna format i driver koji zna protokol, a spremište
zna samo koliko ih je prošlo. Zamjena odredišta je izmjena konfiguracije (`BR-WFL-01`).

## 13. Acceptance Criteria

| | kriterij | stanje |
|---|---|---|
| 1 | Strategije i driver provjeravaju se prije konstrukcije, izričito (ostaje pod `python -O`) | ✅ |
| 2 | Spremište predaje **sebe** kao `caller` | ✅ |
| 3 | Driver stiže strategiji kroz kontekst spremišta | ✅ |
| 4 | `read` i `write` omotani su u `RepositoryException` u obje klase (`BR-PTN-05`) | ✅ |
| 5 | Tok `read`/`write` postoji jednom, u roditelju ([`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md)) | ✅ |
| 6 | `hash()` i `repr()` rade na svakoj podklasi; jednakost je identitet i ne piše audit zapis | ✅ |
| 7 | Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow` | ✅ |
| 8 | Pogrešan `caller` ili `facade` u `write` daje `RepositoryException`, ne `AttributeError` iz izvještaja o kvaru | ✅ |
| 9 | Neizgrađeno spremište na nepoznato ime odgovara tim imenom (ne `_preset`) | ✅ |

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | `workflow/tests/test_repository.py` (`ConstructionTest`) | pogrešna strategija pisanja/čitanja i driver odbijeni; potproces `python -O` odbija |
| 2, 3 | `WriteTest`, `ReadTest`, `ConstructionTest` | strategija prima spremište kao `caller` i `repository=self`; `driver` stiže kroz kontekst |
| 4 | `WriteTest`, `ReadTest` | `RepositoryException` s uzrokom; poruka nosi strategiju, kontekst, identifikator, pozivatelja i tip iznimke |
| 5 | pregled | `RepositoryWithDriver` nadjačava samo `_strategy_context` |
| 6 | `IdentityTest` | `==` je identitet, hash stabilan, dva spremišta u skupu; usporedba ne piše zapis |
| 7 | pregled modula | `__all__`; uvozi `abc`, `typing` + `wattleflow.*` |
| 8, 9 | `WriteTest`, `IdentityTest` | `object()`, `None`, string kao `caller`; `object()`, `None` kao `facade`; strategija se ne zove |
| mutacije | ručno | bez provjere pozivatelja, bez provjere strategije, nezaštićen `__getattr__`, `__eq__` koji piše zapis — svaka ruši test |
| čitanje (svojstvo) | pretraga `\.read\(` nad `workflow/src` i `blackwattle/src` | jedini pozivatelj spremišta je `blackboards/claude.py:579`; tok `Processor`→`Pipeline`→`Blackboard` čitanje ne koristi ([analiza čitanja](../06-ANALYSIS/2026-09-14-read-path.md) §7) |

**Trojka reproducibilnosti (D-10):** alat — `unittest`, pretraga koda; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-REP-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/repository.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka više ne spominje `assert`. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_repository.py`, 21 test), svi defekti zatvoreni: `DEF-REP-02` pogrešan `caller`/`facade` u `write` davao je `AttributeError` (izvještaj o kvaru čita `caller.name` i `facade.identifier`), a provjera je bila `assert` unutar `try` — sada izričita provjera prije svega; `DEF-REP-01` čitanje bez pozivatelja u toku zatvoreno kao provjereno svojstvo (jedini pozivatelj `claude.py`). **Novo:** konstruktorske provjere bile su `assert` (nestaju pod `python -O`) — izričite; `__eq__` je uspoređivao hasheve koji počinju s `id(self)`, pisao audit zapis po usporedbi i mogao dvije različite instance proglasiti jednakima pri koliziji — uklonjen, vrijedi identitet; `__getattr__` na neizgrađenom spremištu odgovara traženim imenom; zakomentirani `trace=` obrisan. Kriteriji 8–9, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa i poveznice izvan distribucije `workflow`. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa izvan distribucije `workflow`; uloge iz drugih projekata zamijenjene općim nazivom. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: „Caller of read” zamijenjen klasom ClaudeBlackboard, uklonjena veza GenericBlackboard–clear (nitko ne poziva), `_write_counter`, `_strategy_read`/`_strategy_write`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |
