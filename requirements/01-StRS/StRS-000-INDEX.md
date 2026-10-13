<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS — stakeholder requirements index

| | |
|---|---|
| **Identifier** | `StRS-<NAME>` (the file name without `.md`); the component name is English, in capitals |
| **Class** | **in use but not in the register** — introducing it requires a documented change ([D-12](../../DOCTRINE.md#d-12)) |
| **Standard** | ISO/IEC/IEEE 29148:2018 — Stakeholder Requirements Specification (StRS); the system requirements (SyRS) are its children |
| **Parents** | [PHILOSOPHY](../../PHILOSOPHY.md) · [METHODOLOGY](../../METHODOLOGY.md) §4 *Requirement ontology* · [DOCTRINE](../../DOCTRINE.md) |
| **Sibling registers** | [SyRS](../02-SyRS/SyRS-000-INDEX.md) · [NFRQ](../03-NFRQ/NFRQ-000-INDEX.md) |

This document is an **index**, not a register: each component lives in its own file.

**What a StRS is.** A stakeholder requirements specification carries the **narrative, the actors, the
use cases, the business rules** (`BR-<abbreviation>-<nn>`) and the **error tracking** of one component, in plain
language for a wide audience. It does not describe how the component is built: that belongs to the
system requirements (SyRS).

**Where the framework sits.** `core/` holds the abstract contracts, `concrete/` (the subject of the entries below)
holds their generic implementations, and the specialisations built on them (for example in `blackwattle`) are outside
these requirements. The dependency runs one way, from the specialisation towards `concrete/` and from `concrete/` towards `core/`
([`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md)).

## Register

Each entry has a use case diagram, a context diagram, a flow chart and, where the component has a state machine, a state diagram.

| StRS | Component | Classes | Module (`concrete/`) | Children (SyRS) |
|---|---|---|---|---|
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard: temporary storage of documents during processing | `GenericBlackboard` | `blackboard.py` | [`SyRS-BLC-blackboard`](../02-SyRS/SyRS-BLC-blackboard.md) |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | Connection: the single boundary towards an external system | `GenericConnection`, `ConnectionObserverInterface` | `connection.py` | [`SyRS-CNC-connection`](../02-SyRS/SyRS-CNC-connection.md) |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | Document: the unit of data travelling through the flow | `Document`, `DocumentAdapter`, `DocumentFacade` | `document.py` | [`SyRS-DCM-document`](../02-SyRS/SyRS-DCM-document.md) |
| [`StRS-DRIVER`](StRS-DRIVER.md) | Driver: uniform access to external resources | `GenericDriver`, `LazyDriverProxy` | `driver.py` | [`SyRS-DRV-driver`](../02-SyRS/SyRS-DRV-driver.md) |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow factory: assembling a flow from configuration | `WorkflowFactory` | `factory.py` | [`SyRS-WRK-workflow`](../02-SyRS/SyRS-WRK-workflow.md) |
| [`StRS-MANAGER`](StRS-MANAGER.md) | Managers: registries of connections, drivers and processors by name | `ConnectionManager`, `DriverManager`, `ProcessorManager` | `manager.py` | [`SyRS-MNG-managers`](../02-SyRS/SyRS-MNG-managers.md) |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | Memento: an immutable snapshot of a component state | `GenericMemento` | `memento.py` | [`SyRS-MMT-memento`](../02-SyRS/SyRS-MMT-memento.md) |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | Memento store: keeping snapshots between runs | `MementoStore`, `MemoryMementoStore`, `FileMementoStore` | `memento_store.py` | [`SyRS-MMT-memento`](../02-SyRS/SyRS-MMT-memento.md) |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | Pipeline: one transformation of one document | `GenericPipeline` | `pipeline.py` | [`SyRS-PPL-pipeline`](../02-SyRS/SyRS-PPL-pipeline.md) |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | Processor: runs records through the pipelines | `GenericProcessor` | `processor.py` | [`SyRS-PRC-processor`](../02-SyRS/SyRS-PRC-processor.md) |
| [`StRS-REPOSITORY`](StRS-REPOSITORY.md) | Repository: permanent storage of documents through strategies | `GenericRepository`, `RepositoryWithDriver` | `repository.py` | [`SyRS-RPS-repository`](../02-SyRS/SyRS-RPS-repository.md) |
| [`StRS-SERIALISATION`](StRS-SERIALISATION.md) | Format boundary: parser, formatter and converter | `GenericParser`, `GenericFormatter`, `GenericConverter`, `ConversionStrategy` | `serialisation.py` | [`SyRS-SRL-PAR-parser`](../02-SyRS/SyRS-SRL-PAR-parser.md) · [`SyRS-SRL-FMT-formatter`](../02-SyRS/SyRS-SRL-FMT-formatter.md) · [`SyRS-SRL-CNV-converter`](../02-SyRS/SyRS-SRL-CNV-converter.md) |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | State machine: permitted transitions between states | `StateMachine`, `GuardedStateMachine` | `state_machine.py` | [`SyRS-SMC-state-machine`](../02-SyRS/SyRS-SMC-state-machine.md) |
| [`StRS-STRATEGY`](StRS-STRATEGY.md) | Strategy: a replaceable way of doing one job | `Strategy`, `StrategyCreate`, `StrategyGenerate`, `StrategyRead`, `StrategyWrite` | `strategy.py` | [`SyRS-STR-strategy`](../02-SyRS/SyRS-STR-strategy.md) |
| [`StRS-WORKFLOW`](StRS-WORKFLOW.md) | Workflow: the assembled flow that runs the processors | `Workflow` | `workflow.py` | [`SyRS-WRK-workflow`](../02-SyRS/SyRS-WRK-workflow.md) |

### Components without a StRS yet

Their requirements are carried by the SyRS alone until a StRS is written.

| Component | Classes | Module | SyRS |
|---|---|---|---|
| Root base class | `Wattleflow` | `concrete/base.py` | [`SyRS-PTR-root-base`](../02-SyRS/SyRS-PTR-root-base.md) |
| Helpers | `Attribute`, `NameHelper` | `concrete/helper.py` | [`SyRS-HLP-helpers`](../02-SyRS/SyRS-HLP-helpers.md) |
| Lazy iterators | `LazyIterator`, `LazyAsyncIterator`, `ThreadSafeLazyIterator` | `concrete/iterator.py` | [`SyRS-ITR-lazy-iterators`](../02-SyRS/SyRS-ITR-lazy-iterators.md) |
| Observable | `ThreadSafeObservable` | `concrete/observable.py` | [`SyRS-OBS-observable`](../02-SyRS/SyRS-OBS-observable.md) |
| Orchestrator | `Orchestrator` | `concrete/orchestrator.py` | [`SyRS-ORC-orchestrator`](../02-SyRS/SyRS-ORC-orchestrator.md) |
| Scheduler | `Scheduler` | `concrete/scheduler.py` | [`SyRS-SCH-scheduler`](../02-SyRS/SyRS-SCH-scheduler.md) |
| Singleton | `Singleton` | `concrete/singleton.py` | [`SyRS-SGT-singleton`](../02-SyRS/SyRS-SGT-singleton.md) |
| Moment (time) | — | `helpers/moment` | — |

## Cross-cutting rules (`BR-PTN`)

These rules apply to every component of the generic layer and have no StRS of their own. They are cited by the SyRS and NFRQ entries, so they are kept here.
The component rules of that requirement (`BR-WFL`, `BR-BBD`, `BR-PRC`, `BR-DRV`) are superseded by the business rules of the
StRS entries above.

| oznaka | pravilo |
|---|---|
| **BR-PTR-01** | Svaki objekt frameworka nosi identitet izveden iz vlastitog tipa; identitet se ne postavlja izvana i ne mijenja nakon konstrukcije. |
| **BR-PTR-02** | Auditabilnost nije opcija. Nasljeđuje se na jednom deklariranom mjestu, pa je svaki potomak nosi po ugovoru, ne kao nuspojavu druge baze. |
| **BR-PTR-03** | Prijelaz stanja koji automat ne dopušta ne izvodi se. Stanje se mijenja isključivo primjenom akcije nad tablicom prijelaza ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md), `BR-SMC-03`). |
| **BR-PTR-04** | Tajna se ne zapisuje. Vrijednost kredencijala nikad ne ulazi u audit zapis; zapisuje se samo *je li* autentikacija postignuta ([`NFRQ-SCR-06`](../03-NFRQ/NFRQ-SCR-06-audit-confidentiality.md)). |
| **BR-PTR-05** | Iznimka jednog sloja ne prolazi kroz drugi nepromijenjena — svaki sloj je omata u vlastiti razred, s uzrokom (`raise … from e`), da trag kaže **gdje** je puklo. |
| **BR-PTR-06** | *Prijedlog.* **Usporedba i granica definiraju se u dokumentaciji prije nego što uđu u kod.** Svaka vrijednost koja se uspoređuje, ograničava, skraćuje, zaokružuje ili za koju se alocira prostor deklarira vrstu, jedinicu, preciznost, uključivost granice, referencu (zona, kodiranje), ponašanje izvan granice i značenje odsutnosti. Usporedive su samo vrijednosti iste vrste — vrijeme s pomakom i vrijeme bez pomaka nikad se ne uspoređuju, a vrijednost druge vrste odbija se **gdje ulazi**, ne gdje bi usporedba pukla. Nedeklarirana granica je kvar dokumentacije, ne izbor implementacije. Definicije i zadane vrijednosti: [`NFRQ-DFN-03`](../03-NFRQ/NFRQ-DFN-03-comparison-and-boundary-values.md). |
| **BR-PTR-07** | Klasa koja nasljeđuje `Wattleflow` deklarira `__slots__` (prazan `()` ako ne drži stanje) i ne ponavlja slotove baze. Klasa bez `__slots__` nosi `__dict__`; to je dopušteno samo uz opravdanje zapisano u njezinu SyRS-u ([`NFRQ-MMR-01`](../03-NFRQ/NFRQ-MMR-01-memory.md)). |

## Traceability to the NFRs

The first two rows are the cross-cutting groups that were carried by the former generic-layer and moment requirements. The remaining rows are taken from the StRS entries themselves.

| StRS | Key NFRs |
|---|---|
| generic layer (all components) | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) · [`NFRQ-SCR-01`](../03-NFRQ/NFRQ-SCR-01-blast-radius.md) · [`NFRQ-SCR-02`](../03-NFRQ/NFRQ-SCR-02-attack-surface.md) · [`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md) · [`NFRQ-SCR-06`](../03-NFRQ/NFRQ-SCR-06-audit-confidentiality.md) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md)…[`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) · [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) · [`NFRQ-DFN-03`](../03-NFRQ/NFRQ-DFN-03-comparison-and-boundary-values.md) · [`NFRQ-MMR-01`](../03-NFRQ/NFRQ-MMR-01-memory.md) |
| moment (time) | [`NFRQ-DFN-03`](../03-NFRQ/NFRQ-DFN-03-comparison-and-boundary-values.md) · [`NFRQ-DFN-04`](../03-NFRQ/NFRQ-DFN-04-moment-aware-helper.md) · [`NFRQ-DFN-05`](../03-NFRQ/NFRQ-DFN-05-moment-naive-helper.md) · [`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md) · [`NFRQ-ORG-09`](../03-NFRQ/NFRQ-ORG-09-external-standard.md) |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | [`NFRQ-DFN-04`](../03-NFRQ/NFRQ-DFN-04-moment-aware-helper.md) · [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) |
| [`StRS-PIPELINE`](StRS-PIPELINE.md), [`StRS-REPOSITORY`](StRS-REPOSITORY.md), [`StRS-STRATEGY`](StRS-STRATEGY.md) | [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | [`NFRQ-SCR-06`](../03-NFRQ/NFRQ-SCR-06-audit-confidentiality.md) |

## Open

1. The StRS entries do not yet link every NFR that applies to them.
2. The `BR-PTN` rules keep their numbers so that the existing references stay valid; whether they should become a StRS of their own (for example for the root base class) is undecided.
