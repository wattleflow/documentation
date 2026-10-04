<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# HLRQ-01-GENERIC-LAYER — Cjevovodna obrada kroz zamjenjive primitive (generički sloj)

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | `Converter`, `Parser` i `Formatter` su uloge — kategorije `CNV`, `PAR`, `FMT` |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `wattleflow-workflow` — clean core tier |
| **Predmet** | `workflow/src/wattleflow/concrete/` — radna osnova frameworka |
| **Djeca** | FRQ-ovi iz stupca *FRQ* tablice u §3 |
| **Sljedivost** | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) (ontologija) · [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) (blast radius) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) (lokalnost distribucije) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md)/[`02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md)/[`03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) (audit zapis) · [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) (`__slots__`) |

## 1. Narativ

Podatkovni posao se u praksi piše kao skripta: dohvat, transformacija i pohrana spleteni u jedan
tok. Takav tok radi dok se ne promijeni jedan njegov kraj — drugi izvor, druga baza, drugi format
— a tada se mijenja **cijeli** jer nijedan njegov dio nema samostalno sučelje.

`core/` na to odgovara skupom apstraktnih ugovora izvedenih iz dizajn patterna. Ali ugovor sam
ne pokreće ništa: između sučelja i konkretnog posla nedostaje sloj koji ugovor **ispunjava** —
koji zna kako se objekt gradi, kako se prijavljuje u audit, kako se ruši, kako se njegovo stanje
sprema i vraća. Bez tog sloja svaka bi specijalizacija te odgovornosti izvodila iznova, i to
različito.

**Zašto.** Sloj `concrete/` postoji da bi zamjena bilo kojeg primitiva bila **izmjena
konfiguracije, ne koda**. Cijena za to je da generička klasa preuzme sve što je zajedničko —
identitet, audit, životni ciklus, automat stanja, granice kvara — a specijalizaciji ostavi
isključivo ono što je za nju specifično. Mjera uspjeha je koliko malo specijalizacija mora
napisati, a ne koliko generička klasa može.

## 2. Mjesto u dekompoziciji

```
core/          apstraktni ugovori (IWattleflow, IBlackboard, IProcessor, …)   ← autoritativno
   ↑
concrete/      generičke implementacije                                       ← OVAJ ZAHTJEV
   ↑
specijalizacije                                                               ← izvan ovog zahtjeva
```

Ovisnost je **jednosmjerna**: `concrete/` ne smije uvoziti iz specijalizacija.
Ta jednosmjernost je ono što `concrete/` drži u clean core tieru — sloj koji bi posegnuo za
third-party ovisnošću preselio bi cijelu distribuciju ([`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md)).

## 3. Ontologija funkcionalnosti

Generičke klase sloja po oznaci kategorije FRQ-a, poredane abecedno.
Deset oznaka su ontološki primitivi (`BBD`, `CON`, `DOC`, `DRV`, `MEM`, `PIP`, `PRC`, `REP`, `STR`, `WFL`);
`SER` nosi tri uloge na granici formata (`Converter`, `Parser`, `Formatter`), što čini trinaest uloga.
Ostale oznake (`PTN`, `HLP`, `ITR`, `MGR`, `OBS`, `ORC`, `SCH`, `SGT`, `SMC`) su pattern-infrastruktura i nisu primitivi.

| oznaka | klasa | modul | FRQ |
|---|---|---|---|
| **BBD** | `GenericBlackboard` | [`blackboard.py`](../../../workflow/src/wattleflow/concrete/blackboard.py) | [`FRQ-BBD`](../02-FRQ/FRQ-BBD-blackboard.md) |
| **CON** | `GenericConnection`, `ConnectionObserverInterface` | [`connection.py`](../../../workflow/src/wattleflow/concrete/connection.py) | [`FRQ-CON`](../02-FRQ/FRQ-CON-connection.md) |
| **DOC** | `Document`, `DocumentAdapter`, `DocumentFacade`, `DummyReadDocument` | [`document.py`](../../../workflow/src/wattleflow/concrete/document.py) | [`FRQ-DOC`](../02-FRQ/FRQ-DOC-document.md) |
| **DRV** | `GenericDriver`, `LazyDriverProxy` | [`driver.py`](../../../workflow/src/wattleflow/concrete/driver.py) | [`FRQ-DRV`](../02-FRQ/FRQ-DRV-driver.md) |
| **HLP** | `Attribute`, `NameHelper` | [`helpers.py`](../../../workflow/src/wattleflow/concrete/helpers.py) | [`FRQ-HLP`](../02-FRQ/FRQ-HLP-helpers.md) |
| **ITR** | `LazyIterator`, `LazyAsyncIterator` | [`iterator.py`](../../../workflow/src/wattleflow/concrete/iterator.py) | [`FRQ-ITR`](../02-FRQ/FRQ-ITR-lazy-iterators.md) |
| **MEM** | `GenericMemento` | [`memento.py`](../../../workflow/src/wattleflow/concrete/memento.py) | [`FRQ-MEM`](../02-FRQ/FRQ-MEM-memento.md) |
| **MGR** | `ConnectionManager`, `DriverManager`, `ProcessorManager` | [`manager.py`](../../../workflow/src/wattleflow/concrete/manager.py) | [`FRQ-MGR`](../02-FRQ/FRQ-MGR-managers.md) |
| **OBS** | `ThreadSafeObservable` | [`observable.py`](../../../workflow/src/wattleflow/concrete/observable.py) | [`FRQ-OBS`](../02-FRQ/FRQ-OBS-observable.md) |
| **ORC** | `Orchestrator` | [`orchestrator.py`](../../../workflow/src/wattleflow/concrete/orchestrator.py) | [`FRQ-ORC`](../02-FRQ/FRQ-ORC-orchestrator.md) |
| **PIP** | `GenericPipeline` | [`pipeline.py`](../../../workflow/src/wattleflow/concrete/pipeline.py) | [`FRQ-PIP`](../02-FRQ/FRQ-PIP-pipeline.md) |
| **PRC** | `GenericProcessor` | [`processor.py`](../../../workflow/src/wattleflow/concrete/processor.py) | [`FRQ-PRC`](../02-FRQ/FRQ-PRC-processor.md) |
| **PTN** | `Wattleflow` | [`base.py`](../../../workflow/src/wattleflow/concrete/base.py) | [`FRQ-PTN`](../02-FRQ/FRQ-PTN-root-base.md) |
| **REP** | `GenericRepository`, `RepositoryWithDriver` | [`repository.py`](../../../workflow/src/wattleflow/concrete/repository.py) | [`FRQ-REP`](../02-FRQ/FRQ-REP-repository.md) |
| **SCH** | `Scheduler` | [`scheduler.py`](../../../workflow/src/wattleflow/concrete/scheduler.py) | [`FRQ-SCH`](../02-FRQ/FRQ-SCH-scheduler.md) |
| **SER** | `GenericConverter`, `GenericParser`, `GenericFormatter` | [`serialisation.py`](../../../workflow/src/wattleflow/concrete/serialisation.py) | [`FRQ-SER-CNV`](../02-FRQ/FRQ-SER-CNV-converter.md),  [`FRQ-SER-PAR`](../02-FRQ/FRQ-SER-PAR-parser.md), [`FRQ-SER-FMT`](../02-FRQ/FRQ-SER-FMT-formatter.md) |
| **SGT** | `Singleton` | [`singleton.py`](../../../workflow/src/wattleflow/concrete/singleton.py) | [`FRQ-SGT`](../02-FRQ/FRQ-SGT-singleton.md) |
| **SMC** | `StateMachine`, `GuardedStateMachine` | [`state_machine.py`](../../../workflow/src/wattleflow/concrete/state_machine.py) | [`FRQ-SMC`](../02-FRQ/FRQ-SMC-state-machine.md) |
| **STR** | `Strategy`, `StrategyGenerate`, `StrategyCreate`, `StrategyRead`, `StrategyWrite`, `StrategyReadDummy` | [`strategy.py`](../../../workflow/src/wattleflow/concrete/strategy.py) | [`FRQ-STR`](../02-FRQ/FRQ-STR-strategy.md) |
| **WFL** | `GenericWorkflow`, `WorkflowFactory`, `WorkflowFactoryLogger` | [`workflow.py`](../../../workflow/src/wattleflow/concrete/workflow.py) | [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) |

Enumi i [`DriverMetadata`](../../../workflow/src/wattleflow/concrete/driver.py) nose oznaku svoje klase; iznimke su pattern-infrastruktura (`PTN`), osim
`ParserError`, `FormatterError` i `ConverterError`, koje idu uz svoje uloge.


## 4. Opseg

**U opsegu:** sve u `workflow/src/wattleflow/concrete/`.


### Zajednički ugovor generičke klase

Svaka klasa koja nasljeđuje `Wattleflow`:

1. **Nasljeđuje `Wattleflow` prvo**, ispred svojeg pattern sučelja, i nikad ne imenuje `Audit`
   sama. Redoslijed baza je **ograničenje, ne stil** — [MRO](../../GLOSSARY.md#abbr-mro) se inače ne linearizira
   ([`TODO.md`](../../workflow/TODO.md) §`__slots__` i MRO).
2. **Prosljeđuje cijeli `**kwargs` naviše nepromijenjen.** Razdvajanje logging ključeva od
   ostatka događa se na jednom mjestu, u `Wattleflow.__init__`; podklasa koja to ponovi
   duplicira podjelu i razilazi se čim se doda novi ključ.
3. **Prijavljuje na ulazu, ne na izlazu.** Audit tok se čita odozgo nadolje redom kojim se posao
   odvija; zatvaranje jedinice pripada sloju koji jedinicu posjeduje — procesoru i
   workflowu.
4. **Ne diže iznimku iz destruktora.** `__del__` prijavljuje kvar i nastavlja; iznimka u
   destruktoru maskira onu koja se stvarno dogodila.
5. **Deklarira `__slots__`** (`BR-PTN-07`, [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md)).

Svaki modul deklarira `__all__` ([`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) t.2). Ugovor ne vrijedi za klase koje ne nasljeđuju
`Wattleflow`: `GenericParser`, `GenericFormatter`, `GenericConverter` (lagana razina, bez audita),
`StateMachine`, `GuardedStateMachine` i `Singleton`.

## 5. Poslovna pravila

Pravila nose oznaku kategorije kojoj pripadaju (`BR-<oznaka>-<nn>`).

| oznaka | pravilo |
|---|---|
| **BR-WFL-01** | Zamjena primitiva je izmjena konfiguracije, ne koda. Klasa se razrješava po imenu iz konfiguracije (`WorkflowFactory.resolve`). |
| **BR-WFL-02** | Konfiguracija koja se ne razriješi zaustavlja workflow **prije** obrade, ne tijekom nje. |
| **BR-BBD-01** | Osnovni blackboard izvozi stanja, akcije i tablicu prijelaza (`BlackboardState`, `BlackboardAction`, `TRANSITIONS`), ali sam ne drži automat; automat drži specijalizacija ([`FRQ-BBD`](../02-FRQ/FRQ-BBD-blackboard.md)). |
| **BR-PRC-01** | Kvar u jednom prolazu ne ostavlja objekt u stanju iz kojeg se ne može ni nastaviti ni čisto završiti — `FAILED` je stanje iz kojeg vodi oporavak (`LOAD`) i završetak (`CLEAN`/`STORE`). |
| **BR-DRV-01** | Sloj ne posjeduje granicu prema vanjskom sustavu. Mrežu, disk i baze dodiruju driver i konekcija; generička klasa iznad njih ne poznaje protokol. |
| **BR-PTN-01** | Svaki objekt frameworka nosi identitet izveden iz vlastitog tipa; identitet se ne postavlja izvana i ne mijenja nakon konstrukcije. |
| **BR-PTN-02** | Auditabilnost nije opcija. Nasljeđuje se na jednom deklariranom mjestu, pa je svaki potomak nosi po ugovoru, ne kao nuspojavu druge baze. |
| **BR-PTN-03** | Prijelaz stanja koji automat ne dopušta ne izvodi se. Stanje se mijenja isključivo primjenom akcije nad tablicom prijelaza. |
| **BR-PTN-04** | Tajna se ne zapisuje. Vrijednost kredencijala nikad ne ulazi u audit zapis; zapisuje se samo *je li* autentikacija postignuta ([`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md)). |
| **BR-PTN-05** | Iznimka jednog sloja ne prolazi kroz drugi nepromijenjena — svaki sloj je omata u vlastiti razred, s uzrokom (`raise … from e`), da trag kaže **gdje** je puklo. |
| **BR-PTN-06** | *Prijedlog.* **Usporedba i granica definiraju se u dokumentaciji prije nego što uđu u kod.** <br> Svaka vrijednost koja se uspoređuje, ograničava, skraćuje, zaokružuje ili za koju se alocira prostor deklarira vrstu, jedinicu, preciznost, uključivost granice, referencu (zona, kodiranje), ponašanje izvan granice i značenje odsutnosti. <br>Usporedive su samo vrijednosti iste vrste — vrijeme s pomakom i vrijeme bez pomaka nikad se ne uspoređuju, a vrijednost druge vrste odbija se **gdje ulazi**, ne gdje bi usporedba pukla. Nedeklarirana granica je kvar dokumentacije, ne izbor implementacije.<br> Definicije i zadane vrijednosti: [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md); provjera rubnim vrijednostima na razini unit, sistemskog i UAT testa. |
| **BR-PTN-07** | Klasa koja nasljeđuje `Wattleflow` deklarira `__slots__` (prazan `()` ako ne drži stanje) i ne ponavlja slotove baze. Klasa bez `__slots__` nosi `__dict__`; to je dopušteno samo uz opravdanje zapisano u njezinu FRQ-u ([`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md)). |

## 6. Nefunkcionalni zahtjevi

| [NFRQ](../../GLOSSARY.md#abbr-nfrq) | posljedica za ovu sposobnost |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | trinaest uloga je zatvoren popis; *cross-cutting* sposobnost je pozivljivi helper, ne novi primitiv |
| [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) | metoda koja referira vlastitu klasu je `@classmethod` |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | zajedničko ponašanje živi u generičkoj klasi, ne prepisano po specijalizacijama |
| [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) | [`BR-PTN-06`](#5-poslovna-pravila); fasete svake usporedbe i granice, zadane vrijednosti i rubni testovi — *prijedlog* |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | [`BR-PTN-07`](#5-poslovna-pravila); `Wattleflow` djeca deklariraju `__slots__` osim uz opravdanje za `__dict__` |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius.md) | jedan primitiv = jedna odgovornost; kompromitacija drivera ne doseže blackboard |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `__all__` u svakom modulu; `PresetDecorator` je jedina ulazna površina za konfiguraciju |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | import closure sloja je `stdlib ∪ wattleflow`; nijedan modul ne smije referirati third-party |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | [`BR-PTN-04`](#5-poslovna-pravila); redakcija je odgovornost `Audit` obitelji, ne pojedine klase |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | značenje, publika i smještaj razina audita; mjeri se lintom |
| [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) | audit zapis je par (događaj, imenovana polja); ime polja ima jedno značenje u cijelom frameworku |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | vlasništvo, redoslijed i volumen audita po jedinici posla |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.1 | 2026-10-02 | Prvi draft dokumentacije |
