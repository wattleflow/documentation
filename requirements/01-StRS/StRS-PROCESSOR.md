<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-PROCESSOR — Generički procesor: obrada dokumenta kroz pipelineove

| | |
|---|---|
| **Verzija** | v0.0.6 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/processor.py`: `GenericProcessor`, `ProcessorState`, `ProcessorAction` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — procesor je vlasnik Blackboarda · [`StRS-PIPELINE`](StRS-PIPELINE.md) — procesor pokreće pipelineove · [`StRS-MANAGER`](StRS-MANAGER.md) — procesorima upravlja upravitelj procesora · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja procesor · [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) — stroj stanja · [`StRS-MEMENTO`](StRS-MEMENTO.md), [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) — snimka i pohrana kontrolnih točaka · [`StRS-DOCUMENT`](StRS-DOCUMENT.md) — dokument koji procesor obrađuje · [`StRS-DRIVER`](StRS-DRIVER.md) — driver koji procesor može koristiti · [`StRS-REPOSITORY`](StRS-REPOSITORY.md) — repozitoriji u koje završavaju obrađeni dokumenti · [`StRS-WORKFLOW`](StRS-WORKFLOW.md) — workflow koji pokreće procesore |
| **Dijagrami** | inline (§2.2, §2.4, §2.5, §2.6) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi generički procesor, tko ga koristi, kako obrađuje dokumente, u kojim se stanjima može nalaziti i što se događa kad obrada ne uspije.

### 1.2 Opseg

**U opsegu:** sastavljanje procesora i registracija Blackboarda, obrada dokumenta kroz zadane pipelineove, stanja procesora, nastavak prekinute obrade i greške koje procesor javlja.

**Izvan opsega:** dohvat podataka iz izvora, transformacije unutar pipelineova, rad Blackboarda (opisan u [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md)) i pohrana rezultata u repozitorije. Procesor te mehanizme pokreće ili koristi, ali ne opisuje. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja procesor i registrira mu Blackboard |
| Workflow | Workflow | dio sustava koji izvršava tok: pokreće procesore i prima ishod ili grešku |
| procesor | Processor | motor koji provodi dokumente jedan po jedan kroz zadane pipelineove |
| dokument | document | strukturirana ili nestrukturirana jedinica podataka koja se obrađuje |
| generator | — | mehanizam koji procesoru predaje dokumente jedan po jedan, pa cijeli skup nikad nije u memoriji odjednom |
| pipeline | Pipeline | niz transformacija kroz koji prolazi svaki dokument |
| Blackboard | Blackboard | međuspremnik u kojem pipelineovi ostavljaju rezultate; procesor mu je vlasnik |
| repozitorij | Repository | mjesto trajne pohrane rezultata, povezano s Blackboardom |
| predaja | flush | nalog Blackboardu da rezultate preda repozitorijima |
| ciklus | cycle | jedan dokument koji je prošao kroz sve pipelineove |
| kontrolna točka | checkpoint | zabilježen napredak obrade, iz kojeg se prekinuta obrada može nastaviti; čuva se u pohrani snimki ([`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)) |
| stanje | state | trenutna faza rada procesora (§2.5) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Tok podataka mora pouzdano provesti velik broj dokumenta kroz niz transformacija, bez obzira na to odakle dokumenti dolaze i kamo se rezultati pohranjuju. Ako svaki tok to rješava na svoj način, obrada postaje neujednačena, a prekinut posao teško je nastaviti bez gubitka ili ponavljanja dokumenta.

**Rješenje** Generički procesor osigurava mehanizam obrade podataka kroz listu zadanih transformacija (pipelineova). Procesor je poput motora u automobilu: ne zna što se događa u prometu, kako radi hladnjak ili mjenjač. Zna samo, na zahtjev pozivatelja i prema zadanoj konfiguraciji, obraditi zaprimljene dokumente jedan po jedan kroz svaki pipeline, dok posao ne privede kraju.

**Smjer** Procesor je vitalan za protok podataka. Prema zadanoj konfiguraciji odabire koje će dokumente obraditi i obrađuje ih jedan po jedan do kraja. Dokumente preuzima generatorom, pa nikad ne mora držati cijeli skup podataka odjednom. Svojim strojem stanja (*state machine*) osigurava ispravan redoslijed rada. Registrirani je vlasnik Blackboarda kroz koji prolaze tokovi podataka, ali o tim tokovima nema nikakvog znanja. Dohvat podataka i pohrana sirovih ili transformiranih podataka posao su drugih mehanizama.

### 2.2 Kontekst

Workflow pokreće procesor. Procesor iz izvora preuzima dokumente jedan po jedan i svaki provodi kroz zadane pipelineove. Rezultati završavaju na Blackboardu, a procesor mu nalaže da ih preda repozitorijima. Ako je uključeno bilježenje napretka, procesor zapisuje kontrolne točke iz kojih se prekinuta obrada može nastaviti. Procesor prije pokretanja iz konfiguracije sastavlja Workflow Factory, koji mu registrira i Blackboard. Po završetku Workflow dobiva ishod ili grešku.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Processor: Data processing"
caption "Context Diagram"

actor "Workflow Factory" as Factory
actor "Workflow" as Workflow
database "Data source" as Source
component Pipelines
component Blackboard
database Repositories
database "Checkpoint store\n(optional)" as CheckPoint

rectangle "Data Processing" as Boundary <<system boundary>> {
  component Processor
}

Factory ..> Processor : assembles,\nregisters Blackboard
Workflow ..> Processor : starts
Source --> Processor : documents\n(one at a time)
Processor ..> Pipelines : runs each document
Processor ..> Blackboard : owns, flushes
Blackboard --> Repositories : stores
Processor ..> CheckPoint : saves / reads progress
Processor --> Workflow : outcome / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava procesore | izrađuje nove vrste procesora (izvor i odabir dokumenta) i održava postojeće |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja procesor | sastavlja procesor s pipelineovima i registrira mu Blackboard |
| Workflow | izvršava tok | pokreće procesor i prima ishod ili grešku |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi procesor

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Pokretanje toka |
| **Preduvjet** | Konfiguracija navodi zadani procesor, njegove pipelineove i Blackboard |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije čita procesor. 2. Procesor se sastavlja s odabranim načinom predaje i, ako ga konfiguracija navodi, s driverom koji Workflow Factory dohvaća od upravitelja drivera. 3. Workflow Factory procesoru registrira pipelineove redoslijedom izvođenja. 4. Ako je zadano, uključuje se bilježenje napretka. |
| **Alternativni tok** | Konfiguracija nije ispravna (npr. bilježenje napretka uz predaju tek na kraju obrade): greška procesora, procesor se ne sastavlja. |
| **Ishod** | Procesor sastavljen, u stanju IDLE |

#### UC2 — Registriraj Blackboard

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Workflow Factory je sastavio Blackboard prema konfiguraciji toka |
| **Preduvjet** | Procesor je sastavljen (UC1) |
| **Glavni tijek** | 1. Workflow Factory predaje Blackboard procesoru. 2. Procesor provjerava da je predan Blackboard. 3. Procesor ga preuzima kao vlasnik. |
| **Alternativni tok** | Predano nije Blackboard: procesor odbija registraciju i javlja grešku. |
| **Ishod** | Procesor je vlasnik Blackboarda i može pokrenuti obradu |

Procesor ima jedan Blackboard: nova registracija zamjenjuje prethodni. Isti slučaj iz perspektive Blackboarda opisan je u [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), UC2.

#### UC3 — Pokreni procesor

| | |
|---|---|
| **Akter** | Workflow |
| **Okidač** | Workflow izvršava tok |
| **Preduvjet** | Procesor je sastavljen (UC1) i ima registriran Blackboard (UC2) |
| **Glavni tijek** | 1. Workflow pokreće procesor. 2. Procesor preuzima sljedeći dokument. 3. Provodi ga redom kroz sve pipelineove. 4. Nalaže Blackboardu predaju rezultata. 5. Bilježi napredak, ako je uključeno. 6. Ponavlja korake 2–5 dok ima dokumenta. 7. Briše kontrolnu točku i javlja Workflowu da je obrada završena. |
| **Alternativni tok** | A1. Nedostaje Blackboard ili pipeline: obrada ne počinje, greška procesora. A2. Zapis ne prođe kroz pipeline: obrada se prekida, procesor prelazi u stanje FAILED, greška pipelinea. A3. Neočekivani kvar: obrada se prekida, procesor prelazi u stanje FAILED, greška procesora. |
| **Ishod** | Svi dokumenti obrađeni; procesor u stanju COMPLETED |

#### UC4 — Nastavi prekinutu obradu

| | |
|---|---|
| **Akter** | Workflow |
| **Okidač** | Workflow ponovno pokreće procesor nakon prekida |
| **Preduvjet** | Bilježenje napretka je uključeno i postoji kontrolna točka |
| **Glavni tijek** | 1. Procesor čita kontrolnu točku. 2. Preskače već obrađene dokumente. 3. Nastavlja obradu (UC3) od prvog dokumenta nakon zadnje kontrolne točke. |
| **Alternativni tok** | Kontrolna točka ne može se pročitati ili je neispravna, ili izvor ima manje dokumenta nego što je zabilježeno: obrada se ne nastavlja, greška procesora. |
| **Ishod** | Obrada nastavljena od zadnje kontrolne točke |

UC4 proširuje UC3: izvodi se samo kad je bilježenje napretka uključeno i kontrolna točka postoji.

#### UC5 — Razvij ili održavaj procesor

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Tok treba novu vrstu procesora ili postojeća treba prilagodbe |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu procesora i određuje odakle dolaze dokumenti i koji se obrađuju. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta procesora koju workflow može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Processor: Data processing"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Workflow
actor Programmer

rectangle "Processor" as Boundary <<system boundary>> {
    usecase "Sastavi procesor" as UC1
    usecase "Registriraj Blackboard" as UC2
    usecase "Pokreni procesor" as UC3
    usecase "Nastavi prekinutu obradu" as UC4
    usecase "Razvij ili održavaj\nprocesor" as UC5
}

Factory -- UC1
Factory -- UC2
Workflow -- UC3
Workflow -- UC4
Programmer -- UC5
UC4 ..> UC3 : <<extend>>
@enduml
```

### 2.5 Stanja procesora

Procesor u svakom trenutku ima jedno stanje i prelazi u drugo samo na propisani događaj ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md)). Time se sprječava, na primjer, nastavak obrade koja je već završena.

| stanje | značenje |
|---|---|
| IDLE | procesor je sastavljen i čeka pokretanje |
| STATE_LOADED | vraćen je zabilježeni napredak; procesor čeka nastavak |
| RUNNING | obrada dokumenta je u tijeku |
| COMPLETED | svi dokumenti su obrađeni |
| FAILED | obrada je prekinuta greškom; može se nastaviti iz kontrolne točke |

```plantuml
@startuml state-diagram
!include requirements/styles/wattleflow.puml

title "Processor: Data processing"
caption "State Diagram"

[*] --> IDLE
IDLE --> STATE_LOADED : load
IDLE --> RUNNING : start
STATE_LOADED --> RUNNING : start
RUNNING --> RUNNING : next item /\ncycle completed
RUNNING --> COMPLETED : records processed
RUNNING --> FAILED : fail
FAILED --> STATE_LOADED : load
COMPLETED --> COMPLETED : store
FAILED --> FAILED : store
COMPLETED --> [*]
@enduml
```

### 2.6 Tok rada i praćenje grešaka

Dijagram prikazuje normalni tok (od sastavljanja do završetka obrade) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Processor: Data processing"
caption "Flow Chart"

start
:Workflow Factory assembles processor\nfrom configuration;
if (configuration valid?) then (no)
  :<b><color:red>ProcessorException</color></b>;
  stop
endif
:Workflow Factory registers Blackboard;
if (Blackboard accepted?) then (no)
  :<b><color:red>ProcessorException</color></b>;
  stop
endif
:Workflow starts processor;
if (blackboard and pipelines set?) then (no)
  :<b><color:red>ProcessorException</color></b>;
  :error returned to Workflow;
  stop
endif
if (checkpoint found?) then (yes)
  :restore progress,\nskip processed documents;
  if (restore valid?) then (no)
    :<b><color:red>ProcessorException</color></b>;
    :error returned to Workflow;
    stop
  endif
endif
while (next document?) is (yes)
  :run document through each pipeline;
  if (pipeline failed?) then (yes)
    :state FAILED;
    :<b><color:red>PipelineException</color></b>;
    :error returned to Workflow;
    stop
  endif
  :flush Blackboard (per document);
  :save checkpoint (if enabled);
endwhile (no)
:flush Blackboard (at the end, if set);
:clear checkpoint;
:state COMPLETED;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ProcessorException` | konfiguracija nije ispravna; pri registraciji nije predan Blackboard; nedostaje Blackboard ili pipeline; kontrolna točka ne može se pročitati ili vratiti; neočekivani kvar izvan obrade dokumenta | procesor se ne sastavlja, registracija se odbija, ili obrada ne počinje ili se prekida; Workflow prima grešku |
| `PipelineException` | dokument ne prođe kroz pipeline | obrada se prekida, procesor prelazi u FAILED; Workflow prima grešku |
| upozorenje | kontrolna točka ne može se zapisati ili obrisati; zatražena radnja koju procesor ne podržava | rad se nastavlja; događaj se bilježi |

Svaka greška nosi uzrok. Procesor bilježi svoje događaje: pokretanje, svaki obrađeni dokument, završetak i greške.

### 2.7 Pretpostavke i ograničenja

- Procesor sam ne dohvaća ni ne pohranjuje podatke: dokumente mu daje izvor, a pohranu obavljaju repozitoriji Blackboarda.
- Procesor ne zna što pipelineovi rade sa dokumentima ni koji tokovi podataka prolaze kroz Blackboard.
- Zapisi se obrađuju slijedno, jedan po jedan.
- Bilježenje napretka nije obavezno; bez njega procesor ne zapisuje i ne čita kontrolne točke.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-PRC-01** | Procesor obrađuje dokumente jedan po jedan; svaki dokument prolazi kroz sve zadane pipelineove, redoslijedom kojim su registrirani. |
| **BR-PRC-02** | Procesor ne počinje obradu bez Blackboarda i barem jednog pipelinea. |
| **BR-PRC-03** | Koje dokumente obraditi određuje konfiguracija procesora; odakle dokumenti dolaze i kamo se pohranjuju, procesor ne zna. |
| **BR-PRC-04** | Neuspjeh jednog dokumenta prekida cijelu obradu; procesor prelazi u stanje neuspjeha iz kojeg se obrada može nastaviti. |
| **BR-PRC-05** | Nakon svakog obrađenog dokumenta procesor nalaže Blackboardu predaju rezultata, osim ako je zadano da se predaja obavi jednom, na kraju obrade. |
| **BR-PRC-06** | Bilježenje napretka moguće je samo uz predaju nakon svakog dokumenta; inače bi se nepredani dokumenti pri nastavku preskočili. |
| **BR-PRC-07** | Nastavak obrade počinje od prvog dokumenta nakon zadnje kontrolne točke; dokumenti obrađeni nakon nje mogu se obraditi ponovno. Ako izvor ima manje dokumenta nego što je zabilježeno, nastavak se odbija. |
| **BR-PRC-08** | Uspješno završena obrada briše kontrolnu točku, kako sljedeće pokretanje ne bi preskočilo dokumente. |
| **BR-PRC-09** | Kvar pri zapisu ili brisanju kontrolne točke ne prekida obradu; bilježi se kao upozorenje. |
| **BR-PRC-10** | Procesor mijenja stanje samo propisanim prijelazima (§2.5). |
| **BR-PRC-11** | Procesor može repozitorijima proslijediti pravila pohrane za svoj skup dokumenta; pravila primjenjuju repozitoriji. |
| **BR-PRC-12** | Kad procesor prestane postojati, čisti svoj Blackboard. |
| **BR-PRC-13** | Blackboard procesoru registrira Workflow Factory. Procesor ima jedan Blackboard; nova registracija zamjenjuje prethodni. |

## 4. Nefunkcionalni zahtjevi

Povezivanje s NFRQ zapisima je otvoreno (§6). Iz narativa i koda proizlaze ove kakvoće:

| kakvoća | što znači za procesor |
|---|---|
| otpornost | prekinuta obrada nastavlja se iz kontrolne točke (BR-PRC-07) |
| skromna potrošnja memorije | dokumenti se preuzimaju jedan po jedan, bez učitavanja cijelog skupa |
| praćenje | procesor bilježi pokretanje, obrađene dokumente, završetak i greške |
| proširivost | programer može izraditi nove vrste procesora za specifične tokove podataka |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-PRC-*` su prijedlog.
2. Naziv sloja u naslovima dijagrama („Data processing") je prijedlog.
3. Pipelineove procesoru registrira Workflow Factory ([`BR-FCT-10`](StRS-FACTORY.md#3-poslovna-pravila)); ovdje je to korak sastavljanja (UC1). Treba odlučiti je li registracija pipelineova zaseban korisnički slučaj, kao registracija Blackboarda (UC2).
4. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.
5. Zadana predaja: procesor zadano nalaže predaju nakon svakog dokumenta (BR-PRC-05), a Blackboard je zadano odgađa ([`BR-BLC-07`](StRS-BLACKBOARD.md#3-poslovna-pravila)). Treba potvrditi koje pravilo vrijedi kad nijedno nije posebno podešeno.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard čiji je procesor vlasnik |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | pipelineovi koje procesor pokreće |
| [`StRS-MANAGER`](StRS-MANAGER.md) | upravitelj procesora |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja procesor |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | stroj stanja |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | snimka stanja |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | pohrana kontrolnih točaka |
| `concrete/processor.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.6 | 2026-10-11 | Poveznice na StRS-MEMENTO i StRS-MEMENTO-STORE. |
| v0.0.5 | 2026-10-11 | Prema kodu Workflow Factoryja: pipelineove registrira Workflow Factory, a procesor po potrebi dobiva driver od njega (UC1); otvoreno pitanje o registraciji pipelineova suženo. Poveznice na StRS-FACTORY i StRS-STATE-MACHINE. |
| v0.0.4 | 2026-10-11 | Nazivi usklađeni s ostalim StRS-ovima: procesor sastavlja i Blackboard mu registrira Workflow Factory (UC1, UC2, BR-PRC-13), a pokreće ga Workflow (UC3, UC4); jedinica obrade zove se *dokument* umjesto *zapis*; dodane poveznice na StRS-PIPELINE i StRS-MANAGER; alias granice u dijagramu korisničkih slučajeva. |
| v0.0.3 | 2026-10-10 | Oznaka bez broja (`StRS-PROCESSOR`). Workflow pokreće procesor: UC3 postaje *Pokreni procesor*, akter „Pozivatelj" uklonjen iz dokumenta. Granica sustava u kontekstnom dijagramu dobila alias (sukob naziva s komponentom). |
| v0.0.2 | 2026-10-10 | Usklađeno s vještinom `wattleflow-strs` i sa StRS-om Blackboarda: Workflow sastavlja procesor iz konfiguracije (UC1) i registrira mu Blackboard (novi UC2); programer razvija i održava procesore (novi UC5); dodano pravilo BR-PRC-13; jedinstveni naslovi dijagrama; narativ *Problem* potvrđen. |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz narativa autora i koda `concrete/processor.py`. |