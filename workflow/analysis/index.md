# Index znanja — wattleflow-workflow

**Status:** NACRT v0.1 (2026-05-29). Faza 1 prema `CLAUDE.md §4`.

Ovaj dokument je polazna karta frameworka. Služi kao dogovorena referenca prije nego krenemo s detaljnom dokumentacijom modula. Sadržaj se mijenja kroz razgovore.

---

## 1. Arhitektonski slojevi

Framework je organiziran u četiri sloja. Hijerarhija odražava ovisnosti — viši slojevi koriste niže.

```
Sloj 4 — Specijalizacije     connections/ drivers/ documents/ pipelines/ processors/
                             strategies/ blackboards/ repositories/ orchestrators/
                             managers/ mappers/ schedulers/ decorators/

Sloj 3 — Cross-cutting       oscal/ audit/ helpers/ constants/ api/

Sloj 2 — Concrete            blackboard connection document driver exception logger
         (radna osnova)      manager memento orchestrator pipeline processor
                             repository scheduler state_machine strategy workflow

Sloj 1 — Core                framework behavioral concurrent creational structural
         (sučelja, GoF+)     transactional
```

**Pravilo ovisnosti:** sloj N može koristiti slojeve `< N`. Suprotno nije dopušteno.

---

## 2. Sloj 1 — `core/` (autoritativna sučelja)

Šest modula, ~1170 linija koda. Definira **~50 sučelja** organiziranih po GoF kategorijama plus konkurentne i transakcijske pattern-e. Sva sučelja nasljeđuju `IWattleflow`.

| Modul | Linija | Svrha | Ključna sučelja |
|---|---|---|---|
| `framework.py` | 44 | Korijensko sučelje i type variables | `IWattleflow`, `T`, `W` |
| `creational.py` | 104 | Kreacijski pattern-i | `IFactory`, `IBuilder`, `IPrototype`, `ISingleton` |
| `structural.py` | 109 | Strukturni pattern-i | `IAdapter`, `IComposite`, `IDecorator`, `IFacade`, `IProxy` |
| `behavioral.py` | 364 | Ponašajni pattern-i (17 sučelja) | `IObserver`, `IStrategy`, `IState`, `IVisitor`, `IMemento`, `ILogger` |
| `concurrent.py` | 338 | Konkurentni/distribuirani pattern-i (20+ sučelja) | `IActor`, `IFuture`, `IPublisher`/`ISubscriber`, `IMapper`/`IReducer`, BSP, Fork/Join |
| `transactional.py` | 232 | Domenski i transakcijski pattern-i | `IDocument`, `IDriver`, `IRepository`, `IBlackboard`, `IPipeline`, `IProcessor`, `IUnitOfWork`, `ISaga`, `IScheduler` |

**Napomena:** `core/` je autoritativno — ne mijenja se bez eksplicitne odluke (vidi `CLAUDE.md §2.5`).

---

## 3. Sloj 2 — `concrete/` (radna osnova frameworka)

15 modula, ~3600 linija. Generičke implementacije sučelja iz `core/`. Korisnici frameworka u pravilu nasljeđuju iz ovih klasa, ne iz `core/`.

| Modul | Klase | Implementira | Uloga |
|---|---|---|---|
| `blackboard.py` | `GenericBlackboard[T]` | `IBlackboard` | Shared canvas, FSM (IDLE→READY→DIRTY→CLEARED), flush u repozitorije |
| `connection.py` | `GenericConnection[C]` | `IObservable` | Lifecycle konekcija (FSM), observer notifikacije, context manager |
| `document.py` | `Document`, `DocumentAdapter`, `DocumentFacade` | `IAdaptee`, `IAdapter`, `ITarget` | Tipizirani sadržaj + audit metadata + UUID |
| `driver.py` | `GenericDriver`, `LazyDriverProxy` | `IDriver`, `IObserver` | I/O driver lifecycle, lazy initialization |
| `exception.py` | `AuditException` + 10+ subklasa | (Python exception) | Location tracking (file/line/code), context chaining |
| `logger.py` | `AuditLogger`, `AsyncHandler`, `ContextFilter` | `ILogger` | Centralizirani audit logging, queue-based async |
| `manager.py` | `ConnectionManager`, `DriverManager`, `ProcessorManager` | `IObserver` | Registracija i lifecycle kolekcija |
| `memento.py` | `GenericMemento` | `IMemento` | Immutable snapshot (MappingProxyType) |
| `orchestrator.py` | `Orchestrator` | `IEventSource`, `IFacade` | Koordinacija procesora, event emitting |
| `pipeline.py` | `GenericPipeline` | `IPipeline` | Data transformation pass |
| `processor.py` | `GenericProcessor` | `IProcessor`, `IOriginator` | Iteracija + transformacija + commit; FSM + memento |
| `repository.py` | `GenericRepository` | `IRepository` | Persistence facade, delegira strategijama |
| `scheduler.py` | `Scheduler` | `IScheduler`, `IEventListener` | Event-driven task scheduling |
| `state_machine.py` | `StateMachine[State, Action]` | `IStateMachine` | Generički FSM engine |
| `strategy.py` | `Strategy`, `StrategyCreate/Read/Write/Generate` | `IStrategy` | Pluggable I/O strategije |
| `workflow.py` | `GenericWorkflow`, `WorkflowFactory` | `IOriginator` | Top-level orkestrator, DI manageri |

**Cross-cutting u concrete/:** `AuditLogger` mixin koristi se u 14+ klasa.

---

## 4. Sloj 3 — Cross-cutting

### 4.1 `oscal/` — NIST OSCAL 1.1.2 integracija

8 datoteka. Compliance gating prema OSCAL kataloga i profila. Trenutni fokus: ASD ISM + Essential Eight (ML1/ML2/ML3).

**Arhitektonska odluka (2026-05-29):** OSCAL će biti **dekorater**, ne bazna klasa. Konekcije nasljeđuju `GenericConnection` i deklariraju kontrole kroz `OSCAL_CONTROLS: ClassVar[Tuple[str, ...]]` class atribut. Dekorater `@oscal_policy(...)` (TODO) čita ovu konstantu i izvršava `OSCALPolicy.verify()`. `OSCALConnection` klasa u `oscal/connection.py` ostaje privremeno do implementacije dekoratera.

| Datoteka | Klase | Uloga |
|---|---|---|
| `models.py` | `Catalog`, `Group`, `Control`, `Profile`, `Param`, `Part`, `Prop`, `Link`, ... | OSCAL data model (frozen dataclasses) |
| `loaders.py` | `ASDOSCALCatalogLoader`, `ASDOSCALProfileLoader` | JSON → model (`IStrategy`) |
| `registry.py` | `OSCALCatalogRegistry` | In-memory katalog lookup |
| `resolver.py` | `resolve(profile, catalog)` | Filtriranje kontrola prema profilu |
| `policy.py` | `OSCALPolicy`, `OSCALPolicyError` | Compliance validacija komponenti |
| `connection.py` | `OSCALConnection` | Bazna klasa konekcija s OSCAL provjerom (`_verify_oscal()` u `ensure_created()`) |

**Pokrivenost OSCAL tipova:**
- ✅ `catalog`, `profile` (resolvanje, registracija, policy enforcement)
- ❌ `component-definition`, `system-security-plan`, `assessment-plan`, `assessment-results`, `plan-of-action-and-milestones`

### 4.2 `audit/` — **PRAZAN folder**

Samo `__init__.py`. SIEM forwarding ne postoji u kodu. Infrastruktura postoji u `concrete/logger.py` (`AsyncHandler` queue-based, `AuditLogger.subscribe_handler()` hook) ali nije aktivirana. Veliki nesklad s CLAUDE.md §6.2.

### 4.3 `helpers/` — utility infrastruktura

- **Time:** `Now` (UTC/lokalno, ISO, timestamp)
- **Reflection:** `Attribute` (type check, conversion, validation), `ClassLoader`
- **Config:** `Config` (YAML loader), `ConfigAdapter`, `ConfigValidator`
- **Secret resolvers (Chain of Responsibility):** `EnvVarResolver`, `AwsSecretsResolver`, `AzureKeyVaultResolver`, `GcpSecretResolver`, `VaultResolver` → `SecretResolverChain`
- **Config validatori (`IConfigValidator`):** `TypeValidator`, `RequiredKeysValidator`, `AllowedKeysValidator`, `AllowedValuesValidator`, `NonEmptyValidator`
- **Pattern support:** `MementoClass`, `ObservableClass`, `TraceHandler`
- **System:** `FileStorage`, `Proxy`, `Project`, `ShellExecutor`, `TempPathHelper`
- **Data:** `DequeList`, `AttributeDict`, `Dictionary`, normaliser/sanitiser/streams/yaml

**Napomena:** ove klase su bile lažno deklarirane u `concrete/__init__.py.__all__` — uklonjeno 2026-05-29.

### 4.4 `constants/` — enumi i semantika

- `enums.py`: `Classification` (UNCLASSIFIED, OFFICIAL, PROTECTED, SECRET, TOP_SECRET — vladina UK/AUS klasifikacija), `ClassificationDLM`, **`Event`** (100+ stavki za audit), `Operation`, `PipelineAction`, `PipelineType`, `ProvenanceHandler`
- `audit.py`: `ConnectionStatus`, `EventLog`, `LogFormat`, `ProtectiveMarkings`, **`WattleflowOSCAL`** (veza OSCAL ↔ constants)
- `mimetypes.py`: `MimeTypes` (30+ tipova)
- `filetype.py`: `FileType` + heuristike
- `keys.py`: string konstante za config (`KEY_NAME`, `KEY_BLACKBOARD`, ...)
- `errors.py`: error poruke

### 4.5 `decorators/`

- **`PresetDecorator`** — koristi se u `concrete.driver/blackboard/connection/repository/pipeline/processor`
- `EncryptedPreset`, `PSPFDecorator`, file decorator

### 4.6 `api/` — FastAPI MVC

Sučelja: `IEndPoint`, `IController`, `IModel`, `IView`. Rute: `/`, `/status`, `/api/youtube/{url}`, `/api/dashboard/uri`.

---

## 5. Sloj 4 — Specijalizacije

Konkretne implementacije nad `concrete/` za specifične izvore podataka i tipove obrade. Sadržaj nije detaljno mapiran u Fazi 1, samo evidentiran.

**Arhitektonska odluka (2026-05-29):** Sloj 4 izlazi iz `wattleflow` i `wattleflow-workflow` core paketa u zaseban projekt (radni naziv `wattleflow-examples`). Razlog: zero-trust paketiranje — svaka specijalizacija povlači third-party ovisnost koja je potencijalni vektor napada. Detalji: [`analysis/zero-trust-architecture.md`](analysis/zero-trust-architecture.md). Vidi i `CLAUDE.md §7`.

| Folder | Sadržaj (primjer) |
|---|---|
| `connections/` | `postgres.py`, `sftp_paramiko.py`, ... (vidljivo iz git statusa) |
| `drivers/` | konkretni I/O driveri |
| `documents/` | `file.py`, `wattle.py`, ... |
| `pipelines/` | `dataframe.py`, `emails.py`, `pdf.py`, `png.py`, `quality.py`, `reductions.py`, `text.py` |
| `processors/`, `strategies/`, `blackboards/`, `repositories/`, `orchestrators/`, `managers/`, `mappers/`, `schedulers/` | specijalizacije |

---

## 6. Identificirane praznine i otvorena pitanja

Stavke za razgovor s korisnikom. Označeno `?` znači da nacrt indexa pretpostavlja, ali nije potvrđeno kodom.

### 6.1 SIEM forwarding

`CLAUDE.md §6.2` kaže da SIEM pristup za audit prosljeđuje informacije eksternim sustavima. U analiziranom kodu (`concrete/logger.py`, `oscal/`) nema eksplicitnog SIEM transporta (npr. CEF/LEEF/syslog/Kafka exporter).

**Pretpostavka:** `audit/` folder sadrži SIEM forwarding logiku — treba potvrditi.

**Pitanje za razgovor:**
- Gdje se trenutno nalazi SIEM forwarding logika?
- Koji su ciljani SIEM protokoli (syslog, CEF, LEEF, OpenTelemetry, custom)?

### 6.2 OSCAL pokrivenost

Trenutno: samo `catalog` + `profile` + ASD ISM/E8.

**Pitanja:**
- Plan za `component-definition` (komponente bi self-deklarirale koje kontrole zadovoljavaju)?
- Plan za `assessment-results` (rezultate provjera u OSCAL formatu)?
- Drugi izvorni katalozi (NIST 800-53, ISO 27001, CIS) — planirani ili out-of-scope?

### 6.3 Observability / Grafana

`CLAUDE.md §6.4` najavljuje export metrika za dashboarde. U trenutnom kodu nema vidljivog Prometheus/OpenTelemetry exportera.

**Pitanja:**
- Format metrika (Prometheus, OpenTelemetry, statsd)?
- Gdje će se metrike emitirati (procesor lifecycle, FSM tranzicije, throughput, latency)?

### 6.4 Test framework

Prema `CLAUDE.md §4`, odluka nije donesena. Treba se dogovoriti prije Faze 3 (standardi implementacije).

### 6.5 Veza Sloj 4 → Sloj 2

Treba potvrditi jesu li sve specijalizacije iz Sloja 4 nasljeđuju iz Sloja 2 (npr. svaka konkretna konekcija iz `connections/` mora nasljediti `GenericConnection` ili `OSCALConnection`).

---

## 7. Sljedeći koraci

Predloženo (potvrditi u razgovoru):

1. **Razgovor o ovom nacrtu** — provjera točnosti, popunjavanje §6 (otvorena pitanja).
2. **Detaljno mapiranje cross-cutting slojeva** — `audit/`, `helpers/`, `constants/`, `decorators/`, `api/`.
3. **Uzorak specijalizacije** — odabrati 1 modul iz Sloja 4 (npr. `connections/postgres.py`) i napraviti end-to-end mapu kako se sučelja iz `core/` materijaliziraju kroz `concrete/` u konkretnu klasu.
4. **Finalizirati strukturu `docs/`** — potvrditi hijerarhiju iz `CLAUDE.md §3.4` na temelju ovog indexa.
5. **Prijelaz u Fazu 2** — dokumentacija pojedinih modula prema dogovorenom Why → What → How predlošku.

---

## 8. Reference

- `CLAUDE.md` — standardi projekta, redoslijed faza, integrirani standardi
- `src/wattleflow/core/` — sučelja
- `src/wattleflow/concrete/` — radna osnova
- `src/wattleflow/oscal/` — OSCAL integracija
- Sirovi izlazi analize (interni — Faza 1): zapisi `Explore` agenata za `core/`, `concrete/`, `oscal/`
