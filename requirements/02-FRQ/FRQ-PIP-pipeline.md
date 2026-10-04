<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-PIP — Generički pipeline

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericPipeline(Wattleflow, IPipeline, ABC)` — jedna transformacija nad jednom stavkom; uz njega `PipelineError` |
| **Sestrinski** | [`FRQ-PRC`](FRQ-PRC-processor.md) (poziva `process`) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) (prima rezultat) · [`FRQ-DOC`](FRQ-DOC-document.md) (`facade`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/pipeline.py` |
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

Pipeline je **najmanja jedinica transformacije**: jedna stavka ulazi, jedna transformacija se
izvodi. Sve što je oko toga — koliko stavki ima, odakle dolaze, kada se rezultat trajno sprema —
ne pripada mu.

Klasa razdvaja dvije metode koje se lako pomiješaju:

| metoda | tko je piše | što radi |
|---|---|---|
| `transform(processor, facade, **kwargs)` | **specijalizacija** — apstraktna | sam posao |
| `process(processor, facade, **kwargs)` | **generički sloj** — konkretna | okvir oko posla |

`process` je omotač koji radi četiri stvari koje specijalizacija ne smije ponavljati: provjeri
tipove ulaza, otvori audit zapis, pozove `transform`, i **svaku** iznimku omota u `PipelineError`
s uzrokom. Time je `BR-PTN-05` ispunjen na jednom mjestu za sve pipelinee.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericPipeline

top to bottom direction

interface IPipeline {
  + process(processor : IProcessor, facade : ITarget, **kwargs)
}
class Wattleflow

abstract class GenericPipeline {
  # _preset : PresetDecorator
  + process(processor : IProcessor, facade : ITarget, **kwargs)
  + {abstract} transform(processor : IProcessor, facade : ITarget, **kwargs) : Any
}

class PipelineError
class PipelineException
class PresetDecorator
interface IProcessor
interface ITarget

IPipeline <|.. GenericPipeline
Wattleflow <|-- GenericPipeline
PipelineException <|-- PipelineError
GenericPipeline *-- PresetDecorator
GenericPipeline ..> PipelineError : raises
GenericPipeline ..> IProcessor : checks
GenericPipeline ..> ITarget : checks
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericPipeline

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "GenericPipeline", "Transforms a single item")
System_Ext(wf, "WorkflowFactory", "Builds pipeline")
System_Ext(pr, "GenericProcessor", "Owns the pass")
System_Ext(bb, "GenericBlackboard", "Result destination")
Rel_L(wf, sys, "Builds pipeline")
Rel_R(pr, sys, "Calls process per item")
Rel_U(sys, bb, "Receives write from transform")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi pipeline iz konfiguracije | `level`, `handler`, preset ključevi |
| **A2** | `GenericProcessor` — poziva `process` po stavci | `processor`, `facade` |
| **A3** | `GenericPipeline` — predmet ovog zahtjeva | transformacija |
| **A4** | `GenericBlackboard` — odredište rezultata | prima `write` iz `transform` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira pipeline → `__init__` |
| **EV02** | A2 poziva `process(processor, facade)` za jednu stavku |
| **EV03** | `transform` vrati rezultat |
| **EV04** | `transform` ili provjera ulaza padne |
| **EV05** | kraj životnog ciklusa → `__del__` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericPipeline

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor WorkflowFactory
  actor GenericProcessor
  actor GenericBlackboard
  
  usecase "Instantiate pipeline" as initiate
  usecase "Process item" as process
  usecase "Write to blackboard" as write
  usecase "Report fault" as raise
  usecase "End lifecycle" as destroy
  
  WorkflowFactory --> initiate
  GenericProcessor --> process
  GenericBlackboard <-- write
  GenericProcessor <-- raise
  WorkflowFactory --> destroy
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. `processor` je `IProcessor`, `facade` je `ITarget` — provjereno izričito u `process` (`PipelineError`), ne `assert`-om, pa provjera ostaje i pod `python -O`.
2. Preset je izgrađen; konfiguracijska imena se razrješavaju kroz `__getattr__`.
3. Specijalizacija je implementirala `transform` — inače je klasa apstraktna i ne instancira se.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — konstruktor prosljeđuje `level` i `handler` naviše zajedno s cijelim `**kwargs`,
   prijavljuje `Constructor` na `DEBUG`, pa gradi `PresetDecorator` (redoslijed: `debug` prije preseta).
2. **EV02** — `process` prijavljuje `Transform` (bez `step`-a) na `DEBUG`, s poljima `processor` i `facade`.
3. Provjera ulaza (`assert`): `processor` mora biti `IProcessor`, `facade` mora biti `ITarget`.
4. **Zapis na ulazu**, na `DEBUG`: `Transform/Started` s imenom izvora i identifikatorom
   dokumenta. Otvara se **prije** posla namjerno — mjesto zapisa i njegova polja odgovaraju redu
   kojim se posao odvija. Razina je `DEBUG`, ne `INFO`, jer obradu dokumenta
   posjeduje **procesor**, a pipeline je korak unutar te jedinice: `INFO` po pipelineu množio bi
   zapise brojem konfiguriranih pipelinea umjesto brojem dokumenata
   .
   Zatvaranje jedinice — i po dokumentu i po prolazu — pripada procesoru i workflowu.
5. Ime izvora izvodi `NameHelper.source_name(facade)` — `facade.filename` sveden na osnovno ime
   putanje. Metoda **nikad ne diže**: dokument bez imena datoteke daje `None`. Stoji u
   `concrete/helpers.py` jer je dijele pipeline i procesor — helper dvaju pod-paketa iste domene
   ide u domenski-interni dijeljeni modul ([`NFRQ-ORG-01`](../03-NFRQ/NFRQ-ORG-01-helper-locality.md)).
6. **EV03** — `transform` se izvede; `Transform/Completed` na `DEBUG` s rezultatom.

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericPipeline

participant GenericProcessor as Processor
participant GenericPipeline as Pipeline  #green
participant NameHelper
participant GenericBlackboard as Blackboard

Processor -> Pipeline : process(processor, facade)
activate Pipeline #gold
Pipeline -> Pipeline : check processor and facade types
Pipeline -> NameHelper : source_name(facade)
activate NameHelper #gold
NameHelper --> Pipeline : name or None
deactivate NameHelper

Pipeline -> Pipeline : transform(processor, facade)
opt transform writes its result
  Pipeline -> Blackboard : write(pipeline, facade)
end
alt completed
  Pipeline --> Processor : done
else raises
  Pipeline --> Processor : PipelineError from the cause
end
deactivate Pipeline
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `processor` nije `IProcessor` ili `facade` nije `ITarget` | `PipelineError(caller=self, error=…)`; `Transform/Failed` na `DEBUG`; `transform` se ne poziva | kvar konfiguracije, ne podatka |
| `transform` digne bilo što | `PipelineError(caller=self, error=…)` s `from e`; `Transform/Failed` na `DEBUG` | trag kaže **koji** je pipeline pao (`BR-PTN-05`) |
| bilo koji kvar | ovaj sloj piše samo `DEBUG` trag i **prosljeđuje** | `ERROR` piše pozivatelj koji zaustavlja propagaciju |
| `facade` nema `filename` | `NameHelper.source_name` vraća `None` | zapis nastaje i bez imena izvora |
| `__init__` padne prije `_preset` | `__del__` i `__getattr__` to prepoznaju: bez šuma, a pogreška je ona izvorna | konstrukcija koja padne ne ostavlja trag u destruktoru |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericPipeline

start
:process(processor, facade);
:trace Transform;
if (processor is IProcessor and facade is ITarget?) then (no)
  :trace Transform, Failed;
  :<b><color:red>FAILED: PipelineError</color></b>;
  kill
endif
:trace Transform, Started;
:transform(processor, facade);
if (transform raised?) then (yes)
  :trace Transform, Failed;
  :<b><color:red>FAILED: PipelineError from the cause</color></b>;
  kill
endif
:trace Transform, Completed;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Pipeline` je rezervirani primitiv; transformacija se ne seli u procesor ni u strategiju |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | **nijedan** `INFO` iz ovog sloja; `DEBUG` nosi ulaz, rezultat i kvar |
| [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) | polja zapisa na ulazu su `msg`, `step`, `source`, `document` — nepromijenjena izmjenom razine |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | volumen na `INFO` je **nula**; stavka je jedinica posla **procesora**, ne pipelinea |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `__all__` izlaže dva imena; imenovanje izvora je preseljeno u `NameHelper` |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; nijedan third-party uvoz |
| [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) | `NameHelper.source_name` je `@staticmethod` i doista ne dira nijedan član klase — izbor dekoratora izražava namjeru |
| [`NFRQ-ORG-01`](../03-NFRQ/NFRQ-ORG-01-helper-locality.md) | imenovanje izvora dijele pipeline i procesor, pa stoji u `concrete/helpers.py`, ne duplicirano u oba |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Jedna stavka je transformirana, rezultat je predan dalje (tipično na platno), a audit tok nosi
otvarajući zapis te transformacije na mjestu koje odgovara redoslijedu izvođenja — na `DEBUG`
razini. Na `INFO` razini ovaj sloj **šuti**; stavku prijavljuje njezin vlasnik, procesor. Kvar je
omotan u `PipelineError` s očuvanim uzrokom i ne stiže pozivatelju kao anonimna iznimka.

## 13. Acceptance Criteria

1. `transform` je apstraktna; `process` je konkretna i jedina ulazna točka za pozivatelja. ✅
2. Svaka iznimka iz `transform` izlazi kao `PipelineError` s očuvanim `__cause__`. ✅
3. Otvarajući zapis nastaje **prije** posla, na `DEBUG` razini. ✅
   - **3a.** Ovaj sloj ne emitira nijedan `INFO` zapis. ✅
4. Ovaj sloj ne piše `ERROR` — piše `DEBUG` trag i prosljeđuje. ✅
5. `NameHelper.source_name` nikad ne diže iznimku. ✅
6. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
7. `__del__` preživi neuspjelu konstrukciju (bez nepodignute iznimke iz destruktora), a `__getattr__` na neizgrađenom objektu odgovara traženim imenom. ✅
8. Klasa deklarira `__slots__` s imenom `_preset` (`BR-PTN-07`). ✅ — učinak ovisi o `__slots__` u `IPipeline` ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md))
9. Provjera ulaza ne ovisi o `assert`-u: pod `python -O` pogrešan ulaz je i dalje `PipelineError`. ✅
10. `PipelineError` je podrazred `PipelineException`: jedna obitelj kvara pipelinea, uhvatljiva jednim `except`. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | `workflow/tests/test_pipeline.py` (`ContractTest`) | klasa bez `transform` se ne instancira; `process` predaje argumente |
| 2, 4 | `FailureTest`, `AuditTest` | `PipelineError` s `__cause__`, ime pipelinea u poruci i `caller`; `DEBUG` trag, nijedan zapis iznad `DEBUG` |
| 3, 3a | `AuditTest` | `Started` prethodi `Completed`; sloj piše samo `DEBUG` |
| 5 | pregled `NameHelper.source_name` ([`FRQ-HLP`](FRQ-HLP-helpers.md)) | `try/except Exception: return None` |
| 6, 8 | pregled modula | `__all__ = ["GenericPipeline", "PipelineError"]`; `__slots__ = ("_preset",)` |
| 7 | `LifecycleTest` | `sys.unraisablehook` ne dobiva ništa nakon neuspjele konstrukcije; neizgrađen objekt odgovara traženim imenom |
| 9 | `FailureTest` | potproces `python -O`: pogrešan procesor je `PipelineError` |
| 10 | `FailureTest` | `issubclass(PipelineError, PipelineException)` |
| mutacije | ručno | bez provjere fasade, nezaštićen `__del__`, bez uzroka, odvojena obitelj iznimki — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-PIP-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/pipeline.py`
- Testovi: `workflow/tests/test_pipeline.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka više ne spominje `AssertionError`. |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (D-11 uklonjen), svi defekti zatvoreni: `DEF-PIP-01` `__del__`/`__getattr__` zaštićeni od nepostavljenog `_preset` (**dokument je krivo tvrdio da se izvorna iznimka maskira: iznimke iz `__del__` Python ne propagira, ostaje šum na stderr**); `DEF-PIP-02` zakomentirani `trace=` obrisan, uzrok nosi `__cause__`; `DEF-PIP-03` `PipelineError` je podrazred `PipelineException` (jedna obitelj). **Novo:** provjera ulaza bila je `assert` i nestajala pod `python -O` — zamijenjena izričitom provjerom; zapis `preset=` uklonjen iz `__del__`. Kriteriji 7–10, 18 testova, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — obrnuto inženjerstvo zatečenog. Ovaj sloj ne emitira `INFO` |
