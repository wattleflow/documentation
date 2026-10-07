<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-WFL — Generički workflow i tvornica

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericWorkflow(Wattleflow, IOriginator, ABC)`, `WorkflowFactory`, `WorkflowFactoryLogger`, `WorkflowFactoryException` |
| **Sestrinski** | [`FRQ-PRC`](FRQ-PRC-processor.md) (tvornica ih gradi i uvezuje) · [`FRQ-MGR`](FRQ-MGR-managers.md) (tri upravitelja) |
| **Izvedba** | `workflow/src/wattleflow/concrete/workflow.py` (789 linija) |
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

Workflow je **cjelina**, a tvornica je mjesto na kojem `BR-WFL-01` postaje istinit ili ne postaje:
zamjena bilo kojeg primitiva je izmjena konfiguracije samo ako postoji jedno mjesto koje ime iz
konfiguracije prevodi u klasu. To mjesto je `WorkflowFactory.resolve`.

Podjela je namjerno oštra:

| klasa | što je | zašto tako |
|---|---|---|
| `GenericWorkflow` | **objekt frameworka** — `Wattleflow` + `IOriginator` | drži tri menadžera i (po želji) monitor; `run()` je jedan mjeren prolaz, `execute()` piše specijalizacija; `save_state`/`restore_state` imaju zadanu izvedbu (snimka svakog procesora po registriranom imenu), pa specijalizacija ne treba vlastiti memento |
| `WorkflowFactory` | **obična klasa**, ne `Wattleflow` | tvornica nije sudionik toka; gradi ga izvana |
| `WorkflowFactoryLogger` | `Wattleflow` bez tijela | tvornica ipak treba auditirati, pa posuđuje objekt koji to zna |

Tvornica gradi **odozdo prema gore**, redoslijedom ovisnosti: konekcije → driveri (koji dobivaju
menadžer konekcija) → procesori (koji dobivaju drivere) → i unutar procesora: pipelinei →
blackboard sa `strategy_create` → spremišta sa `strategy_write` i driverom.

**Konfiguracija se čita relativno.** `sections` je ključna putanja okolišnog bloka (npr.
`("infrastructure", "dev")`); svaki dohvat ide pod njom, pa sam adapter ostaje neopsežen i
jednostavan.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title WorkflowFactory, GenericWorkflow

top to bottom direction

interface IOriginator
interface IConfig {
  + find(*keys, default)
}
class Wattleflow

abstract class GenericWorkflow {
  # _connections : ConnectionManager
  # _drivers : DriverManager
  # _processors : ProcessorManager
  # _monitor : Monitor | None
  + connections : ConnectionManager
  + drivers : DriverManager
  + processors : ProcessorManager
  + monitor : Monitor | None
  + attach_monitor(monitor)
  + run(**kwargs)
  + measured_paths() : list[str]
  + audit_drivers()
  + save_state() : GenericMemento
  + restore_state(memento)
  + {abstract} execute()
}

class WorkflowFactory {
  - {static} _registry : dict[str, type]
  # {static} STRUCTURAL : frozenset
  + {static} register(name, class_name)
  + {static} resolve(type_name) : type
  + {static} build(adapter, sections, **kwargs) : GenericWorkflow
}

class WorkflowFactoryLogger
class WorkflowFactoryException
class WorkflowException

IOriginator <|.. GenericWorkflow
Wattleflow <|-- GenericWorkflow
Wattleflow <|-- WorkflowFactoryLogger
WorkflowFactory ..> GenericWorkflow : builds
WorkflowFactory ..> IConfig : reads
WorkflowFactory ..> WorkflowFactoryLogger : logs
WorkflowFactory ..> WorkflowFactoryException : raises
GenericWorkflow ..> WorkflowException : raises
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title WorkflowFactory, GenericWorkflow

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "WorkflowFactory, GenericWorkflow", "Builds the whole from configuration by name")
Person(pz, "Caller", "Script or notebook")
System_Ext(ca, "IConfig", "YAML or JSON source")
System_Ext(up, "ConnectionManager, DriverManager, ProcessorManager", "Connections, drivers, processors")
Rel_L(pz, sys, "Requests build and run")
Rel_R(sys, ca, "Reads configuration (find)")
Rel_U(sys, up, "Registers built primitives")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Pozivatelj (skripta, notebook) | `adapter`, `sections`, opcionalno `workflow_name` |
| **A2** | `IConfig` adapter — YAML/JSON izvor | `find(*path, default=…)` |
| **A3** | `WorkflowFactory` — predmet ovog zahtjeva | registar imena → klasa |
| **A4** | `ConnectionManager` / `DriverManager` / `ProcessorManager` | registri izgrađenih primitiva |
| **A5** | `GenericWorkflow` — izgrađena cjelina | `execute()` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | klasa se registrira → `register(name, class)` |
| **EV02** | A1 traži gradnju → `build(adapter=…, sections=…)` |
| **EV03** | ime iz konfiguracije se razrješava → `resolve(type_name)` |
| **EV04** | `runtime:` blok se primjenjuje → `_apply_runtime_env` |
| **EV05** | A1 pokreće → `workflow.run()` (bez monitora to je `execute()`) |
| **EV06** | ime se ne razriješi → `WorkflowFactoryException` |
| **EV07** | gradnja je gotova → `_configure_monitor` (`monitoring:` blok) |
| **EV08** | `memento:` blok na unosu workflowa → `_build_memento`; svaki procesor dobiva pohranu, registrirano ime kao ključ i `checkpoint_every` ([`FRQ-MEM`](FRQ-MEM-memento.md)) |
| **EV09** | A1 snima ili vraća workflow → `save_state()` / `restore_state(memento)` nad procesorima po imenu |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title WorkflowFactory, GenericWorkflow

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Caller" as A1
  actor "IConfig" as A2
  actor "ConnectionManager, DriverManager, ProcessorManager" as A4
  actor "GenericWorkflow" as A5
    usecase "Register class" as EV01
    usecase "Build from configuration" as EV02
    usecase "Resolve name to class" as EV03
    usecase "Apply runtime block" as EV04
    usecase "Run workflow" as EV05
    usecase "Report unresolved name" as EV06
    usecase "Configure monitor" as EV07
  A1 --> EV01
  A1 --> EV02
  A2 --> EV02
  A4 --> EV02
  A2 --> EV03
  A2 --> EV04
  A1 --> EV05
  A5 --> EV05
  A1 --> EV06
  A2 --> EV07
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Svaka klasa koja se smije pojaviti u konfiguraciji je registrirana (`EV01`).
2. `adapter` implementira `IConfig`; `sections` nije `None`.
3. `workflows` blok postoji pod `sections`; ako ih je više, `workflow_name` bira jedan.
4. Svaki unos deklarira `type:` koje odgovara registriranom imenu (`BR-WFL-01`).

## 08. Sequence Diagrams

### Normalan tok

1. **EV02** — `adapter` i `sections` se provjere; nedostatak jednog je kvar prije ijedne gradnje
   (`BR-WFL-02`).
2. `workflows` se dohvati pod `sections`. Lista s jednim unosom uzima se bez imena; s više unosa
   traži se `workflow_name`.
3. **EV03** — klasa workflowa se razriješi kroz `_resolve_section`, koji kvar obogaćuje **sekcijom
   i imenom unosa** — poruka kaže *gdje* je u konfiguraciji problem, ne samo *koje* ime fali.
4. **EV04** — `runtime:` blok se preslikava u varijable okoline: poznati ključevi kroz
   `_RUNTIME_KEY_TO_ENV` (`tika_server_jar` → `TIKA_SERVER_JAR`, `java_home` → `JAVA_HOME`,
   `time_zone` → `WATTLEFLOW_TIME_ZONE`…), ostatak iz `runtime.env` doslovno. Nakon toga tvornica
   jednom postavlja zonu workflowa (`MomentHelper.configure`, [`FRQ-MMN`](FRQ-MMN-moment.md)).
5. Globalne postavke zapisa iz `logging:`: `handler` i `format` (kao `formatting`) čitaju se jednom i
   prosljeđuju svakom izgrađenom objektu; svaki unos ih smije nadjačati kroz `_audit`. `level` se
   ne prosljeđuje: postavlja razinu korijenskog loggera i loggera tvornice, a unos smije zadati
   vlastiti `level` (vrijedi za njegovu klasu).
6. Gradnja redom: `_build_connections` → `_build_drivers` → `_build_processors`. Driver koji
   deklarira `connection_name` dobiva i `connection_manager`, pa svoju konekciju razrješava pri
   učitavanju.
7. Procesori se grade zajedno sa svojim pipelineima, blackboardom (`strategy_create` dobiva audit
   postavke blackboarda) i spremištima. Driver procesora i spremišta stiže kao razriješena
   instanca po imenu, ne ime; spremište ga dobiva samo ako njegova klasa deklarira `driver` u
   `ALLOWED` (`_driver_context`), i tada je `configuration.driver` obvezan.
8. **EV07** — ako `monitoring:` postoji i `level` nije `OFF`, gradi se `Monitor` (uz
   `ResourceManager` i sinkove iz `exporters`) i veže na workflow kroz `attach_monitor`; inače
   monitora nema.
9. **EV05** — `run()` promatra audit zapise kroz monitor dok traje `execute()` i jednom izvještava;
   `execute()` je apstraktan: specijalizacija odlučuje što „pokreni" znači.

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title WorkflowFactory, GenericWorkflow

actor Caller
participant WorkflowFactory as Factory
participant IConfig as Config
participant "Managers" as Managers
participant GenericWorkflow as Workflow

Caller -> Factory : build(adapter, sections)
activate Factory
Factory -> Config : find(*sections, "workflows")
Factory -> Factory : resolve(type), runtime block, level, memento block
Factory -> Managers : build connections, drivers, processors
Factory -> Workflow : workflow_class(adapter, managers, ...)
opt monitoring set and level is not OFF
  Factory -> Workflow : attach_monitor(monitor)
end
Factory --> Caller : GenericWorkflow
deactivate Factory
Caller -> Workflow : run()
activate Workflow
Workflow -> Workflow : execute()
deactivate Workflow
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| klasa koja nije klasa se registrira (`register("X", 42)`) | `WorkflowFactoryException` | registar drži samo klase |
| ime se registrira drugom klasom | upozorenje s imenom i obje klase; nova klasa vrijedi | zamjena se vidi; ista klasa ponovno je tiha |
| `logging.level` nije zadan | razina tvornice slijedi razinu korijenskog loggera | `INFO` gradnje vidljiv koliko i ostatak procesa |
| unos nosi ključ koji tvornica ne troši (npr. `pattern` uz procesor umjesto u `configuration:`) | upozorenje s sekcijom, imenom i ključevima | ključ ne nestaje nezapaženo; samo blackboard spaja inline ključeve |
| `runtime.env` nosi vrijednost | upisuje se u `os.environ`, u zapis ide samo ime (`value="<set>"`) | token ne dospijeva u audit trag |
| `adapter` ne implementira `IConfig` | `WorkflowFactoryException` | gradnja ne počinje |
| `sections` je `None` | `WorkflowFactoryException` | konfiguracija bez opsega se odbija |
| workflow nije nađen po imenu | `WorkflowFactoryException` s imenom | — |
| `type:` nedostaje | poruka nabraja **koje** vrste unosa ga moraju imati | kvar konfiguracije je čitljiv |
| `type:` nije registriran | poruka nudi **do tri bliska imena** (`difflib`), popis prvih 20 registriranih i broj svih | tipfeler se ispravlja bez čitanja koda |
| kvar u pod-unosu | `_resolve_section` dodaje sekciju i `name` unosa, pa ponovno diže `from e` | uzrok očuvan (`BR-PTN-05`) |
| nema procesora u workflowu | `WorkflowFactoryException` | workflow bez posla se ne gradi |
| unos procesora nije rječnik | `WorkflowFactoryException` | — |
| poznati `runtime` ključ je `None` ili prazan niz, ili je vrijednost u `runtime.env` `None` | preskače se | prazna postavka ne briše okolinu |
| spremištu ili procesoru treba driver koji nedostaje u `configuration.driver` ili nije registriran | `WorkflowFactoryException` sa sekcijom i imenom unosa (isti oblik za oba puta) | — |
| `monitoring:` ima nepoznat ključ | upozorenje, ključ se odbacuje | — |
| izvoznik u `monitoring.exporters` nema `sink:` | `WorkflowFactoryException` | — |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title WorkflowFactory, GenericWorkflow

start
:build(adapter, sections);
if (adapter is IConfig and sections set?) then (no)
  :<b><color:red>FAILED: WorkflowFactoryException</color></b>;
  kill
endif
if (workflow found and its type registered?) then (no)
  :<b><color:red>FAILED: WorkflowFactoryException, close names suggested</color></b>;
  kill
endif
:report inline keys that no builder consumes (warning);
:runtime block into environment (extra values are not traced);
if (logging.level declared?) then (yes)
  :set the factory level to it;
else (no)
  :follow the root logger level;
endif
:build connections, drivers, processors;
if (a driver of a processor or repository missing?) then (yes)
  :<b><color:red>FAILED: WorkflowFactoryException with section and entry</color></b>;
  kill
endif
:memento block gives every processor a store and its name as key;
:workflow_class(adapter, managers, ...);
if (monitoring set and level is not OFF?) then (yes)
  :Monitor + attach_monitor;
endif
:return GenericWorkflow;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Workflow` je rezervirani primitiv; tvornica je pattern oko njega, ne novi primitiv |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | registar je jedina točka kroz koju ime iz konfiguracije postaje izvršni kod — jedno mjesto za nadzor |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `STRUCTURAL` razdvaja ključeve koje tvornica troši od onih koje prosljeđuje; `PresetGate.resolve` iz `ALLOWED` klase čita prima li ona `driver` |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; `_RUNTIME_KEY_TO_ENV` **imenuje** third-party alate (Tika, Spark) ali ih ne uvozi |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | `runtime:` vrijednosti se zapisuju doslovno — vidi odjeljak 15 t.4 |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | jedan `INFO` po gradnji (broj konekcija, drivera, procesora) — koji se ne vidi dok konfiguracija ne deklarira `logging.level` (odjeljak 15 t.1) |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | `_audit` i `_resolve_section` postoje upravo zato da se obrasci ne prepisuju po vrstama unosa |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Iz jedne konfiguracijske datoteke nastaje uvezana cjelina: konekcije, driveri i procesori s
pipelineima, platnima i spremištima — svaki razriješen po imenu. Zamjena bilo kojeg dijela je
izmjena `type:` u konfiguraciji, bez dodirivanja koda (`BR-WFL-01`), a kvar konfiguracije zaustavlja
gradnju prije obrade (`BR-WFL-02`).

## 13. Acceptance Criteria

1. Svaki primitiv se razrješava po imenu iz jednog registra. ✅
2. Nerazriješeno ime zaustavlja gradnju i poruka imenuje sekciju, unos i bliske kandidate. ✅
3. Gradnja ide redoslijedom ovisnosti; driver dobiva menadžer konekcija kad ga treba. ✅
4. Postavke zapisa (`handler`, `format`) prosljeđuju se svakom objektu, `level` se nasljeđuje s korijenskog loggera; sve se mogu nadjačati po unosu. ✅
5. Tvornica nije objekt frameworka, ali auditira kroz namjenski objekt. ✅
6. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
7. Bez `logging.level` zapisi tvornice slijede razinu korijenskog loggera (nisu skriveni iza `ERROR`). ✅
8. Inline ključ koji tvornica ne troši ne nestaje nezapaženo: prijavljuje se sa sekcijom, unosom i ključevima. ✅
9. `__slots__` ne ponavlja slotove baze. ✅ — [`FRQ-PTN`](FRQ-PTN-root-base.md) kriterij 7
10. Vrijednost iz `runtime.env` ne ulazi u zapis ([`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md)); poznati ključevi (instalacijske putanje) ulaze. ✅
11. Klasa workflowa bez vlastitog memento koda može se izgraditi; zadani `save_state`/`restore_state` snimaju i vraćaju procesore po imenu i odbijaju nevaljanu snimku bez izmjene. ✅
12. `memento:` blok gradi pohranu (`memory`, `file`, registrirani razred) i predaje je procesorima; neispravan blok ili ime procesora koje nije obično ime je pogreška konfiguracije. ✅
13. Registar drži samo klase; zamjena imena drugom klasom je vidljiva. ✅
14. Nepoznat driver procesora izlazi kao `WorkflowFactoryException` s sekcijom i unosom, kao i za spremište. ✅
15. `runtime.time_zone` postaje `WATTLEFLOW_TIME_ZONE`, a zona workflowa razrješava se jednom nakon bloka `runtime:`; bez ključa vrijedi globalna postavka ili zona sustava (`BR-MMN-12`).

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | pregled `_registry` i `resolve` | jedan `dict[str, type]`, jedan ulaz |
| 2 | pregled `resolve` / `_resolve_section` | `difflib.get_close_matches(…, n=3, cutoff=0.6)`; poruka nosi `section` i `name` |
| 3 | pregled `build` | tri poziva redom; `configuration.setdefault("connection_manager", connections)` |
| 4 | pregled `build` i `_audit` | `handler`/`formatting`: `config.get(k, nested.get(k, default[k]))`; `level` samo ako ga unos zada; razina iz `logging.level` ide na `getLogger().setLevel` |
| 5 | pregled | `class WorkflowFactory:` bez baze; `logger = WorkflowFactoryLogger(level="ERROR", …)` |
| 6 | pregled modula | `__all__` = 4 imena; uvozi `os`, `difflib`, `abc`, `logging`, `typing` + `wattleflow.*` |
| 7 | `workflow/tests/test_workflow_factory.py` (`LevelTest`) | korijen `INFO` → zapis gradnje; korijen `WARNING` → tišina; deklarirana razina pobjeđuje (hvata se rukovateljem na loggeru, bez promjene njegove razine) |
| 8 | `InlineKeysTest` | nepoznat ključ uz driver i procesor prijavljen sa sekcijom; strukturni ključevi i ključevi unutar `configuration:` ne |
| 9 | `workflow/tests/test_wattleflow_base.py` (`StructureTest`) | nijedna klasa sloja ne ponavlja slot baze |
| 10 | `RuntimeEnvTest` | poznati ključ upisan i praćen putanjom; `API_TOKEN` upisan u okolinu, ime u zapisu, vrijednost nikad |
| 13 | `RegistrationTest` | `42`, tekst, `None` i instanca odbijeni; ista klasa tiha; druga klasa upozorenje; bliska imena u poruci |
| 14 | `DriverLookupTest` | poruka nosi ime drivera i unosa; registrirani driver stiže do procesora |
| 15 | `RuntimeEnvTest.test_time_zone_sets_the_workflow_zone_once` | `runtime.time_zone` upisan u `WATTLEFLOW_TIME_ZONE`; `now()` nosi tu zonu |
| 11–12 | `workflow/tests/test_workflow_memento.py` (13 testova) | zadani memento i tvornica; probni prolaz kroz izmijenjen primjer `blackwattle/examples/workflows/05_markdown` ([`FRQ-MEM`](FRQ-MEM-memento.md) §14) |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`. Mutacije (svaka ruši test): bilo koji objekt se registrira, tiha zamjena, razina se ne prati, vrijednost `runtime.env` u zapisu, nepoznati ključevi neprijavljeni, `ManagerException` nepreslikan.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-WFL-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

Zapisana svojstva (ne defekti): upis `runtime:` u `os.environ` je **procesni** nuspojava, pa dva workflowa u istom procesu dijele okolinu; `GenericWorkflow` drži tri upravitelja, a monitor je neobavezan.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/workflow.py`
- Testovi: `workflow/tests/test_wattleflow_base.py`, `workflow/tests/test_workflow_factory.py`, `workflow/tests/test_workflow_memento.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-07 | `runtime.time_zone` (→ `WATTLEFLOW_TIME_ZONE`) i jedno postavljanje zone workflowa pri gradnji; kriterij 15 (`HLRQ-MMN` `BR-MMN-12`). |
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; class, sequence i flow chart prema kodu (memento, razina, prijava ključeva). |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno u cjelini (`test_workflow_factory.py`, 16 testova), svi defekti zatvoreni: `DEF-WFL-01` bez `logging.level` tvornica slijedi razinu korijenskog loggera (prije je `INFO` gradnje bio nevidljiv); `-02` inline ključ koji se ne troši prijavljuje se sa sekcijom i unosom (spajanje inline ključeva ostaje samo za blackboard); `-03` slotovi (ranije); `-04` vrijednosti `runtime.env` ne ulaze u zapis (`value="<set>"`), poznati ključevi (putanje) ulaze; upis u `os.environ` zapisan kao procesno svojstvo; `-05` nepoznat driver procesora je `WorkflowFactoryException` sa sekcijom i unosom (kao za spremište); `-06` mrtva `_strategy_defaults` obrisana; `-07` `fmt` (ranije); `-08` tipfeler u komentaru. **Novo:** `register` je prihvaćao bilo koji objekt i tiho prepisivao ime drugom klasom — sada klasa je obvezna, a zamjena se prijavljuje. Kriteriji 7–8, 10, 13–14, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-04 | Zadani `save_state`/`restore_state` u `GenericWorkflow` i `memento:` blok u tvornici (`_build_memento`, `WorkflowException`); EV08–EV09, kriteriji 11–12 ([`FRQ-MEM`](FRQ-MEM-memento.md)). |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: AuditException premještena u paket wattleflow.concrete; potpisi register, attach_monitor, run; poziv find(*sections, "workflows", default=None). |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |
