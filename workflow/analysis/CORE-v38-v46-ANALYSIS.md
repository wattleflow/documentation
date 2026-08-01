# CORE v0.0.0.38 → v0.0.0.46 — utjecaj na `wattleflow-workflow`

**Datum analize:** 2026-07-28
**Analizirani raspon (core):** `d79d49a` (v0.0.0.39) … `6ae672e` (v0.0.0.46), tj. `HEAD~8..HEAD`
**Stanje workflowa:** `default` @ `b26c5d0` (v0.0.0.85), radni direktorij na v0.0.0.86
**Metoda:** git diff nad core stablom + statička provjera imena uvoza + runtime provjera `__abstractmethods__`
uz privremeni `T` shim + build sdista

---

## 1. Uzrok: što se točno promijenilo u core-u

Core je u tom rasponu pretvoren u **čisti sloj ugovora**. Sva funkcionalnost koja je prije živjela
u sučeljima ispala je i mora se nadomjestiti u `concrete/` sloju workflowa.

| Prije | Sada | Posljedica za workflow |
|---|---|---|
| `IWattleflow` = **konkretna** klasa, `__init__` postavlja `self.name` | `IWattleflow(ABC)` s **apstraktnim read-only** `name` + `__slots__ = ()` | svaka klasa u workflowu postala je apstraktna |
| `T`, `W` TypeVar u `framework.py` | uklonjeni; role-TypeVarovi su module-scoped i namjerno se **ne** re-exportaju (ORG-03) | `from wattleflow.core import T` puca |
| `ISingleton` (creational) | **uklonjen** | `IScheduler` više nije singleton |
| `IIterator` / `IAsyncIterator` s lazy `__init__` / `__next__` / `__iter__` | samo `create_iterator()` | iterator implementacije nemaju `__next__` |
| `IObservableReactive` konkretan (RLock + snapshot + suppress) | čisti ABC | nema implementacije |
| `IAdapter.__init__(adaptee)` + `self._adaptee` | apstraktno `adaptee` property | konstruktor ne postoji |
| `IAdaptee.specific_request()` → `return self` | apstraktna | identity default nestao (DR-009) |
| `IDocument.identifier`, `ISignal.identifier` / `__slots__` | uklonjeni iz ugovora | |
| `behavioral.py` | `behavioural.py` | preimenovanje modula |
| `ICreator` / `IProduct` | zamijenjene definicije (ispravak semantike) | |
| `IEventSource` | `Generic[Event]`, `emit_event(event, **kwargs)` | |
| `IScheduler(ISingleton, Generic[T], IEventSource)` | `IScheduler(IEventSource[Event], ABC)` | |
| `IComponent/IComposite/IDecorator/IFlyweight` nad `T` | nad `Element` / `Extrinsic` | |

Nadomjesne implementacije stigle su kao četiri nove datoteke u `concrete/`:
[`wattleflow.py`](src/wattleflow/concrete/wattleflow.py),
[`singleton.py`](src/wattleflow/concrete/singleton.py),
[`iterator.py`](src/wattleflow/concrete/iterator.py),
[`observable.py`](src/wattleflow/concrete/observable.py).

---

## 2. BLOKERI (P0) — workflow se trenutno **ne može ni importati**

### 2.1 `wattleflow.concrete` puca na importu paketa

[src/wattleflow/concrete/workflow.py:137](src/wattleflow/concrete/workflow.py#L137) instancira `AuditLogger`
na razini modula:

```
TypeError: Can't instantiate abstract class AuditLogger with abstract method name
```

Poziv ide preko [src/wattleflow/concrete/__init__.py:37](src/wattleflow/concrete/__init__.py#L37),
pa je **cijeli `concrete` paket neuvozan**. Sve ostalo je zamaskirano iza ovoga.

### 2.2 `from wattleflow.core import T`

[blackboard.py:24](src/wattleflow/concrete/blackboard.py#L24) i
[document.py:25](src/wattleflow/concrete/document.py#L25). `T` više ne postoji.
Zamjena role-imenom (`Content` za document, `Item`/`Element` za blackboard) ujedno je i ORG-03 usklađenje.

### 2.3 Nijedna konkretna klasa ne nasljeđuje `Wattleflow` → sve su apstraktne

Novi `Wattleflow` mixin **nigdje se ne koristi**. Mjereno uz privremeni `T` shim:

**`concrete/` — 26 klasa apstraktno zbog `name`:**
`AuditLogger`, `ConnectionObserverInterface`, `GenericConnection`, `Document`, `DocumentAdapter`,
`DocumentFacade`, `GenericDriver`, `LazyDriverProxy`, `ConnectionManager`, `DriverManager`,
`ProcessorManager`, `GenericMemento`, `Orchestrator`, `GenericPipeline`, `GenericProcessor`,
`GenericRepository`, `RepositoryWithDriver`, `Scheduler`, `StateMachine`, `GuardedStateMachine`,
`Strategy`, `StrategyGenerate`, `StrategyCreate`, `StrategyRead`, `StrategyWrite`
(+ `GenericBlackboard`, `GenericWorkflow` — nisu ni došle do provjere zbog §2.1)

**`helpers/`, `schedulers/` — još 11:**
`IConfigValidator`, `TypeValidator`, `RequiredKeysValidator`, `AllowedKeysValidator`,
`AllowedValuesValidator`, `NonEmptyValidator`, `ConfigAdapter`, `MementoClass`, `ObservableClass`,
`ClassLoader`, `CronJobScheduler`

### 2.4 Same nove datoteke su nedovršene

```
LazyIterator          abstract=['create_iterator', 'name']
LazyAsyncIterator     abstract=['create_iterator', 'name']
ThreadSafeObservable  abstract=['name']        ← nije upotrebljiva
Singleton             abstract=[]  ✔
Wattleflow            abstract=[]  ✔
```

`ThreadSafeObservable` je zamišljen kao gotova politika, a ne može se instancirati.
`iterator.py` i `observable.py` trebaju naslijediti `Wattleflow`.
(Kod `LazyIterator` je `create_iterator` legitimno apstraktan — `name` nije.)

### 2.5 Nove datoteke nisu izložene

[src/wattleflow/concrete/__init__.py](src/wattleflow/concrete/__init__.py) ne importa niti exporta
`Wattleflow`, `Singleton`, `LazyIterator`, `LazyAsyncIterator`, `ThreadSafeObservable`.

---

## 3. Tiha pogrešna ponašanja (P1)

Tijelo apstraktnog `name` je `...`, pa **`self.name` vraća `None`** umjesto da baci grešku.

| Mjesto | Efekt |
|---|---|
| [connection.py:132-133](src/wattleflow/concrete/connection.py#L132) | `self._observers[observer.name]` → svi observeri pod ključem `None`; **registrira se samo jedan** |
| [connection.py:145](src/wattleflow/concrete/connection.py#L145), [document.py:198](src/wattleflow/concrete/document.py#L198), [manager.py:88](src/wattleflow/concrete/manager.py#L88), [logger.py:173](src/wattleflow/concrete/logger.py#L173) | audit zapisi oblika `None:uuid` — **falsificiran audit trail**, izravno protiv DR-002 obrazloženja |
| [helpers.py:263](src/wattleflow/concrete/helpers.py#L263) | poruka `Mandatory: None.foo` |

### Pisanje u read-only property → `AttributeError`

- [state_machine.py:47](src/wattleflow/concrete/state_machine.py#L47) — `self.name = name`
- [state_machine.py:99](src/wattleflow/concrete/state_machine.py#L99) — `self.name = getattr(inner, "name", ...)`

Ovo je **odluka, ne samo popravak**: `StateMachine` traži instance-scoped ime, a DR-002 propisuje
ime izvedeno iz tipa i neizmjenjivo.

---

## 4. Slomljeni konstruktori i ugovori (P1)

| Mjesto | Problem |
|---|---|
| [document.py:214-221](src/wattleflow/concrete/document.py#L214) | `DocumentAdapter` zove `IAdapter.__init__(self, adaptee=adaptee)` — više ne postoji → `TypeError`; `self._adaptee` nikad nije postavljen; `adaptee` property neimplementiran |
| [scheduler.py:51](src/wattleflow/concrete/scheduler.py#L51) | `IScheduler.__init__(self, *args, **kwargs)` → `object.__init__` odbija argumente |
| [scheduler.py:30](src/wattleflow/concrete/scheduler.py#L30) | **`Scheduler` je izgubio singleton semantiku**; guard `if not hasattr(self, "_initialised")` je mrtav kod. Core docstring propisuje `class Scheduler(Wattleflow, IScheduler[Event], Singleton)` |
| [orchestrator.py:61](src/wattleflow/concrete/orchestrator.py#L61) | `IEventSource` je sada `Generic[Event]` — treba parametrizirati |
| [system.py:73](src/wattleflow/helpers/system.py#L73) | `IWattleflow.__init__(self)` radi, ali `ClassLoader` ostaje apstraktan |
| [document.py:53](src/wattleflow/concrete/document.py#L53) | `Document(IAdaptee, ...)` — **ne nasljeđuje `IDocument`**; usto je `identifier` ispao iz core ugovora, a `Document` ga i dalje nudi |

---

## 5. Latentni ciklički import

```
helpers.config → concrete.exception → concrete/__init__ → concrete.workflow
              → helpers.config_adapter → helpers.config   ✗
```

```
ImportError: cannot import name 'Config' from partially initialized module
'wattleflow.helpers.config' (most likely due to a circular import)
```

Puca kad se `wattleflow.helpers.config` uveze **prvi**. Istovremeno je **NFR-ORG-01 prekršaj**
(`helpers` → `concrete`), pa je dobar prvi realni test za ORG-01 pravilo linta.

---

## 6. Packaging — release blocker za v0.0.0.86

[MANIFEST.in](MANIFEST.in) je izgubio sve `recursive-include src/wattleflow/… *.py` retke.
Provjereno stvarnim buildom:

```
wattleflow_workflow-0.0.0.86.tar.gz
├── LICENSE, PKG-INFO, README.md, pyproject.toml, requirements-dev.txt, setup.cfg
├── src/wattleflow_workflow.egg-info/SOURCES.txt
└── tools/{naming_registry.yaml, wem_lint.py}
```

**Nula Python izvornog koda.** Uz `global-exclude *` na vrhu, wheel (gradi se iz sdista) bio bi prazan.

Uz to:

- `recursive-include documentation/*.md` je sintaktički kriv — traži `<dir> <pattern>`; ovo je no-op.
- `prune documentation` + `recursive-include documentation` su kontradiktorni.
- Uklonjen `prune src/wattleflow/core` — obrana od dev symlinka je nestala.

---

## 7. `wem_lint.py` — analiza i plan

### 7.1 Dvije potpuno različite alatke

| | core `wem_lint.py` v0.4.1 | workflow `wem_lint.py` v1.2.0 |
|---|---|---|
| Predmet | sloj sučelja (`core/*.py`) | cijelo stablo (`src/wattleflow`) |
| Pravila | `R-CORE-{HDR,SFX,IMP,TYP,STA,FAC,ABS,EXC}` | `ORG-01/02/03` |
| Registry | **JSON** preko `--registry`, defaulti ugrađeni | **YAML**, obavezan (nema ugrađenog rječnika) |
| Uzorci | `ITemplate` run + `IStrategy` pravila + `IBuilder` report + lokalni `WemComponent(IWattleflow)` | `IStrategyContext` + `IStrategy` + `IFactory` + `IIterator`/`ISyncAggregate` + `IBuilder` |
| Izlaz | vektor po dimenzijama, **bez agregatnog skora** (NFR §3.4) | friendly/compact + ASCII graf ovisnosti |
| Ovisnosti | samo stdlib | traži PyYAML |

Nisu duplikati — **komplementarni su**. Core mjeri „je li ovo doista samo ugovor",
workflow mjeri „je li implementacija strukturno ispravna".

### 7.2 Workflow lint je i sam slomljen istim uzrokom

```
$ python tools/wem_lint.py --src src/wattleflow --registry tools/naming_registry.yaml
TypeError: Can't instantiate abstract class DependencyLocalityRule with abstract method name
```

- `NomenclatureRule`, `DependencyLocalityRule`, `TypeVarRule`, `WemLint`, `RuleFactory`,
  `ImportGraphBuilder` — svi apstraktni na `name`
- [PyFileIterator:186](tools/wem_lint.py#L186) nasljeđuje `IIterator[Path]` koji više nema `__next__`
  → treba `LazyIterator`

Core je isti problem riješio lokalnim `WemComponent(IWattleflow)` mixinom, uz eksplicitno obrazloženje
da core tools **ne smiju** ovisiti o workflow distribuciji. Workflow lint smije koristiti prave
`Wattleflow` / `LazyIterator` — to je legitimni dogfooding vlastite distribucije.

### 7.3 Što vrijedi prenijeti iz core verzije

1. **Instrument-validity changelog.** Unos v0.4.0 je metodološki najvrjedniji dio:
   `inspect.isabstract` je izgubio diskriminativnu moć čim je korijen postao apstraktan
   → proxy se razišao s konstruktom **bez ijedne izmjene alata**. Ista klasa greške sada
   pogađa workflow (37 klasa „apstraktno" iako nijedna to nije po namjeri).
   Workflow lint treba **`R-WF-ABS-03` obrnutog smjera**: *konkretna klasa čiji su jedini
   apstraktni članovi naslijeđeni iz korijena = nedovršena implementacija identiteta* —
   to bi automatski našlo svih 37 klasa iz §2.3.
2. **Deklarirane slijepe točke.** `exclude_modules` → `R-CORE-EXC-01 INFO`.
   Workflow registry ima `blind_spots:` kao slobodan tekst; treba ga učiniti strojno emitiranim.
3. **Vektorski izlaz bez agregatnog skora**, s eksplicitnom napomenom na nominalnu skalu
   (izravno iz [ANALIZA.md](tools/ANALIZA.md) §3.4).
4. **Verzioniranje kriterija.** `registry_version` + politika „promjena semantike pravila = minor".
   Workflow registry ima `registry_version: "0.2.0"` ali nema politiku.

### 7.4 Nesuglasice koje treba razriješiti

| | core | workflow | napomena |
|---|---|---|---|
| `python_reference` | `3.10` (svjesna odluka: `sys.stdlib_module_names`) | `3.12` | `pyproject` traži `>=3.11`, `ruff target-version = "py310"` — **tri različita broja** |
| Format registryja | JSON | YAML | jedan alat čita drugi format |
| `WattleType` | `tolerated` | `tolerated` | u core-u je **stvarno uklonjen** → unos je mrtav |
| `bases.root` | — | `[IWattleflow]` | treba `[IWattleflow, Wattleflow, Singleton]` |
| `type_vars.roles` | 20 uloga (+`Content`, `Entity`, `Event`, `Extrinsic`, `Item`) | 16 (+`Connection`, `Adaptee`) | rječnici se razilaze |

---

## 8. Prijedlog redoslijeda

**Faza 0 — otključavanje**

1. `Wattleflow` u `LazyIterator` / `LazyAsyncIterator` / `ThreadSafeObservable`
2. `AuditLogger(Wattleflow, ILogger)` — otključava ~24 klase odjednom
3. `T` → role-imena u `blackboard.py`, `document.py`
4. Export svih 5 novih simbola iz `concrete/__init__.py`
5. Vratiti `recursive-include` retke u `MANIFEST.in`

**Faza 1 — ugovori**

6. `DocumentAdapter.adaptee` property
7. `Scheduler(Wattleflow, IScheduler[Event], Singleton)`
8. `Orchestrator(Wattleflow, IEventSource[Event], IFacade)`
9. `StateMachine.name` — odluka o `label`
10. `helpers` → `concrete` ciklus

**Faza 2 — lint**

11. Popraviti `tools/wem_lint.py` (§7.2)
12. Uskladiti registry (§7.4)
13. Dodati `R-WF-ABS-03` pravilo (§7.3.1)

**Faza 3 — dokumentacija**

14. FR.md / NFR.md dopune na temelju izmjerenog

---

## 9. Otvorene odluke

1. **`Wattleflow` vs `Singleton` duplikacija** — obje imaju identičan `name` / `__str__` / `__repr__`.
   Da `Singleton(Wattleflow)`, ili ostaje namjerna duplikacija (zbog `__slots__` / MRO)?
2. **`StateMachine.name`** — preimenovati u `label`, ili definirati ADR iznimku za instance-scoped ime?
3. **`Document` i `IDocument`** — `Document` ne nasljeđuje `IDocument`. Spojiti, ili je `IDocument` mrtav ugovor?
4. **`python_reference`** — 3.10, 3.11 ili 3.12? Trenutno tri različita broja na tri mjesta.
5. **Registry format** — YAML svugdje (traži PyYAML u core toolsu) ili JSON svugdje
   (gubi komentare, a oni nose ADR obrazloženja)?
6. **Jedan lint ili dva?** — spojiti core i workflow pravila u jedan alat s profilima,
   ili držati odvojeno uz zajednički registry?

---

## Dodatak A — reprodukcija mjerenja

```bash
# apstraktnost svih klasa u concrete/ (uz privremeni T shim)
PYTHONPATH=src python - <<'EOF'
import typing, wattleflow.core as core
core.T = typing.TypeVar("T")
import importlib, inspect
mods = ["blackboard","connection","document","driver","exception","helpers","iterator",
        "logger","manager","memento","observable","orchestrator","pipeline","processor",
        "repository","scheduler","singleton","state_machine","strategy","wattleflow","workflow"]
for m in mods:
    try: mod = importlib.import_module(f"wattleflow.concrete.{m}")
    except Exception as e: print(f"IMPORT FAIL {m}: {type(e).__name__}: {e}"); continue
    for n, c in vars(mod).items():
        if inspect.isclass(c) and c.__module__ == mod.__name__:
            ab = getattr(c, "__abstractmethods__", frozenset())
            if ab: print(f"{m}.{n}: ABSTRACT -> {sorted(ab)}")
EOF

# sadržaj sdista
python -m build --sdist --no-isolation -o /tmp/wf-dist && tar tzf /tmp/wf-dist/*.tar.gz

# workflow lint
python tools/wem_lint.py --src src/wattleflow --registry tools/naming_registry.yaml
```
