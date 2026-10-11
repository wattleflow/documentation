<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS — stakeholder requirements index

| | |
|---|---|
| **Identifier** | `StRS-<NAME>` (the file name without `.md`); the component name is English, in capitals |
| **Class** | **in use but not in the register** — introducing it requires a documented change (D-12) |
| **Standard** | ISO/IEC/IEEE 29148:2018 — Stakeholder Requirements Specification (StRS); the system requirements (SyRS) are its children |
| **Parents** | [PHILOSOPHY](../../PHILOSOPHY.md) · [METHODOLOGY](../../METHODOLOGY.md) §4 *Requirement ontology* · [DOCTRINE](../../DOCTRINE.md) |
| **Sibling registers** | [FRQ](../02-FRQ/FRQ-000-INDEX.md) · [NFRQ](../03-NFRQ/NFRQ-000-INDEX.md) |
| **Language** | Index EN; the entries are Croatian (source, `DOCUMENTATION.md` §3.2) |

This document is an **index**, not a register: each component lives in its own file.

**What a StRS is.** A stakeholder requirements specification carries the **narrative, the actors, the
use cases, the business rules** (`BR-<abbreviation>-<nn>`) and the **error tracking** of one component, in plain
language for a wide audience. It does not describe how the component is built: that belongs to the
system requirements (SyRS). The older functional records (FRQ) listed below are being aligned with the StRS.

**Where the framework sits.** `core/` holds the abstract contracts, `concrete/` (the subject of the entries below)
holds their generic implementations, and the specialisations built on them (for example in `blackwattle`) are outside
these requirements. The dependency runs one way, from the specialisation towards `concrete/` and from `concrete/` towards `core/`
([`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md)).

## Register

Each entry has a use case diagram, a context diagram, a flow chart and, where the component has a state machine, a state diagram.

| StRS | Component | Classes | Module (`concrete/`) | Children (FRQ) |
|---|---|---|---|---|
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard: temporary storage of documents during processing | `GenericBlackboard` | `blackboard.py` | [`FRQ-BBD`](../02-FRQ/FRQ-BBD-blackboard.md) |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | Connection: the single boundary towards an external system | `GenericConnection`, `ConnectionObserverInterface` | `connection.py` | [`FRQ-CON`](../02-FRQ/FRQ-CON-connection.md) |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | Document: the unit of data travelling through the flow | `Document`, `DocumentAdapter`, `DocumentFacade` | `document.py` | [`FRQ-DOC`](../02-FRQ/FRQ-DOC-document.md) |
| [`StRS-DRIVER`](StRS-DRIVER.md) | Driver: uniform access to external resources | `GenericDriver`, `LazyDriverProxy` | `driver.py` | [`FRQ-DRV`](../02-FRQ/FRQ-DRV-driver.md) |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow factory: assembling a flow from configuration | `WorkflowFactory` | `factory.py` | [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) |
| [`StRS-MANAGER`](StRS-MANAGER.md) | Managers: registries of connections, drivers and processors by name | `ConnectionManager`, `DriverManager`, `ProcessorManager` | `manager.py` | [`FRQ-MGR`](../02-FRQ/FRQ-MGR-managers.md) |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | Memento: an immutable snapshot of a component state | `GenericMemento` | `memento.py` | [`FRQ-MEM`](../02-FRQ/FRQ-MEM-memento.md) |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | Memento store: keeping snapshots between runs | `MementoStore`, `MemoryMementoStore`, `FileMementoStore` | `memento_store.py` | [`FRQ-MEM`](../02-FRQ/FRQ-MEM-memento.md) |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | Pipeline: one transformation of one document | `GenericPipeline` | `pipeline.py` | [`FRQ-PIP`](../02-FRQ/FRQ-PIP-pipeline.md) |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | Processor: runs records through the pipelines | `GenericProcessor` | `processor.py` | [`FRQ-PRC`](../02-FRQ/FRQ-PRC-processor.md) · [`FRQ-PRC-01.22`](../02-FRQ/FRQ-PRC-01.22-document-flow.md) |
| [`StRS-REPOSITORY`](StRS-REPOSITORY.md) | Repository: permanent storage of documents through strategies | `GenericRepository`, `RepositoryWithDriver` | `repository.py` | [`FRQ-REP`](../02-FRQ/FRQ-REP-repository.md) |
| [`StRS-SERIALISATION`](StRS-SERIALISATION.md) | Format boundary: parser, formatter and converter | `GenericParser`, `GenericFormatter`, `GenericConverter`, `ConversionStrategy` | `serialisation.py` | [`FRQ-SER-PAR`](../02-FRQ/FRQ-SER-PAR-parser.md) · [`FRQ-SER-FMT`](../02-FRQ/FRQ-SER-FMT-formatter.md) · [`FRQ-SER-CNV`](../02-FRQ/FRQ-SER-CNV-converter.md) · converters [`FRQ-CNV-01…07`](../02-FRQ/FRQ-000-INDEX.md) |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | State machine: permitted transitions between states | `StateMachine`, `GuardedStateMachine` | `state_machine.py` | [`FRQ-SMC`](../02-FRQ/FRQ-SMC-state-machine.md) |
| [`StRS-STRATEGY`](StRS-STRATEGY.md) | Strategy: a replaceable way of doing one job | `Strategy`, `StrategyCreate`, `StrategyGenerate`, `StrategyRead`, `StrategyWrite` | `strategy.py` | [`FRQ-STR`](../02-FRQ/FRQ-STR-strategy.md) |
| [`StRS-WORKFLOW`](StRS-WORKFLOW.md) | Workflow: the assembled flow that runs the processors | `Workflow` | `workflow.py` | [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) |

### Components without a StRS yet

Their requirements are carried by the FRQ alone until a StRS is written.

| Component | Classes | Module | FRQ |
|---|---|---|---|
| Root base class | `Wattleflow` | `concrete/base.py` | [`FRQ-PTN`](../02-FRQ/FRQ-PTN-root-base.md) |
| Helpers | `Attribute`, `NameHelper` | `concrete/helper.py` | [`FRQ-HLP`](../02-FRQ/FRQ-HLP-helpers.md) |
| Lazy iterators | `LazyIterator`, `LazyAsyncIterator`, `ThreadSafeLazyIterator` | `concrete/iterator.py` | [`FRQ-ITR`](../02-FRQ/FRQ-ITR-lazy-iterators.md) |
| Observable | `ThreadSafeObservable` | `concrete/observable.py` | [`FRQ-OBS`](../02-FRQ/FRQ-OBS-observable.md) |
| Orchestrator | `Orchestrator` | `concrete/orchestrator.py` | [`FRQ-ORC`](../02-FRQ/FRQ-ORC-orchestrator.md) |
| Scheduler | `Scheduler` | `concrete/scheduler.py` | [`FRQ-SCH`](../02-FRQ/FRQ-SCH-scheduler.md) |
| Singleton | `Singleton` | `concrete/singleton.py` | [`FRQ-SGT`](../02-FRQ/FRQ-SGT-singleton.md) |
| Moment (time) | — | `helpers/moment` | [`FRQ-MMN`](../02-FRQ/FRQ-MMN-moment.md) |

## Cross-cutting rules (`BR-PTN`)

These rules apply to every component of the generic layer and have no StRS of their own. They were carried by the former
generic-layer requirement (`HLRQ-01-GENERIC-LAYER`) and are cited by the FRQ and NFRQ entries, so they are kept here.
The component rules of that requirement (`BR-WFL`, `BR-BBD`, `BR-PRC`, `BR-DRV`) are superseded by the business rules of the
StRS entries above.

| oznaka | pravilo |
|---|---|
| **BR-PTN-01** | Svaki objekt frameworka nosi identitet izveden iz vlastitog tipa; identitet se ne postavlja izvana i ne mijenja nakon konstrukcije. |
| **BR-PTN-02** | Auditabilnost nije opcija. Nasljeđuje se na jednom deklariranom mjestu, pa je svaki potomak nosi po ugovoru, ne kao nuspojavu druge baze. |
| **BR-PTN-03** | Prijelaz stanja koji automat ne dopušta ne izvodi se. Stanje se mijenja isključivo primjenom akcije nad tablicom prijelaza ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md), `BR-SMC-03`). |
| **BR-PTN-04** | Tajna se ne zapisuje. Vrijednost kredencijala nikad ne ulazi u audit zapis; zapisuje se samo *je li* autentikacija postignuta ([`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md)). |
| **BR-PTN-05** | Iznimka jednog sloja ne prolazi kroz drugi nepromijenjena — svaki sloj je omata u vlastiti razred, s uzrokom (`raise … from e`), da trag kaže **gdje** je puklo. |
| **BR-PTN-06** | *Prijedlog.* **Usporedba i granica definiraju se u dokumentaciji prije nego što uđu u kod.** Svaka vrijednost koja se uspoređuje, ograničava, skraćuje, zaokružuje ili za koju se alocira prostor deklarira vrstu, jedinicu, preciznost, uključivost granice, referencu (zona, kodiranje), ponašanje izvan granice i značenje odsutnosti. Usporedive su samo vrijednosti iste vrste — vrijeme s pomakom i vrijeme bez pomaka nikad se ne uspoređuju, a vrijednost druge vrste odbija se **gdje ulazi**, ne gdje bi usporedba pukla. Nedeklarirana granica je kvar dokumentacije, ne izbor implementacije. Definicije i zadane vrijednosti: [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md). |
| **BR-PTN-07** | Klasa koja nasljeđuje `Wattleflow` deklarira `__slots__` (prazan `()` ako ne drži stanje) i ne ponavlja slotove baze. Klasa bez `__slots__` nosi `__dict__`; to je dopušteno samo uz opravdanje zapisano u njezinu FRQ-u ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md)). |

## Traceability to the NFRs

The first two rows are the cross-cutting groups that were carried by the former generic-layer and moment requirements. The remaining rows are taken from the StRS entries themselves.

| StRS | Key NFRs |
|---|---|
| generic layer (all components) | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) · [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) · [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) · [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md)…[`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) · [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) · [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) · [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) |
| moment (time) | [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) · [`NFRQ-DEF-04`](../03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md) · [`NFRQ-DEF-05`](../03-NFRQ/NFRQ-DEF-05-moment-naive-helper.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) · [`NFRQ-ORG-09`](../03-NFRQ/NFRQ-ORG-09-external-standard.md) |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | [`NFRQ-DEF-04`](../03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md) · [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) |
| [`StRS-PIPELINE`](StRS-PIPELINE.md), [`StRS-REPOSITORY`](StRS-REPOSITORY.md), [`StRS-STRATEGY`](StRS-STRATEGY.md) | [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) |

## Open

1. The StRS entries do not yet link every NFR that applies to them.
2. The `BR-PTN` rules keep their numbers so that the existing references stay valid; whether they should become a StRS of their own (for example for the root base class) is undecided.
