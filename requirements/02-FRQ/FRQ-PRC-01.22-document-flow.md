<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-PRC-01.22 — Tok dokumenta: nastanak i pohrana

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | Kategorija `PRC` odabrana po vlasništvu prolaza; izbor je otvoren (odjeljak 15 t.3) |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ i zajednički ugovor generičke klase |
| **Predmet** | **Suradnja** primitivâ u dva toka, ne građa jednog primitiva: nastanak dokumenta i njegova pohrana |
| **Sestrinski** | [`FRQ-PRC`](FRQ-PRC-processor.md) · [`FRQ-PIP`](FRQ-PIP-pipeline.md) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) · [`FRQ-STR`](FRQ-STR-strategy.md) |
| **Izvedba** | `concrete/processor.py`, `concrete/pipeline.py`, `concrete/repository.py`, `concrete/blackboard.py`, `concrete/strategy.py` (workflow) · `blackboards/small.py`, `processors/file.py`, `strategies/documents/*.py` (blackwattle) |
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

Zahtjev opisuje **dva toka** kojima dokument putuje kroz sustav, i pravilo da nijedan sudionik ne
preuzima tuđi korak. Tok nastanka odgovara na *što je ovaj predmet*, tok pohrane na *gdje ide ono
što je nad njim napravljeno*.

Odluke koje tokovi utjelovljuju:

| pitanje | nositelj | ne smije nositi |
|---|---|---|
| koje jedinice postoje | procesor | sadržaj, imenovanje; brisanje izvora samo uz potvrđen flush (`FileDocumentProcessor`) |
| što je predmet | create strategija | transformaciju, pretvorbu, ime izlaza |
| što s njim učiniti | pipeline | pohranu, odabir odredišta |
| kamo ide | write strategija (jedna po repozitoriju) | sam upis u spremište |
| kako se čita i piše | driver | odluku *što* i *kamo* |

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Document flow

top to bottom direction

  class WorkflowFactory
  class DriverManager
  abstract class GenericProcessor
  abstract class GenericBlackboard
  abstract class StrategyCreate
  abstract class GenericPipeline
  abstract class GenericRepository
  class RepositoryWithDriver
  abstract class StrategyWrite
  abstract class GenericDriver
  abstract class DocumentFacade
WorkflowFactory .right.> DriverManager : registers drivers
WorkflowFactory .right.> GenericProcessor : builds
DriverManager o-- "0..*" GenericDriver
GenericProcessor o-- "1" GenericBlackboard
GenericProcessor o-- "1..*" GenericPipeline
GenericBlackboard o-- "1" StrategyCreate
GenericBlackboard o-- "1..*" RepositoryWithDriver
RepositoryWithDriver o-- "1" StrategyWrite
RepositoryWithDriver -up-|> GenericRepository
RepositoryWithDriver -right-> "1" GenericDriver
StrategyCreate .right.> DocumentFacade : returns
GenericPipeline .right.> GenericBlackboard : write
StrategyWrite .right.> GenericDriver : write
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Document flow

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "Document flow", "Creates and stores a document through the primitives")
System_Ext(wf, "WorkflowFactory", "Builds and wires")
System_Ext(vs, "External system", "File, database, service")
System_Ext(iz, "Unit source", "Files or records")
Rel_L(wf, sys, "Builds drivers, processors and repositories")
Rel_R(sys, vs, "Receives written content")
Rel_U(iz, sys, "Supplies units of the pass")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `DriverManager` — drži driver po imenu, jednu instancu; pri uništenju ga otpušta (`ensure_unloaded`) | registracija iz tvornice |
| **A2** | `WorkflowFactory` — gradi drivere (`_build_drivers`), razrješava imena u instance i povezuje ih | konfiguracija |
| **A3** | `GenericProcessor` — vlasnik prolaza | generator, pipelinei, granica flusha |
| **A4** | `GenericBlackboard` — platno između stope pipelinea i stope repozitorija | `strategy_create`, registrirani repozitoriji |
| **A5** | `StrategyCreate` — nastanak dokumenta | opisni ključevi |
| **A6** | `GenericPipeline` — jedna transformacija nad jednom stavkom | facade + ključevi procesora |
| **A7** | `RepositoryWithDriver` — **jedan po odredištu**; nosi svoj driver i svoju write strategiju | `driver`, `strategy_write` |
| **A8** | `StrategyWrite` — priprema plaćeni sadržaj i imenuje izlaz | facade, driver |
| **A9** | `GenericDriver` — čita, piše, pretražuje | payload, ime, sufiks |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger | tok |
|---|---|---|
| **EV01** | A2 gradi driver i registrira ga u A1 (`register_driver`); driver ostaje u stanju `PENDING` | priprema |
| **EV02** | A2 predaje driver repozitoriju (`_driver_context`) i procesoru (`configuration.driver`) | priprema |
| **EV03** | generator daje jedinicu → `blackboard.create(caller=procesor, …)` | nastanak |
| **EV04** | blackboard poziva `strategy_create.create(caller=self, processor=…, blackboard=self, …)` | nastanak |
| **EV05** | strategija vraća `DocumentFacade` → procesor ga `yield`-a | nastanak |
| **EV06** | `pipeline.process(processor, facade)` → `transform(...)` | pohrana |
| **EV07** | pipeline zove `processor.blackboard.write(pipeline=self, facade=…)` | pohrana |
| **EV08** | ciklus dovršen → `blackboard.flush(caller=procesor, **processor.write_context)` | pohrana |
| **EV08a** | flush dovršen → procesor piše granicu dokumenta (`msg=Processed`) | pohrana |
| **EV09** | blackboard obilazi **svaki** registrirani repozitorij → `repository.write(...)` | pohrana |
| **EV10** | repozitorij zove `strategy_write.write(..., driver=self.driver, ...)` | pohrana |
| **EV11** | strategija zove `driver.write(payload, filename=…, suffix=…)` | pohrana |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Document flow

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "DriverManager" as A1
  actor "WorkflowFactory" as A2
  actor "GenericProcessor" as A3
  actor "GenericBlackboard" as A4
  actor "StrategyCreate" as A5
  actor "GenericPipeline" as A6
  actor "RepositoryWithDriver" as A7
  actor "StrategyWrite" as A8
  actor "GenericDriver" as A9
    usecase "Build and hand over driver" as EV01
    usecase "Create document" as EV02
    usecase "Transform and write to canvas" as EV03
    usecase "Flush canvas to repositories" as EV04
  A1 --> EV01
  A2 --> EV01
  A3 --> EV02
  A4 --> EV02
  A5 --> EV02
  A3 --> EV03
  A4 --> EV03
  A6 --> EV03
  A3 --> EV04
  A4 --> EV04
  A7 --> EV04
  A8 --> EV04
  A9 --> EV04
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Driver je instanciran i registriran **prije** procesora — instancu drži `DriverManager`, ne
   potrošač (`BR-WFL-02` za registraciju, [`FRQ-DRV`](FRQ-DRV-driver.md) za sam driver). Podiže se lijeno: driver
   zove `ensure_live()` pri prvom `read` / `write`.
2. Blackboard nosi `strategy_create`; bez nje se ne konstruira (`BlackboardException`, provjera ostaje i pod `python -O`).
3. Blackboard nosi **barem jedan** repozitorij; bez njega `write` diže `BlackboardException`.
4. Svaki repozitorij nosi vlastitu `strategy_write`; `RepositoryWithDriver` uz to i `driver`.

## 08. Sequence Diagrams

### Normalan tok

#### 5.1 Nastanak

1. Procesor otkriva jedinicu i filtrira je (ime, datum, redoslijed). **Ne otvara je.**
2. Poziva `blackboard.create(self, <opisni ključevi>)`. Blackboard provjerava da je pozivatelj `IProcessor`.
3. Blackboard prosljeđuje **sebe** kao `caller`, a procesor kao `processor=` — pa create
   strategija provjerava `IBlackboard`, ne `IProcessor`.
4. Strategija provjeri obvezne ključeve, sagradi dokument svojeg predmeta, žigoše ishodišne
   metapodatke (`created_by`, `created_at`, `caller`, `source_format`) i vrati `DocumentFacade`.
5. Procesor `yield`-a facade. Identitet dokumenta od te točke stoji.

#### 5.2 Pohrana

1. Procesor provlači facade kroz **sve** pipelinee redom.
2. `GenericPipeline.process` provjerava tipove, piše zapise **na `DEBUG`** i zove
   `transform` — jedina metoda koju specijalizacija piše. Na `INFO` razini pipeline šuti:
   dokument je jedinica posla procesora, ne pipelinea.
3. `transform` uzme dokument iz facade, izvede **svoju jednu** obradu i obogati dokument
   sadržajem i/ili metapodacima.
4. Pipeline zove `processor.blackboard.write(pipeline=self, facade=…)`. Blackboard postavi platno;
   uz `defer_flush=False` odmah emitira u sve repozitorije, inače čeka flush.
5. Nakon svih pipelinea procesor uveća ciklus i — ako je `flush_per_cycle` — zove
   `blackboard.flush(caller=self, **self.write_context)`. Generički `write_context` je prazan
   rječnik; nijedna podklasa ga danas ne nadjačava (ispitana točka proširenja), pa strategije ne dobivaju nijedan
   ključ ovim putem.
6. Blackboard obilazi **svaki registrirani repozitorij redom** (`for`, ne istodobno). *N*
   repozitorija znači *N* write strategija i *N* drivera nad **istim** dokumentom.
7. Repozitorij pridoda vlastiti driver kroz `_strategy_context()` i zove svoju write strategiju.
8. Strategija razriješi formatter (`FormatterFactory`), sagradi payload, složi ime izlaza i preda
   ga `driver.write(...)`. Ime se **komponira pri upisu**, nikad ne čuva unaprijed.
9. Strategija žigoše metapodatke pohrane i vrati `True`; repozitorij uveća brojač; platno se prazni.
   `SmallBlackboard.flush` zbraja odgovore repozitorija u jedan `bool` i vraća ga; procesor ga pamti
   za dokument (`flush_outcome`). Tek uz `True` `FileDocumentProcessor` smije ukloniti izvor
   (`delete_source`, `FRQ-PRC-25.6`). `Large`, `Bundle` i `Claude` blackboard vraćaju
   `None`, što se tumači kao nepotvrđeno.
10. **EV08a** — procesor zatvara jedinicu dokumenta jednim `INFO` zapisom (`Processed`, s
    `cycle`, `source`, `document`). To je **jedini** `INFO` koji ovaj tok proizvodi po dokumentu
    iz sloja vlasnika; koraci lanca prijavljuju se čije je zatečeno stanje
    nalaz ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) §4).

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Document flow

  participant "GenericProcessor" as P
  participant "GenericBlackboard" as B
  participant "StrategyCreate" as S
  participant "GenericPipeline" as L
  participant "RepositoryWithDriver" as R
  participant "StrategyWrite" as W
  participant "GenericDriver" as D
  activate P
  P -> B : create(caller, **kwargs)
  activate B
  B -> S : create(caller=blackboard, processor=processor, blackboard=blackboard, **kwargs)
  activate S
  S --> B : DocumentFacade
  deactivate S
  B --> P : facade
  deactivate B
  P -> L : process(processor, facade)
  activate L
  L -> B : write(pipeline, facade)
  activate B
  deactivate B
  deactivate L
  opt flush_per_cycle
    P -> B : flush(caller, **write_context)
    activate B
    loop each repository
      B -> R : write(caller, facade, **write_context)
      activate R
      R -> W : write(caller=repository, facade, repository=repository, driver=driver)
      activate W
      W -> D : write(payload, filename, suffix)
      activate D
      deactivate D
      W --> R : True
      deactivate W
      R --> B : bool
      deactivate R
    end
    B --> P : bool
    deactivate B
  end
  deactivate P
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| nema registriranih repozitorija | `BlackboardException` | pipeline ne može pisati |
| `defer_flush=False` uz *N* pipelinea | repozitoriji se pišu **nakon svakog** pipelinea | *N* upisa po dokumentu; namjerno za audit/debug |
| `flush_per_cycle=False` | platno se drži do kraja prolaza | jedan završni flush nakon zadnje stavke |
| write strategija ne prepoznaje predmet | `warning`, `return False` | brojač se ne miče; ostali repozitoriji rade; ishod flusha je `False` (`SmallBlackboard`) |
| kvar u bilo kojem repozitoriju | `RepositoryException` → `BlackboardException` | **cijeli flush pada**; preostali repozitoriji se ne obilaze (odjeljak 15 t.2) |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Document flow

start
:driver built and registered (PENDING);
:processor discovers and filters a unit;
:blackboard.create -> strategy returns DocumentFacade;
repeat
:pipeline.process -> transform;
:blackboard.write(pipeline, facade);
repeat while (more pipelines?) is (yes)
if (no repositories registered?) then (yes)
:BlackboardException;
stop
else (no)
endif
if (flush_per_cycle?) then (yes)
:blackboard.flush(caller, **write_context);
else (no)
:canvas is held; one flush after the last item;
endif
repeat
:repository.write -> _strategy_write.write -> driver.write;
if (strategy recognises the subject?) then (no)
  :warning; False; _write_counter unchanged;
else (yes)
  if (repository fault?) then (yes)
    :RepositoryException -> BlackboardException; flush fails;
    stop
  else (no)
    :True; _write_counter += 1;
  endif
endif
repeat while (more repositories?) is (yes)
:canvas empty; flush outcome;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | primitivi su rezervirani; tok je mjesto gdje se njihova podjela vidi ili krši |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | petlja, granice kvara i audit žive u generičkim klasama, ne po specijalizacijama |
| `NFRQ-OBS-01/03` | **procesor** prijavljuje i stavku i prolaz (`2 + N` zapisa); pipeline je na `DEBUG`. Što od koraka lanca doista prijavljuje — vidi mjerenje u [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) §4 |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | procesor ne poznaje spremišta osim kroz platno; strategija ne poznaje putanju osim kroz driver |

## 12. Results

Dokument je nastao jednom, obogaćen onoliko puta koliko ima pipelinea, i pohranjen u onoliko
odredišta koliko je registrirano repozitorija — svako preko vlastite write strategije i vlastitog
drivera. Nijedan sudionik nije obavio tuđi korak.

Stanja kroz koja pritom prolazi: `[opisan] → [obogaćen] → [pohranjen]`.

## 13. Acceptance Criteria

1. Procesor ne otvara, ne imenuje i ne premješta izvor; briše ga jedino `FileDocumentProcessor`, uz `delete_source` i tek kad je `flush_outcome` `True`. ✅
2. Create strategija ne transformira sadržaj; gradi dokument i žigoše ishodište. ✅
3. Blackboard predaje **sebe** kao `caller`, procesor kao `processor=`. ✅
4. Transformacija se događa isključivo u `transform`. ✅
5. `flush` obilazi **svaki** registrirani repozitorij. ✅
6. Repozitorij predaje **svoj** driver strategiji kroz `_strategy_context()`. ✅
7. Driver je instanciran jednom i registriran kod `DriverManager`-a. ✅
8. Write strategija imenuje izlaz pri upisu i žigoše metapodatke pohrane. ⚠ — ključ žiga nije jedinstven (odjeljak 15 t.4)
9. Pipeline dohvaća driver kroz `processor` koji prima u `process`. ✅
10. Create strategija dohvaća driver kroz `processor` koji joj blackboard predaje. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | čitanje `processors/file.py` | `create_generator` predaje samo `filename`; `unlink` je u `_remove_source`, iza provjere `flush_outcome is True` |
| 2 | čitanje `strategies/documents/pdf.py` | `CreatePdfDocument` gradi `FileDocument` i žigoše metapodatke; ne otvara PDF |
| 3 | čitanje `blackboards/small.py:127` | `create(caller=self, processor=caller, blackboard=self, **kwargs)` |
| 5 | čitanje `blackboards/small.py:165` | `for repository in self._repositories: repository.write(...)` |
| 6 | čitanje `concrete/repository.py:277` | `_strategy_context() -> {"driver": self.driver}` |
| 7 | čitanje `concrete/workflow.py:_build_drivers` | jedna instanca po imenu, upisana u `DriverManager` |
| 9 | `command grep -rn "processor\.driver" src/wattleflow/pipelines/` | `pipelines/nlp/entities.py:85` — `processor.driver.read(table="entitet")`; mehanizam je u upotrebi |
| 10 | `command grep -n "processor=caller" src/wattleflow/blackboards/*.py` | sva četiri blackboarda predaju `processor=caller` create strategiji |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda, `command grep`; kriterij — odjeljak 13; platforma — radna stabla `workflow` i
`blackwattle`, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/{processor,pipeline,repository,blackboard,strategy,workflow}.py`, `blackboards/small.py`,
`processors/file.py`, `strategies/documents/*.py`.
**Slijepa pjega (D-11):** kriteriji 2, 4, 6 i 8–10 ostaju pregled koda; kriteriji 1, 3, 5 i uvjeti iz odjeljka 07 pokriveni su testovima.
Testovi (`workflow/tests`): `test_flow_preconditions.py` (provjere pri gradnji blackboarda i procesora, i pod `python -O`), `test_processor.py` (završni flush, kvar stavke → `FAILED`, `write_context`), `test_blackboard.py`, `test_repository.py` i `blackwattle/tests/processors/test_delete_source.py` (ishod flusha i brisanje izvora).

## 15. Open issues

1. **Grana „nema `strategy_create`" je nedostižna.** `GenericBlackboard.__init__` odbija konstrukciju bez `StrategyCreate` (`BlackboardException`), pa grana `if not self._strategy_create` u `create` (`warning` i `None`) postaje dostižna tek nakon `__del__`. Je li to obrana ili mrtav kod — nije odlučeno.
2. **Kvar jednog repozitorija ruši cijeli flush.** `for` petlja u `flush` (blackwattle `small`, `large`, `bundle`) nema po-repozitorijsku granicu, pa neuspjeh drugog odredišta poništava obilazak trećeg. Je li to željeno (atomarnost) ili nije (otpornost) — nije odlučeno.
3. **Kategorija ovog zapisa.** Predmet je suradnja više primitiva, a os kategorija daje po jednu kategoriju po primitivu. `PRC` je odabran jer procesor posjeduje prolaz i oba toka počinju u njemu. Alternativa je nova kategorija za tokove — ali ona ulazi uz dokumentiranu izmjenu (D-12), ne dopisivanjem. Do odluke oznaka je **provizorna**.
4. **Ključ žiga pohrane nije jedinstven.** Write strategije žigošu `output` (file, json, pdf, word, audio, records), `storage_filename` (text, dataframe, avro, orc, protobuf, image, graph i `WriteEmailCopyOLD`), `output_filename` (`WriteEmailCopy`) i `storage_uri` (opensearch, solr); ostale žigošu vlastite ključeve (`records_sent`, `opensearch_id`, `pgvector_id`, `annotations_filename`). Tko god želi strojno provjeriti *je li dokument pohranjen* mora poznavati sve. Razrješenje traži dokumentiranu izmjenu (D-12): proširiti popis ili uvesti kanonski ključ i uskladiti strategije.

## 16. References

- Izvedba: `concrete/processor.py`, `concrete/pipeline.py`, `concrete/repository.py`, `concrete/blackboard.py`, `concrete/strategy.py` (workflow) · `blackboards/small.py`, `processors/file.py`, `strategies/documents/*.py` (blackwattle)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-04 | Dijagrami (klasa, kontekst, slučajevi uporabe, slijed, tok) izrađeni i renderirani; zatvorene stavke bez dijagrama i `write_context`. Preduvjeti blackboarda i procesora provode se izričitim iznimkama (`BlackboardException`, `ProcessorException`) umjesto `assert`; završni flush uz `flush_per_cycle=False` proveden i testiran (`test_flow_preconditions.py`, `test_processor.py`). |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: poruke create/write prema potpisima iz koda (caller, processor, blackboard, repository, driver), `_strategy_write`, `_write_counter`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Prijedlog. Oba toka **provedena su u kodu** |
