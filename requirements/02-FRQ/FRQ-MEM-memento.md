<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-MEM — Snimka stanja (memento)

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericMemento(Wattleflow, IMemento)` — nepromjenjiva snimka stanja |
| **Sestrinski** | [`FRQ-PRC`](FRQ-PRC-processor.md) (proizvođač i potrošač snimke) · [`FRQ-SMC`](FRQ-SMC-state-machine.md) (stanje koje snimka nosi) |
| **Izvedba** | `workflow/src/wattleflow/concrete/memento.py` · `memento_store.py` (pohrana) · `processor.py` (kontrolna točka) · `workflow.py` (zadani `save_state`/`restore_state`, odjeljak `memento:`) |
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

Memento je **nepromjenjiv spremnik** koji nosi ono što je potrebno za nastavak prekinutog posla.
Jedini je primitiv ovog sloja bez apstraktnih metoda i bez specijalizacija — jer i nema što
specijalizirati: sadržaj snimke određuje onaj tko je snima (`IOriginator`), ne snimka.

Tri načina pristupa istom sadržaju, namjerno:

| pristup | čemu služi |
|---|---|
| `memento.cycle` (atribut) | čitljivost na mjestu poziva |
| `get_state()` | kanonski ključ `"state"`, koji `IMemento` traži |
| `to_dict()` | kad se snimka predaje dalje kao podatak |

**Nepromjenjivost je plitka, i to je zapisano.** `_data` je `MappingProxyType` nad kopijom
rječnika, pa se sam skup ključeva ne može mijenjati; vrijednosti se čuvaju **po referenci**. Za
integritet promjenjive vrijednosti odgovoran je vlasnik snimke, ne snimka. Ta je granica u
docstringu izrečena, ne prešućena (D-11).

**Payload se ne krati.** Konstruktor prosljeđuje cijeli `**payload` naviše, a `Wattleflow` čita
logging ključeve s **vlastite kopije** — pa snimka zadrži svaki ključ koji je pozivatelj predao,
uključujući onaj koji se slučajno zove `level` ili `handler`.

### Pohrana snimke i kontrolna točka

Snimka sama po sebi živi u memoriji. Da bi prekinuti posao mogao nastaviti i nakon pada procesa,
snimku sprema **pohrana** (`MementoStore`, strategija po uzoru na `IStrategy`), a **kontrolnu točku** vodi procesor.

| element | uloga |
|---|---|
| `MementoStore` (`write`, `read`, `clear`, `execute`) | zadnja snimka po ključu; ne tumači sadržaj, drži samo obične podatke (`Enum` se zapisuje imenom člana) |
| `MemoryMementoStore` | snimke u procesu: preživljavaju pad posla, ne pad procesa |
| `FileMementoStore(path)` | jedna JSON datoteka po ključu, zamjena atomična (privremena datoteka + `os.replace`); isti `path` nakon ponovnog pokretanja nastavlja posao |
| `GenericProcessor(memento_store=, memento_key=, checkpoint_every=)` | nakon svakog ciklusa (ili svakog n-tog) zapisuje `cycle` i stanje `FAILED` (prekinut posao je neuspjeli), pri `start()` nastavlja iza spremljenog ciklusa, a po uspješnom završetku briše snimku |
| `GenericWorkflow.save_state/restore_state` | zadano: snimka svakog procesora po registriranom imenu; klasa workflowa ne treba vlastiti `WorkflowMemento` |
| `memento:` u YAML-u workflowa | `store: memory \| file \| <registrirani razred>`, `path`, `checkpoint_every`; bez odjeljka procesori ništa ne snimaju |

Kontrolna točka je **opcionalna**: bez `memento_store` procesor ne piše, ne čita i ne briše ništa. Pretpostavka
nastavka je isti skup podataka i isti `path` (aspiracija, D-05: skup podataka se ne provjerava,
preskaču se prvih `cycle` stavki).

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericMemento

top to bottom direction

  interface IMemento {
    {abstract} +get_state() : State
  }
  interface IStrategy
  interface IOriginator {
    {abstract} +save_state() : IMemento
    {abstract} +restore_state(memento : IMemento)
  }
  class Wattleflow
  class GenericMemento {
    #_data : MappingProxyType
    +get_state() : Any
    +to_dict() : dict[str, Any]
  }
Wattleflow <|-down- GenericMemento
IMemento <|.down. GenericMemento
IOriginator .right.> IMemento : creates and receives
  abstract class MementoStore {
    {abstract} +write(key : str, memento : GenericMemento)
    {abstract} +read(key : str) : GenericMemento
    {abstract} +clear(key : str)
    +execute(caller, **kwargs)
  }
  class MemoryMementoStore
  class FileMementoStore {
    -_path : Path
  }
  class GenericProcessor {
    #_memento_store : MementoStore
    #_memento_key : str
    #_checkpoint_every : int
    +save_state() : GenericMemento
    +restore_state(memento : GenericMemento)
  }
IStrategy <|.. MementoStore
MementoStore <|-- MemoryMementoStore
MementoStore <|-- FileMementoStore
GenericProcessor o-right-> MementoStore : checkpoints to
GenericProcessor ..> GenericMemento : creates
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericMemento

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "GenericMemento", "Immutable snapshot of state")
System_Ext(or, "GenericProcessor", "Originator")
System_Ext(pz, "Snapshot holder", "Keeps the snapshot")
Rel_L(or, sys, "Saves and restores state")
Rel_R(pz, sys, "Reads snapshot as data")
@enduml
```

</div>

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `IOriginator` (u `workflow`: `GenericProcessor`) — snima i vraća | `cycle`, `state` |
| **A2** | `GenericMemento` — predmet ovog zahtjeva | payload |
| **A3** | Pozivatelj koji čuva snimku | `to_dict()` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 snima → `save_state()` → `GenericMemento(cycle=…, state=…)` |
| **EV02** | A1 vraća → `restore_state(memento)` → `get_state()` + atributi |
| **EV03** | A3 čita snimku kao podatak → `to_dict()` / `in` |
| **EV04** | procesor završi ciklus i isprazni blackboard → `_checkpoint()` → `MementoStore.write(key, GenericMemento(cycle, FAILED))` |
| **EV05** | procesor bez generatora kreće → `_resume()` → `MementoStore.read(key)` → `restore_state()` |
| **EV06** | procesor uspješno završi → `_release()` → `MementoStore.clear(key)` |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericMemento

left to right direction

skinparam nodesep 10
skinparam ranksep 20

actor "GenericProcessor" as A1
actor "Snapshot holder" as A3
usecase "Save state" as EV01
usecase "Restore state" as EV02
usecase "Read snapshot as data" as EV03
A1 --> EV01
A1 --> EV02
A3 --> EV03
@enduml
```

## 07. Constraints and Preconditions

1. Payload je predan kao **imenovani** argument; pozicijskih nema.
2. Za `get_state()` payload nosi ključ `state`; inače je rezultat `None`.
3. Kontrolna točka traži `flush_per_cycle`: bez ispraznjivanja nakon ciklusa stavke još nisu zapisane, pa bi ih nastavak preskočio. Procesor to odbija pri izgradnji.
4. Ključ snimke je obično ime (`[A-Za-z0-9._-]`, bez staze); tvornica koristi registrirano ime procesora, a bez njega ime razreda.
5. Nastavak pretpostavlja isti skup podataka, isti redoslijed i isti `path` (aspiracija, D-05). Probni prolaz je pokazao da zadani redoslijed `scan` datotečnog procesora nije abecedni (obrađeni `d1`, `d3`); za nastavak je pouzdaniji deterministički redoslijed.
6. Vrijednosti koje moraju preživjeti nepromijenjene su ili nepromjenjive same, ili ih vlasnik
   kopira prije snimanja.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — konstruktor kopira payload u rječnik pa ga zamota u `MappingProxyType`.
2. **EV02** — pristup atributom ide kroz `__getattr__` na `_data`; ključ kojeg nema daje običan
   `AttributeError` s imenom ključa.
3. `get_state()` vraća `_data.get("state")` — **bez** iznimke ako ključa nema.
4. `to_dict()` vraća novi rječnik; pozivatelj ga smije mijenjati bez posljedica za snimku.
5. `__repr__` ispisuje ime i ključeve (`GenericMemento(keys=[cycle, state])`), nikad vrijednosti.
6. `GenericProcessor.restore_state` provjerava cijelu snimku (`state` je `ProcessorState`, `cycle` nenegativan `int`) prije izmjene i diže `ProcessorException`; stanje automata obnavlja novim `StateMachine` (svojstvo `state` je samo za čitanje).

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericMemento

participant "GenericProcessor" as O
participant "GenericMemento" as M
participant "Snapshot holder" as P
activate O #gold
O -> M : GenericMemento(cycle=..., state=...)
activate M #gold
M -> M : _data = MappingProxyType(copy)
deactivate M
activate P #gold
P -> M : to_dict()
activate M #gold
M --> P : new dict
deactivate M
deactivate P
O -> O : restore_state(memento)
O -> M : get_state()
activate M #gold
M --> O : state or None
deactivate M
deactivate O
@enduml
```

</div>

### Kontrolna točka i nastavak

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericMemento

  participant "GenericProcessor" as O
  participant "MementoStore" as S
  activate O
  O -> S : read(key)
  activate S
  S --> O : memento or None
  deactivate S
  opt memento exists
    O -> O : restore_state(memento)
  end
  loop each item
    O -> O : pipelines, flush
    opt cycle % checkpoint_every = 0
      O -> S : write(key, GenericMemento(cycle, FAILED))
    end
  end
  O -> S : clear(key)
  deactivate O
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| pristup nepostojećem ključu | `AttributeError(key)` | ključ se tretira kao odsutan, ne kao interni kvar |
| `get_state()` bez ključa `state` | `None` | odsutnost stanja nije iznimka |
| pokušaj izmjene `_data` | `TypeError` iz `MappingProxyType` | skup ključeva je zaključan |
| promjena vrijednosti kroz vanjsku referencu | **snimka se mijenja s njom** | plitka kopija; odgovornost vlasnika |
| payload nosi `level` / `handler` | ključ ostaje u snimci **i** stiže loggeru | dvostruka namjena ključa je prihvaćena |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericMemento

start
:owner saves: GenericMemento(**payload);
:_data = MappingProxyType(copy of payload);
:owner restores: restore_state(memento);
if (key "state" exists?) then (yes)
:get_state() returns state;
else (no)
:get_state() returns None;
endif
if (attribute requests missing key?) then (yes)
:AttributeError(key);
else (no)
:value from _data;
endif
if (attempt to modify _data?) then (yes)
:TypeError (MappingProxyType);
else (no)
:to_dict() returns new copy;
endif
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Memento` je rezervirani primitiv; snimka ne postaje spremište ([`FRQ-REP`](FRQ-REP-repository.md)) |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | jedno javno ime u `__all__`; skup ključeva zaključan proxyjem |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; nijedan third-party uvoz |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | `__repr__` ispisuje samo ključeve (`test_memento_repr.py`) |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | snimka ne prijavljuje ništa; prijavljuje originator |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Prekinuti posao ima čime nastaviti: snimka nosi stanje automata i napredak, a njezin skup ključeva
se od trenutka nastanka ne mijenja. Nijedan potrošač snimke ne mora znati koji ju je originator
napravio. Uz konfiguriranu pohranu procesor nakon pada (i nakon ponovnog pokretanja, ako je pohrana
datoteka) nastavlja iza zadnjeg dovršenog ciklusa, a uspješan posao ne ostavlja snimku.

## 13. Acceptance Criteria

1. Skup ključeva je nepromjenjiv nakon konstrukcije. ✅
2. Plitkost kopije je **deklarirana**, ne prešućena (D-11). ✅
3. Payload se ne krati — logging ključevi ostaju i u snimci. ✅
4. Odsutan ključ daje `AttributeError`, odsutno `state` daje `None`. ✅
5. `to_dict()` vraća kopiju koju pozivatelj smije mijenjati. ✅
6. `__slots__` je deklariran; modul deklarira `__all__`. ✅
7. Import closure je `stdlib ∪ wattleflow`. ✅
8. Snimka se može trajno pohraniti (`FileMementoStore`), a posao nastavlja nakon ponovnog pokretanja. ✅ — kriteriji 12–15
9. `__repr__` ne otkriva povjerljive vrijednosti. ✅
10. `restore_state` odbija snimku bez `state`/`cycle` ili s pogrešnim tipom s `ProcessorException`, ništa ne mijenja, a ispravna snimka nastavlja nakon spremljenog ciklusa. ✅
11. Klasa `GenericMemento` nije apstraktna: nema što specijalizirati (zatvoreno kao svojstvo, nije jedinstveno u sloju). ✅
12. Pohrana vraća točno ono što je zapisano; ključevi su neovisni; nepoznat ključ daje `None`; ključ koji nije obično ime, i vrijednost koju JSON ne može zapisati, odbijaju se. ✅
13. `FileMementoStore` zapisuje atomično (nema privremenih datoteka, prijašnja snimka ostaje ako zapis ne uspije), oštećena datoteka je pogreška, a ne prazno čitanje. ✅
14. Procesor s pohranom sprema nakon svakog (n-tog) ciklusa, nastavlja iza spremljenog ciklusa i briše snimku nakon uspjeha; bez pohrane ništa ne čita ni ne piše; odbija pohranu bez `flush_per_cycle` i neispravan `checkpoint_every`. ✅
15. Kvar pohrane pri zapisu ili brisanju ne zaustavlja posao (zapis `Event.Save`/`Event.Clear` s `Failed` na `WARNING`); nečitljiva ili nevaljana snimka zaustavlja `start()` s `ProcessorException`. ✅
16. `GenericWorkflow` bez vlastitog koda snima i vraća procesore po imenu; nevaljana snimka ne mijenja ništa. ✅
17. `memento:` odjeljak gradi pohranu (`memory`, `file`, registrirani razred), svakom procesoru daje pohranu, ključ (registrirano ime) i `checkpoint_every`; neispravan odjeljak ili ime procesora je pogreška konfiguracije. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | pregled `__init__` | `MappingProxyType(dict(payload))` |
| 2 | pregled docstringa | „shallow copy … values are stored by reference. Owners are responsible…" |
| 3 | pregled `__init__` + komentar | cijeli `**payload` ide i naviše i u `_data` |
| 4 | pregled `__getattr__` / `get_state` | `raise AttributeError(key)` odnosno `.get("state")` |
| 5 | pregled `to_dict` | `return dict(self._data)` |
| 6 | pregled modula | `__slots__ = ("_data",)`; `__all__ = ["GenericMemento"]` |
| 7 | pregled uvoza | `types`, `typing`, `collections.abc` + `wattleflow.*` |
| 8 | `test_memento_store.py`, `test_processor_checkpoint.py` (`FileResumeTest`) | novi `FileMementoStore` na istom `path`-u nastavlja neuspjeli posao iza spremljenog ciklusa |
| 9 | `workflow/tests/test_memento_repr.py` (5 testova) | vrijednosti i ugniježđeni ključevi se ne ispisuju; vrijednosti dostupne kroz `get_state()`, `to_dict()`, atribute; mutacija (`!r` vrijednosti) ruši test |
| 10 | `workflow/tests/test_processor_restore.py` (9 testova) | odbijanje bez `state`, sa stringom, bez `cycle`, s ciklusom `"2"`/-1/1.5/None/`True`, izvan skupa podataka; odbijena snimka ništa ne mijenja; krug `save_state` → `restore_state`; mutacije (bez provjere stanja, bez provjere ciklusa) uhvaćene |
| 12–13 | `workflow/tests/test_memento_store.py` (25 testova) | ugovor zajednički objema pohranama, `FileMementoStore` (isti `path`, Enum kao ime, atomičnost, oštećena datoteka), `execute`; mutacije (bez `os.replace`, bez provjere oblika, bez provjere ključa) uhvaćene |
| 14–15 | `workflow/tests/test_processor_checkpoint.py` (16 testova) | pad i nastavak (memorija i datoteka), brisanje nakon uspjeha, `checkpoint_every`, neovisni ključevi, kvarovi pohrane; mutacije (bez brisanja, zapis uz svaki ciklus, bez nastavka, pogrešno stanje u snimci) uhvaćene |
| 16–17 (probni prolaz) | izmijenjen `blackwattle/examples/workflows/05_markdown` (uklonjen `WorkflowMemento`, dodan `memento:` s `store: file`), pokrenut kroz stvarni `YAMLConfig` i `WorkflowFactory` s privremenim mapama i pipelineom koji pada na 3. dokumentu | 1. pokretanje: pad nakon 2 dokumenta, `.memento/markdown-to-word/processor-md.json` = `{"cycle": 2, "state": "FAILED"}`; 2. pokretanje (novi proces, isti `path`): zapis `Restore cycle=2`, obrađena preostala 3, sva 5 izlaza, snimka obrisana. Neizmijenjena putanja primjera (0 ciklusa zbog filtra datuma) i dalje se pokreće bez promjene |
| 16–17 | `workflow/tests/test_workflow_memento.py` (13 testova) | zadani `save_state`/`restore_state`, tvornica i odjeljak `memento:`; mutacije (bez vraćanja, bez provjere imena, bez ključa, bez provjere tipa pohrane, bez provjere ključa) uhvaćene |
| 11 | `inspect.isabstract` nad `Generic*` | 8 apstraktnih, `GenericMemento`/`GenericRepository`/`GenericConverter` nisu |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda; kriterij — odjeljak 13; platforma —
`workflow` radno stablo, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/memento.py`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-MEM-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

Nalaz izvan opsega `workflow`: `blackwattle/src/wattleflow/blackboards/large.py:343` ima isti kvar obnove (`self._fsm.state = …` nad svojstvom bez setter-a).

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/memento.py` (75 linija)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | `WorkflowMemento`, `save_state` i `restore_state` uklonjeni iz klasa workflowa u svih 19 primjera u `blackwattle/examples/workflows` i svih 5 u `blackwattle/dockers/workflow-runner/workflows` (plus 3 suvišna nadjačavanja u procesorima `01`, `10`, `13`); blok `memento:` ostaje samo u YAML-u primjera `05_markdown`. |
| v0.0.5 | 2026-10-04 | Probni prolaz kroz primjer `05_markdown` (bez `WorkflowMemento`, s blokom `memento:`): pad, nastavak u novom procesu, brisanje snimke. Usklađeni [`FRQ-PRC`](FRQ-PRC-processor.md) i [`FRQ-WFL`](FRQ-WFL-workflow.md). |
| v0.0.5 | 2026-10-04 | DEF-MEM-01 zatvoren: pohrana snimke (`MementoStore`, `MemoryMementoStore`, `FileMementoStore`), kontrolna točka i nastavak u `GenericProcessor`, zadani `save_state`/`restore_state` u `GenericWorkflow` i odjeljak `memento:` u tvornici; opt-in, kriteriji 12–17, 54 testa, mutacije. Svi defekti MEM zatvoreni. |
| v0.0.5 | 2026-10-04 | DEF-MEM-02, -03, -04 zatvoreni: `__repr__` ispisuje samo ključeve; `restore_state` validira snimku prije izmjene i obnavlja automat novim `StateMachine` (**uz to je ispravljen skriveni kvar: `self._fsm.state = …` je dizalo `AttributeError` i za ispravnu snimku — obnova nikad nije radila**); `DEF-MEM-03` zatvoren kao svojstvo; kriteriji 10–11, 14 testova. Isti kvar ostaje u `blackwattle/blackboards/large.py:343` (izvan opsega). |
| v0.0.5 | 2026-10-04 | Otvorene stavke u 15 preimenovane u defekte `DEF-MEM-<nn>` i provjerene prema kodu: DEF-MEM-03 ispravljen (nije jedina ne-apstraktna klasa), DEF-MEM-04 dopunjen `AttributeError` pri obnovi. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa i poveznice izvan distribucije `workflow`. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: „IOriginator” zamijenjen klasama iz koda; redoslijed `restore_state` → `get_state` u slijedu; povratni tipovi. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu |
