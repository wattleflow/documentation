# Index znanja — wattleflow-workflow

**Status:** NACRT v0.2 (2026-08-10). Prethodna inačica: v0.1 (2026-05-29).
**Doseg:** distribucija `wattleflow-workflow` — stablo `src/wattleflow`. `core/` je **zasebna
distribucija** (repozitorij `wattleflow/core`, `DR-WFL-006`) i ovdje se opisuje kao ovisnost.
Specijalizacije (Sloj 4 iz v0.1) više nisu u ovom stablu — vidi §5.

Ovaj dokument je karta frameworka, **ne registar**: gdje se razilazi s `03-NFRQ/` ili
`DOCTRINE.md`, prednost ima registar (D-02). Sadržaj je provjeren nad stablom na gornji datum;
brojke su opažanja, ne kriterij.

> **Što je v0.2 ispravila.** v0.1 je (a) svrstavala `helpers/` **iznad** `concrete/`, suprotno
> `NFR-ORG-01`; (b) tvrdila da `framework.py` nosi type variable `T` i `W`; (c) opisivala
> `oscal/`, `api/`, `audit/` i specijalizacije kojih u ovom stablu nema. Detalji ispravaka su
> u §7.

---

## 1. Arhitektonski slojevi

**Ispravak v0.2.** v0.1 je stavljala `helpers/` u „Sloj 3 — cross-cutting", **iznad**
`concrete/`, uz pravilo „sloj N smije koristiti slojeve `< N`" — po tome bi
`helpers → concrete` bio dopušten. `NFR-ORG-01` tvrdi suprotno: dijeljeni `helpers/` core
**nikad** ne smije uvoziti iz domenskog paketa (Acyclic Dependencies Principle), i to provodi
`wem_lint` (`ORG-01` K1/K2). Registar je nadređen ovom nacrtu (D-02), pa je model ispravljen
prema njemu. Posljedica te ispravke je otvoreni problem dizajna zapisan u
[`type-registry-ownership.md`](type-registry-ownership.md).

```
Sloj 4 — Domenski moduli    schedulers/  managers/  orchestrators/     (danas uglavnom stubovi)

Sloj 3 — Concrete           blackboard connection document driver exception helpers iterator
         (radna osnova)     logger manager memento observable orchestrator pipeline processor
                            repository scheduler singleton state_machine strategy wattleflow
                            workflow

Sloj 2 — Dijeljeni temelj   helpers/   decorators/

Sloj 1 — Konstante          constants/

Sloj 0 — Sučelja            core/   (zasebna distribucija: framework behavioural concurrent
         (GoF+)                      creational structural transactional)
```

**Pravilo ovisnosti:** sloj N smije uvoziti samo iz slojeva `< N`. Suprotan smjer je nalaz.

**Izmjereni bridovi (2026-08-10, `wem_lint --select ORG-01`):**

| Paket | Uvozi iz | Napomena |
|---|---|---|
| `concrete` | `core`, `constants`, `helpers`, `decorators` | u skladu s modelom |
| `schedulers` | `concrete`, `constants` | u skladu s modelom |
| `helpers` | `core`, `constants`, **`concrete`** | **prekršaj** — 1× K1, 3× K2 (§6) |
| `decorators` | `core`, `constants` | čisto u praćenom stablu |
| `constants` | — | list |
| `core` | — | list (vanjska distribucija) |

`decorators/oscal/` uvozi `concrete.state_machine` i `wattleflow.oscal`, ali **nije praćen u
gitu** i `wattleflow.oscal` se ne razrješava (`CLAUDE.md §6.1`). U grafu se pojavljuje jer alat
čita disk, ne indeks — deklarirana razlika, ne izuzeće.

---

## 2. Sloj 0 — `core/` (autoritativna sučelja, zasebna distribucija)

Šest modula + `__init__.py`, **2018 linija**, **83 sučelja**. Sva nasljeđuju `IWattleflow`.

| Modul | Linija | Sučelja | Ključna sučelja |
|---|---|---|---|
| `framework.py` | 53 | 1 | `IWattleflow` — jedini ugovor: stabilan `name` |
| `creational.py` | 125 | 5 | `IFactory`, `IBuilder`, `ICreator`, `IProduct`, `IPrototype` |
| `structural.py` | 216 | 12 | `IAdapter`, `IComposite`, `IDecorator`, `IFacade`, `IProxy`, `IFlyweight(Factory)` |
| `behavioural.py` | 459 | 23 | `IObserver`/`IObservable`, `IStrategy`, `IState(Machine)`, `IMemento`, `IOriginator`, `ILogger`, `IHandler` |
| `concurrent.py` | 558 | 27 | `IActor`, `IFuture`, `IPublisher`/`ISubscriber`, `IMapper`/`IReducer`, BSP, Fork/Join |
| `transactional.py` | 410 | 15 | `IDocument`, `IDriver`, `IRepository`, `IBlackboard`, `IPipeline`, `IProcessor`, `IUnitOfWork`, `ISaga`, `IScheduler` |

**Ispravci prema v0.1:**
- `framework.py` **ne sadrži type variable** `T` ni `W` — samo `IWattleflow` (53 linije).
  Type variable žive u `behavioural`/`concurrent`/`structural`/`transactional` i imenovane su
  **po ulozi** prema `NFR-ORG-03` (`Action`, `Context`, `Result`, `State`, `Element`, `Key`,
  `Value`, `Message`, `Input`, `Output`, `Content`, `Entity`, `Event`, `Item`, `Vertex`,
  `Edge`, `Extrinsic`, `Destination`). Golih `T`/`W` u `core/` nema.
- Modul se zove **`behavioural.py`** (UK), ne `behavioral.py`.
- **`ISingleton` ne postoji** u `creational.py`. Singleton je izveden u
  `concrete/singleton.py`, bez sučelja u `core/`.
- Broj sučelja: 83, ne „~50"; opseg: 2018 linija, ne „~1170".

`core/` je autoritativan i mijenja se samo kroz `DR-COR` (`CLAUDE.md §2.5`).

---

## 3. Sloj 3 — `concrete/` (radna osnova frameworka)

21 modul + `__init__.py`, **4933 linije**. Korisnici frameworka nasljeđuju odavde, ne iz `core/`.

| Modul | Klase | Implementira | Uloga |
|---|---|---|---|
| `wattleflow.py` | `Wattleflow` | `IWattleflow` | korijen: `AuditLogger` + identitet; baza svega u `concrete/` |
| `blackboard.py` | `BlackboardState`, `BlackboardAction`, `GenericBlackboard` | `IBlackboard` | shared canvas, FSM, flush u repozitorije |
| `connection.py` | `ConnectionAction/State`, `ConnectionObserverInterface`, `GenericConnection` | `IObservable` (preko `ConnectionObserverInterface`) | lifecycle konekcija (FSM), context manager |
| `document.py` | `Document`, `DocumentAdapter`, `DocumentFacade` | `IAdaptee` / `IAdapter` / `ITarget` (redom) | tipizirani sadržaj + audit metadata + UUID |
| `driver.py` | `DriverMetadata/Action/State`, `GenericDriver`, `LazyDriverProxy` | `IDriver`, `IObserver` | I/O lifecycle, lazy inicijalizacija |
| `exception.py` | `AuditException` + 26 podklasa | (Python exception) | praćenje lokacije (file/line/code), lanac konteksta |
| `helpers.py` | `Attribute`, `NameHelper` | — | domenski-lokalna refleksija i provjera atributa |
| `iterator.py` | `LazyIterator`, `LazyAsyncIterator` | `IIterator`, `IAsyncIterator` | lijena iteracija |
| `logger.py` | `AuditLogger`, `AsyncHandler`, `ContextFilter` | `ILogger` | centralizirani audit, queue-based async |
| `manager.py` | `ConnectionManager`, `DriverManager`, `ProcessorManager` (+3 iznimke) | `IObserver` | registracija i lifecycle kolekcija |
| `memento.py` | `GenericMemento` | `IMemento` | nepromjenjiva snimka (`MappingProxyType`) |
| `observable.py` | `ThreadSafeObservable` | `IObservableReactive` | thread-safe obavijesti |
| `orchestrator.py` | `Orchestrator` | `IEventSource`, `IFacade` | koordinacija procesora, emitiranje događaja |
| `pipeline.py` | `GenericPipeline` | `IPipeline` | transformacijski prolaz |
| `processor.py` | `ProcessorState/Action`, `GenericProcessor` | `IProcessor`, `IOriginator` | iteracija + transformacija + commit; FSM + memento |
| `repository.py` | `GenericRepository`, `RepositoryWithDriver` | `IRepository` | perzistencija, delegira strategijama |
| `scheduler.py` | `Scheduler` | `IScheduler` | raspoređivanje vođeno događajima |
| `singleton.py` | `Singleton` | `IWattleflow` | jedinstvena instanca (bez vlastitog sučelja u `core/`) |
| `state_machine.py` | `StateMachine`, `GuardedStateMachine` | `IStateMachine` | generički FSM |
| `strategy.py` | `Strategy`, `StrategyCreate/Read/Write/Generate` | `IStrategy` | zamjenjive I/O strategije |
| `workflow.py` | `GenericWorkflow`, `WorkflowFactory`, `WorkflowFactoryLogger` | `IOriginator` | vršni orkestrator, DI manageri, registar tipova |

**Ispravak prema v0.1:** nedostajalo je 5 modula (`helpers`, `iterator`, `observable`,
`singleton`, `wattleflow`); opseg je 4933, ne „~3600" linija.

---

## 4. Slojevi 1–2 — temelj

### 4.1 `constants/` — jedini pravi list

Bez ijednog uvoza iz `wattleflow.*`.

| Modul | Sadržaj |
|---|---|
| `enums.py` | `Classification`, `ClassificationDLM`, **`Event`**, `Operation`, `PipelineAction`, `PipelineType`, `ProvenanceHandler` |
| `audit.py` | `ConnectionStatus`, `EventLog`, `LogFormat`, `ProtectiveMarkings`, `WattleflowOSCAL` |
| `filetype.py` | `FileType` + heuristike detekcije |
| `mimetypes.py` | `MimeTypes` |
| `keys.py` | string konstante za konfiguraciju (`KEY_NAME`, `KEY_BLACKBOARD`, …) |
| `errors.py` | tekstovi poruka o pogreškama |

### 4.2 `helpers/` — dijeljene sposobnosti

Namespace paket (PEP 420, bez `__init__.py`); uvoz je uvijek **eksplicitan submodul**.

| Sposobnost | Moduli | Ključna imena |
|---|---|---|
| konfiguracija | `config`, `config_adapter`, `config_validator`, `dotenv`, `yaml` | `Config`, `ConfigAdapter`, `ConfigValidator`, `ISecretResolver`, `EnvVarResolver`, `DotEnvResolver`, `SecretResolverChain` |
| vrijeme | `datetime` | `Now`, `CreatedVerdict`, `CreatedWithin` |
| refleksija / sustav | `system`, `pathadder`, `functions` | `ClassLoader`, `FileStorage`, `Project`, `Proxy`, `ShellExecutor`, `TempPathHelper` |
| datoteke i tokovi | `files`, `streams`, `digest` | `FileScanner`, `FileSourceScanner`, `TextStream`, `TextFileStream`, `FileDigest` |
| tekst | `normaliser`, `sanitiser`, `macros`, `generators`, `randomiser` | `Normaliser`, `CaseText`, `TextMacros`, `Snowflake`, `Timestamp` |
| usmjeravanje | `routing` | `RoutingLabel`, `RoutingRule`, `DestinationRouter`, `LocalStorageDestinationRouter` |
| podaci | `dictionaries` | `AttributeDict`, `Dictionary` |
| pattern podrška | `memento`, `handlers`, `audit` | `MementoClass`, `ObservableClass`, `TraceHandler`, `AsyncAuditHandler` |
| formatiranje/parsiranje | — | preseljeno: ugovor je `IParser`/`IFormatter` (core `transactional`), generička baza `GenericParser`/`GenericFormatter` (`concrete/serialisation.py`), implementacije i tvornice u `wattleflow-processors` (`DR-COR-015`) |

**Ispravci prema v0.1:** `AwsSecretsResolver`, `AzureKeyVaultResolver`, `GcpSecretResolver`,
`VaultResolver` i `DequeList` **ne postoje** u ovom stablu. `IConfigValidator` iz
`config_adapter.py` je lanac odgovornosti (`TypeValidator`, `RequiredKeysValidator`,
`AllowedKeysValidator`, `AllowedValuesValidator`, `NonEmptyValidator`) i **različit je pojam**
od `ConfigValidator` iz `config_validator.py` — imenska kolizija je otvoreno pitanje (§7).

### 4.3 `decorators/`

`PresetDecorator` (`preset.py`) koristi se u 7 modula `concrete/`
(`blackboard`, `connection`, `driver`, `pipeline`, `processor`, `repository`, `scheduler`).
Uz njega `FileClass` (`file.py`) i `PSPFDecorator` (`pspf.py`).
`decorators/oscal/` (`policy.py`, `wrappers.py`) postoji na disku, **nije praćen u gitu** i ne
razrješava se dok OSCAL paket nije deployan (`CLAUDE.md §6.1`).

---

## 5. Što je izašlo iz ovog stabla

**Odluka iz v0.1 (2026-05-29) je provedena.** Specijalizacije `connections/`, `drivers/`,
`documents/`, `pipelines/`, `processors/`, `strategies/`, `blackboards/`, `repositories/`,
`mappers/` više nisu u `wattleflow-workflow`; žive u distribuciji **`wattleflow-processors`**
(radni naziv `wattleflow-examples` iz v0.1 je napušten). Regulira `DR-WFL-002` + `NFR-SEC-03`;
detalji u [`zero-trust-architecture.md`](zero-trust-architecture.md), `CLAUDE.md §7`.

**Ne postoje u ovom stablu** (v0.1 ih je opisivala kao prisutne): `oscal/`, `api/`, `audit/`.
- `oscal/` — vlastita distribucija, još nije deployana (`CLAUDE.md §6.1`).
- `audit/` — SIEM forwarding je **aspiracija** (`CLAUDE.md §6.2`); infrastruktura postoji u
  `concrete/logger.py` (`AsyncHandler`, `AuditLogger.subscribe_handler()`) i
  `helpers/audit.py` (`AsyncAuditHandler`), ali transport ne postoji.
- `api/` — FastAPI MVC sloj (`IEndPoint`/`IController`/`IModel`/`IView`, rute `/status`,
  `/api/youtube/{url}`) nije u ovom stablu; status neodlučen (§7).

**Preostali domenski moduli su stubovi:** `managers/__init__.py` (12 linija),
`orchestrators/__init__.py` (12), `schedulers/` (`__init__.py` 19 + `cron_job.py` 56).

---

## 6. Mjereno stanje (`wem_lint`)

| | |
|---|---|
| alat | `wem_lint 1.12.0`, pravila `ORG-01` |
| kriterij | `tools/dictionary.json 0.8.0` (`dictionary.yaml 0.2.0`) |
| platforma | python 3.11.15 (Linux) |
| stablo | `wattleflow-workflow` — `src/wattleflow` |
| vektor | `import-cycle` ERROR 1 · `wrong-direction-import` ERROR 3 · `single-consumer-helper` WARNING 4 · `declared-blind-spot` INFO 4 |

Vektor je nominalan — zbrajanje je jedina dopuštena operacija, ukupne ocjene nema (D-09).
Run s greškama arhivira se kao `finding-vector`, ne kao C-snimka (`CLAUDE.md §9`).

**Deklarirane slijepe pjege:**
- fan-in broji samo izravne `wattleflow.helpers.<mod>` uvoze; agregatni uvoz nije razriješen →
  donja granica (`DR-WFL-007`).
- uvozi **unutar** `helpers/` nisu bridovi → ciklusi i slojevitost unutar `helpers/` su
  nemjereni.
- pravilo klasificira samo bridove `helpers ↔ domena`; **domena ↔ domena** ciklusi nisu
  pokriveni.
- `ORG-01` kriteriji 2 i 4 nisu automatizirani.

---

## 7. Otvorena pitanja

1. **Vlasništvo registra tipova** — `helpers/config_validator.py` vs `concrete/workflow.py`.
   Zaseban zapis: [`type-registry-ownership.md`](type-registry-ownership.md). Otvoreno: je li
   registar sposobnost (`helpers/`) ili domenski primitiv (`concrete/`), i je li validacija
   jednokratan čin ili nadzor (Observer).
2. **Preostali `helpers → concrete` prekršaji** — `config.py:15,16` i `config_adapter.py:20`
   posežu za `AuditException` i `Wattleflow` bazom. Traži spuštanje tih primitiva ispod
   `helpers/` ili lokalne iznimke; dira `concrete/` (`CLAUDE.md §2.5`).
3. **Imenska kolizija `ConfigValidator` vs `IConfigValidator`** — dva različita pojma pod
   jednim korijenom imena (§4.2). Kandidat za preimenovanje kroz DR (D-12).
4. **Status `api/` sloja** — je li napušten, odgođen ili pripada drugoj distribuciji.
5. **SIEM forwarding** — protokoli (syslog/CEF/LEEF/OpenTelemetry) i mjesto emitiranja;
   danas aspiracija (`CLAUDE.md §6.2`).
6. **OSCAL pokrivenost** — `component-definition`, `assessment-results`; katalozi izvan ASD ISM
   (NIST 800-53, ISO 27001, CIS).
7. **Observability** — format metrika i transport; svaka mjera podliježe protokolu V1–V6 i
   povelji `[M]`.
8. **Test framework** — nije odabran; blokira Fazu 4 (`CLAUDE.md §4`).

---

## 8. Reference

- `CLAUDE.md` / `POLICY.md` — policy sloj; `03-NFRQ/` — `NFR-ORG-01…05`, `NFR-SEC-01…05`
- `documentation/workflow/dr/DR-WFL-INDEX.md` — aktivna serija odluka
- [`zero-trust-architecture.md`](zero-trust-architecture.md) — lokalnost distribucije
- [`type-registry-ownership.md`](type-registry-ownership.md) — otvoreni problem dizajna
- `src/wattleflow/concrete/` — radna osnova; `tools/wem_lint.py` — provedba kriterija
- Distribucija `wattleflow` (core, GitHub `wattleflow/core`) — sučelja
