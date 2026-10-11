<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-WORKFLOW — Workflow: tok koji pokreće procesore i drži njihove upravitelje

| | |
|---|---|
| **Verzija** | v0.0.2 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/workflow.py`: `Workflow`, `WorkflowException` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-FACTORY`](StRS-FACTORY.md) — Workflow Factory koja sastavlja workflow · [`StRS-MANAGER`](StRS-MANAGER.md) — upravitelji koje workflow drži · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — procesori koje workflow pokreće · [`StRS-DRIVER`](StRS-DRIVER.md) — driveri o kojima workflow izvješćuje · [`StRS-CONNECTION`](StRS-CONNECTION.md) — konekcije u upravitelju · [`StRS-MEMENTO`](StRS-MEMENTO.md) — snimka u kojoj se bilježi stanje toka · [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) — prethodni zapis iste komponente |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi workflow, tko ga koristi, što radi kad se pokrene, kako se njegovo stanje bilježi i vraća te što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** workflow kao gotov tok koji drži upravitelje konekcija, drivera i procesora, pokretanje procesora jednog za drugim, praćenje rada toka, izvješće o driverima, bilježenje i vraćanje stanja procesora te greške koje workflow javlja.

**Izvan opsega:** čitanje konfiguracije i sastavljanje konekcija, drivera, procesora, Blackboarda, repozitorija i pipelineova (radi Workflow Factory, [`StRS-FACTORY`](StRS-FACTORY.md)), rad upravitelja ([`StRS-MANAGER`](StRS-MANAGER.md)), rad procesora ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) i mjerenje koje provodi monitor. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| workflow | Workflow | gotov tok podataka: drži upravitelje konekcija, drivera i procesora i pokreće procesore |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije stvara konekcije, drivere i procesore, sastavlja workflow i predaje mu ih |
| konfiguracija | configuration | zapis (npr. YAML datoteka) koji opisuje što tok sadrži i kako je podešen |
| upravitelj | Manager | evidencija objekata jedne vrste po imenu ([`StRS-MANAGER`](StRS-MANAGER.md)); workflow ima upravitelje konekcija, drivera i procesora |
| procesor | Processor | motor koji zapise provodi kroz pipelineove ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| driver | Driver | dio sustava koji razgovara s vanjskim resursom ([`StRS-DRIVER`](StRS-DRIVER.md)) |
| monitor | Monitor | dio koji mjeri rad toka (vrijeme, količinu, resurse) dok se tok izvodi; neobvezan |
| pokretač toka | Initiator | dio sustava ili osoba koja pokreće izvedbu toka, npr. raspoređivač |
| stanje toka | workflow state | snimka u kojoj su zabilježena stanja svih procesora koji se mogu spremiti i vratiti ([`StRS-MEMENTO`](StRS-MEMENTO.md)) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Tok podataka sastoji se od mnogo dijelova: konekcija, drivera, procesora, Blackboarda i pipelineova. Netko ih mora držati na okupu, pokrenuti ispravnim redoslijedom i znati što se s njima dogodilo. Ako to radi svaki tok na svoj način, pokretanje, praćenje i nastavak nakon prekida postaju neujednačeni.

**Rješenje** Workflow je gotov tok. Drži tri upravitelja (konekcije, drivere i procesore) i pokreće procesore jednog za drugim dok svaki ne završi. Uz to prati rad toka kad je praćenje uključeno, izvješćuje što su driveri napravili te može zabilježiti i vratiti stanje svojih procesora.

**Smjer** Workflow sam ne sastavlja dijelove: dobiva ih gotove od Workflow Factory, koja ih stvara iz konfiguracije. Zna samo držati i pokrenuti. Konkretne tokove izrađuje programer. Što svaki procesor radi i kako, zna procesor; workflow ga samo pokreće.

### 2.2 Kontekst

Workflow Factory iz konfiguracije stvara konekcije, drivere i procesore, upisuje ih u upravitelje i predaje ih novom workflowu, a po potrebi mu priključuje monitor. Pokretač toka (npr. raspoređivač) pokreće workflow. Workflow redom pokreće svoje procesore. Ako je uključeno praćenje, monitor mjeri rad toka dok traje izvedba. Po završetku pokretač dobiva ishod ili grešku.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Workflow: Data flow"
caption "Context Diagram"

actor "Initiator" as Initiator
actor "Workflow Factory" as Factory
component "Managers\n(connections, drivers,\nprocessors)" as Managers
component "Processors" as Processors
component "Monitor\n(optional)" as Monitor
component "Drivers" as Drivers

rectangle "Workflow" as Boundary <<system boundary>> {
  component Workflow
}

Factory ..> Workflow : assembles,\nattaches Monitor
Factory ..> Managers : fills
Workflow *-- Managers : holds
Initiator ..> Workflow : runs
Workflow ..> Processors : starts one by one
Workflow ..> Monitor : starts, stops
Workflow ..> Drivers : asks to report
Workflow --> Initiator : outcome / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste workflowa | izrađuje nove tokove i određuje što se događa pri pokretanju |
| Workflow Factory | sastavlja workflow iz konfiguracije | stvara dijelove, upisuje ih u upravitelje, predaje ih workflowu i priključuje monitor |
| Pokretač toka | pokreće workflow | pokreće izvedbu, traži bilježenje i vraćanje stanja i izvješće o driverima te prima ishod ili grešku |
| Procesor | izvodi posao toka | obrađuje zapise kad ga workflow pokrene |
| Monitor | prati tok | mjeri rad toka dok traje izvedba (neobvezno) |
| Driver | izvor ili odredište podataka | na zahtjev izvješćuje što je napravio |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi workflow

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Konfiguracija navodi tok koji treba pokrenuti |
| **Preduvjet** | Workflow Factory je stvorila upravitelje konekcija, drivera i procesora te učitala konfiguraciju |
| **Glavni tijek** | 1. Workflow Factory predaje workflowu konfiguraciju i tri upravitelja. 2. Workflow provjerava da je svaki predani dio ispravne vrste. 3. Workflow preuzima upravitelje. 4. Workflow Factory po potrebi priključuje monitor. |
| **Alternativni tok** | A1. Neki od predanih dijelova nije ispravne vrste: workflow se ne sastavlja, greška. |
| **Ishod** | Workflow sastavljen i spreman za pokretanje; praćenje uključeno ako je zadano |

#### UC2 — Pokreni tok

| | |
|---|---|
| **Akter** | Pokretač toka (primarni), Procesor i Monitor (podupirući) |
| **Okidač** | Pokretač zatraži izvedbu toka |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Ako je priključen monitor, workflow ga pokreće i daje mu putanje na koje driveri zapisuju. 2. Workflow zapisuje imena procesora koje će pokrenuti. 3. Workflow redom pokreće svaki procesor i čeka da završi. 4. Workflow zapisuje broj procesora i ukupan broj obrađenih ciklusa. 5. Ako je priključen monitor, workflow ga zaustavlja. |
| **Alternativni tok** | A1. Procesor javi grešku: izvedba se prekida, preostali procesori se ne pokreću, pokretač prima grešku. Monitor se i tada zaustavlja. |
| **Ishod** | Svi procesori završili, s mjerenjem ako je uključeno; ili izvedba prekinuta greškom |

#### UC3 — Zabilježi stanje toka

| | |
|---|---|
| **Akter** | Pokretač toka |
| **Okidač** | Pokretač želi sačuvati stanje toka |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Pokretač traži bilježenje stanja. 2. Workflow od svakog procesora koji se može spremiti traži njegovo stanje. 3. Workflow vraća zajedničku snimku sa stanjima po imenu procesora. |
| **Alternativni tok** | — |
| **Ishod** | Snimka sa stanjem svih procesora koji se mogu spremiti; procesori koji se ne mogu spremiti nisu u njoj |

#### UC4 — Vrati stanje toka

| | |
|---|---|
| **Akter** | Pokretač toka |
| **Okidač** | Pokretač želi vratiti ranije zabilježeno stanje toka |
| **Preduvjet** | Postoji snimka stanja (UC3) |
| **Glavni tijek** | 1. Pokretač predaje snimku. 2. Workflow provjerava snimku i svaki njezin zapis, prije nego išta vrati. 3. Workflow svakom procesoru vraća njegovo stanje. |
| **Alternativni tok** | A1. Predano nije snimka ili u njoj nema stanja procesora: povrat se odbija, greška workflowa. A2. U snimci je procesor koji workflow nema ili se ne može spremiti: povrat se odbija, greška workflowa. A3. Zapis za procesor nije ispravna snimka: povrat se odbija, greška workflowa. |
| **Ishod** | Procesori vraćeni u zabilježeno stanje; ako je snimka neispravna, nijedan procesor nije dirnut |

#### UC5 — Izvijesti o driverima

| | |
|---|---|
| **Akter** | Pokretač toka (primarni), Driver (podupirući) |
| **Okidač** | Pokretač traži izvješće o radu drivera |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Pokretač traži izvješće. 2. Workflow redom traži od svakog drivera da izvijesti što je napravio. |
| **Alternativni tok** | A1. Driver ne zna izvijestiti: bilježi se da izvješće nije dano, rad se nastavlja. |
| **Ishod** | Svaki driver koji zna izvijestiti je izvijestio; za ostale je zabilježeno da izvješća nema |

#### UC6 — Razvij ili održavaj workflow

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Treba novi tok ili postojeći treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu workflowa i određuje što se događa pri pokretanju. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta workflowa koju Workflow Factory može sastaviti iz konfiguracije |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Workflow: Data flow"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Initiator
actor Processor
actor Monitor
actor Driver
actor Programmer

rectangle "Workflow" as Boundary <<system boundary>> {
    usecase "Sastavi workflow" as UC1
    usecase "Pokreni tok" as UC2
    usecase "Zabilježi stanje toka" as UC3
    usecase "Vrati stanje toka" as UC4
    usecase "Izvijesti o driverima" as UC5
    usecase "Razvij ili održavaj\nworkflow" as UC6
}

Factory -- UC1
Initiator -- UC2
UC2 -- Processor
UC2 -- Monitor
Initiator -- UC3
Initiator -- UC4
Initiator -- UC5
UC5 -- Driver
Programmer -- UC6
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Workflow nema stanja koja prolazi: ili je sastavljen i spreman, ili se izvodi. Dijagram prikazuje normalni tok (od sastavljanja do završetka izvedbe) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Workflow: Data flow"
caption "Flow Chart"

start
:Workflow Factory hands over\nconfiguration and three Managers;
if (every part of the right type?) then (no)
  :<b><color:red>AttributeException</color></b>;
  :error returned to Workflow Factory;
  stop
endif
:Initiator runs the Workflow;
if (Monitor attached?) then (yes)
  :Monitor started;
endif
:Workflow starts each Processor in turn;
if (Processor failed?) then (yes)
  :remaining Processors not started;
  if (Monitor attached?) then (yes)
    :Monitor stopped;
  endif
  :error of the Processor;
  :error returned to Initiator;
  stop
endif
:number of Processors and cycles recorded;
if (Monitor attached?) then (yes)
  :Monitor stopped;
endif
:outcome returned to Initiator;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `AttributeException` | pri sastavljanju neki predani dio nije ispravne vrste (konfiguracija ili jedan od upravitelja) | workflow se ne sastavlja; Workflow Factory prima grešku |
| `WorkflowException` | pri vraćanju stanja predano nije snimka, u njoj nema stanja procesora, procesor ne postoji ili se ne može spremiti, ili zapis za procesor nije snimka | povrat se odbija prije ikakve promjene; pokretač prima grešku |
| greška procesora | procesor pri pokretanju javi grešku (npr. `ProcessorException`, `PipelineException`) | izvedba se prekida, preostali procesori se ne pokreću; pokretač prima grešku procesora; monitor se zaustavlja |
| izvješće nije dano | driver ne zna izvijestiti | bilježi se da izvješća nema; rad se nastavlja |

Workflow bilježi početak izvedbe (s imenima procesora), završetak (s brojem procesora i obrađenih ciklusa) i driver koji nije izvijestio.

### 2.6 Pretpostavke i ograničenja

- Workflow sam ne čita konfiguraciju i ne stvara dijelove: to radi Workflow Factory.
- Procesori se pokreću jedan za drugim, redom u kojem su upisani u upravitelja; workflow ne pokreće dva istodobno.
- Prekid jednog procesora prekida izvedbu cijelog workflowa; workflow ga ne pokušava ponoviti.
- Stanje toka sadrži samo procesore koji se mogu spremiti; konekcije i driveri nisu u njemu.
- Praćenje rada neobvezno je i priključuje ga Workflow Factory.
- Workflow ne čisti upravitelje: oni oslobađaju svoje objekte kad prestanu postojati.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-WFL-01** | Workflow se sastavlja iz konfiguracije i triju upravitelja (konekcije, driveri, procesori); bez njih ispravne vrste se ne sastavlja. |
| **BR-WFL-02** | Workflow sam ne stvara konekcije, drivere ni procesore: dobiva ih od Workflow Factory. |
| **BR-WFL-03** | Workflow pokreće procesore jedan po jedan, redom upisa, i svaki mora završiti prije sljedećeg. |
| **BR-WFL-04** | Greška procesora prekida izvedbu cijelog workflowa; preostali procesori se ne pokreću. |
| **BR-WFL-05** | Ako je priključen monitor, pokreće se prije izvedbe i zaustavlja nakon nje, i kad izvedba završi greškom. |
| **BR-WFL-06** | Workflow bilježi početak izvedbe, završetak (broj procesora i ukupan broj obrađenih ciklusa) i greške. |
| **BR-WFL-07** | Stanje toka obuhvaća samo procesore koji se mogu spremiti. |
| **BR-WFL-08** | Vraćanje stanja provjerava cijelu snimku prije nego što išta vrati: neispravna snimka ne mijenja nijedan procesor. |
| **BR-WFL-09** | Snimka stanja smije sadržavati samo procesore koje workflow ima i koji se mogu vratiti. |
| **BR-WFL-10** | Izvješće o driverima traži se od svakog drivera; driver koji ne zna izvijestiti ne prekida izvješćivanje. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; za njih još nema poveznice na zapise zahtjeva na kakvoću (§6).

| kakvoća | što znači za workflow |
|---|---|
| ujednačenost | svi tokovi se pokreću, prate i vraćaju na isti način |
| otpornost | neispravna snimka ne mijenja nijedan procesor (BR-WFL-08); prekinuta izvedba ne ostavlja monitor uključen (BR-WFL-05) |
| praćenje | workflow bilježi početak, završetak i greške izvedbe te izvješćuje o driverima |
| proširivost | programer može izraditi nove vrste tokova (UC6) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-WFL-*` i naziv sloja u naslovima dijagrama („Data flow") su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. **Tko pokreće tok.** Raspoređivač pokreće workflow nakon što ga Workflow Factory sastavi. Ostali pokretači (skripta toka, osoba) u pregledanom kodu nisu vidljivi, a pozivatelja bilježenja stanja, vraćanja stanja i izvješća o driverima nema. Treba odrediti tko su i kada ih traže (akter „Pokretač toka").
4. Stanje toka (UC3, UC4) odvojeno je od kontrolne točke koju svaki procesor može zapisivati sam ([`StRS-PROCESSOR`](StRS-PROCESSOR.md), [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)). Treba potvrditi odnos ta dva mehanizma i čemu služi stanje toka.
5. Praćenje rada (monitor) u ovom je zapisu opisano samo koliko ga workflow pokreće i zaustavlja. Treba odlučiti treba li zaseban zapis za monitor.
6. Prethodni zapis [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) još nosi staru oznaku i poziva se na obrisani `HLRQ-01-GENERIC-LAYER`; poveznice treba provjeriti.
7. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory koja sastavlja workflow |
| [`StRS-MANAGER`](StRS-MANAGER.md) | upravitelji koje workflow drži |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesori koje workflow pokreće |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driveri o kojima workflow izvješćuje |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | konekcije u upravitelju konekcija |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | snimka u kojoj se bilježi stanje toka |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | pohrana kontrolnih točaka procesora |
| [`FRQ-WFL`](../02-FRQ/FRQ-WFL-workflow.md) | prethodni zapis iste komponente |
| `concrete/workflow.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.2 | 2026-10-11 | Obnovljen: datoteka je sadržavala kopiju zapisa `StRS-FACTORY`, pa je tekst o workflowu ponovno napisan prema kodu `concrete/workflow.py`. Nazivlje *Workflow Factory* usklađeno s ostalim zapisima; snimka umjesto bilješke; poveznica na `StRS-MEMENTO` i `StRS-MEMENTO-STORE`. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/workflow.py`. |
