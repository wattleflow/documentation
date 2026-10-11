<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-WORKFLOW — Workflow: vlasnik glavnog procesa koji pokreće procesore i evidentira greške

| | |
|---|---|
| **Verzija** | v0.0.4 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/workflow.py`: `Workflow` (konačan razred), `WorkflowException` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-FACTORY`](StRS-FACTORY.md) — Workflow Factory koja sastavlja workflow · [`StRS-MANAGER`](StRS-MANAGER.md) — upravitelji koje workflow drži · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — procesori koje workflow pokreće · [`StRS-DRIVER`](StRS-DRIVER.md) — driveri o kojima workflow izvješćuje · [`StRS-CONNECTION`](StRS-CONNECTION.md) — konekcije u upravitelju · [`StRS-MEMENTO`](StRS-MEMENTO.md) — snimka u kojoj se bilježi stanje toka |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda i odluka o dizajnu, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi workflow, tko ga koristi, što radi kad se pokrene, kako evidentira greške i razlikuje ozbiljne od manjih, kako se njegovo stanje bilježi i vraća te što pokretač dobiva kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** workflow kao vlasnik glavnog procesa (izvođenja toka) i svih njegovih podprocesa, pokretanje procesora jednog za drugim, evidencija i razvrstavanje grešaka, prosljeđivanje kardinalnih grešaka pokretaču, praćenje rada toka, izvješće o driverima te bilježenje i vraćanje stanja procesora.

**Izvan opsega:** vrste workflowa (workflow je konačan razred i nema podvrsta), čitanje konfiguracije i sastavljanje konekcija, drivera, procesora, Blackboarda, repozitorija i pipelineova (radi Workflow Factory, [`StRS-FACTORY`](StRS-FACTORY.md)), rad upravitelja ([`StRS-MANAGER`](StRS-MANAGER.md)), rad procesora i greške koje procesor sam obradi ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)), mjerenje koje provodi monitor te ulazna točka koju poziva vanjski pokretač. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| workflow | Workflow | gotov tok podataka: drži upravitelje konekcija, drivera i procesora, pokreće procesore i vlasnik je glavnog procesa |
| konačan razred | final class | razred koji se ne smije nasljeđivati; u kodu označen dekoratorom `@final` (PEP 591) |
| glavni proces | main process | jedno izvođenje toka, od početka do završetka ili prekida; radna jedinica čiji je vlasnik workflow |
| podproces | subprocess | dio glavnog procesa koji workflow pokreće ili traži: rad procesora, praćenje (monitor), izvješće drivera, bilježenje i vraćanje stanja |
| kardinalna greška | cardinal error | greška zbog koje glavni proces ili zatraženi zahtjev ne može biti obavljen; workflow je evidentira i prosljeđuje pokretaču |
| manja greška | minor error | greška uz koju posao ostaje obavljen, ali s lošijim ishodom (npr. praćenje nije uspjelo); workflow je evidentira i nastavlja |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije stvara konekcije, drivere i procesore, sastavlja workflow i predaje mu ih |
| konfiguracija | configuration | zapis (npr. YAML datoteka) koji opisuje što tok sadrži i kako je podešen |
| upravitelj | Manager | evidencija objekata jedne vrste po imenu ([`StRS-MANAGER`](StRS-MANAGER.md)); workflow ima upravitelje konekcija, drivera i procesora |
| procesor | Processor | motor koji dokumente provodi kroz pipelineove ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| driver | Driver | dio sustava koji razgovara s vanjskim resursom ([`StRS-DRIVER`](StRS-DRIVER.md)) |
| monitor | Monitor | dio koji mjeri rad toka (vrijeme, količinu, resurse) dok se tok izvodi; neobvezan |
| pokretač | Runner | vanjski program (npr. cron) koji pokreće workflow; u budućnosti može biti i proces frameworka |
| stanje toka | workflow state | snimka u kojoj su zabilježena stanja svih procesora koji se mogu spremiti i vratiti ([`StRS-MEMENTO`](StRS-MEMENTO.md)) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Tok podataka sastoji se od mnogo dijelova: konekcija, drivera, procesora, Blackboarda i pipelineova. Netko ih mora držati na okupu, pokrenuti ispravnim redoslijedom i znati što se s njima dogodilo. Kad nešto pođe po zlu, netko mora odlučiti je li posao propao ili je samo obavljen lošije, zabilježiti to jednom i javiti pokretaču. Ako to radi svaki dio na svoj način, ista greška pojavi se u zapisima više puta ili nijednom, a pokretač ne zna je li tok uspio.

**Rješenje** Workflow je vlasnik glavnog procesa i svih njegovih podprocesa. Drži tri upravitelja (konekcije, drivere i procesore) i pokreće procesore jednog za drugim dok svaki ne završi. Svaku grešku koja do njega stigne evidentira jednom i razvrstava: **kardinalnu** grešku, zbog koje posao ne može biti obavljen, prosljeđuje pokretaču; **manju** grešku, uz koju je posao ipak obavljen, samo zapisuje i nastavlja. Uz to prati rad toka kad je praćenje uključeno, izvješćuje što su driveri napravili te može zabilježiti i vratiti stanje svojih procesora.

**Smjer** Workflow je **konačan razred**: postoji samo jedna vrsta workflowa, a tokovi se razlikuju isključivo konfiguracijom (procesori, pipelineovi, driveri). Nijedna podvrsta ne može promijeniti način na koji workflow pokreće procesore i postupa s greškama, pa se pravila o greškama ne mogu zaobići. Workflow sam ne sastavlja dijelove: dobiva ih gotove od Workflow Factory. Što svaki procesor radi i kako, zna procesor; greške koje procesor sam obradi ostaju procesorove. Ali svaka greška koja izađe iz podprocesa završava kod workflowa, jer je on jedino mjesto na kojem se zna što ona znači za cijeli tok. Workflow je zadnji sloj frameworka prije pokretača, pa je i jedino mjesto na kojem greška glavnog procesa ulazi u zapise frameworka.

### 2.2 Kontekst

Workflow Factory iz konfiguracije stvara konekcije, drivere i procesore, upisuje ih u upravitelje i predaje ih novom workflowu, a po potrebi mu priključuje monitor. Pokretač pokreće workflow. Workflow redom pokreće procesore. Ako je uključeno praćenje, monitor mjeri rad toka dok traje izvedba. Greške podprocesa dolaze do workflowa; on ih evidentira i pokretaču vraća ishod ili kardinalnu grešku.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Workflow: Data flow"
caption "Context Diagram"

actor "Runner" as Runner
actor "Workflow Factory" as Factory
component "Managers\n(connections, drivers,\nprocessors)" as Managers
component "Processors" as Processors
component "Monitor\n(optional)" as Monitor
component "Drivers" as Drivers
database "Audit records" as Records

rectangle "Workflow" as Boundary <<system boundary>> {
  component Workflow
}

Factory ..> Workflow : assembles,\nattaches Monitor
Factory ..> Managers : fills
Workflow *-- Managers : holds
Runner ..> Workflow : runs
Workflow ..> Processors : starts one by one
Workflow ..> Monitor : starts, stops
Workflow ..> Drivers : asks to report
Processors --> Workflow : outcome / error
Monitor --> Workflow : error
Workflow --> Records : start, end,\nevery error once
Workflow --> Runner : outcome /\ncardinal error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava dijelove toka | izrađuje i mijenja procesore, pipelineove i drivere; workflow ne nasljeđuje i ne mijenja |
| Workflow Factory | sastavlja workflow iz konfiguracije | stvara dijelove, upisuje ih u upravitelje, predaje ih workflowu i priključuje monitor |
| Pokretač | pokreće workflow | pokreće izvedbu, traži bilježenje i vraćanje stanja i izvješće o driverima; prima ishod ili kardinalnu grešku i grešku ne bilježi ponovno u zapise frameworka |
| Procesor | izvodi posao toka | obrađuje dokumente kad ga workflow pokrene; grešku koju ne obradi sam predaje workflowu |
| Monitor | prati tok | mjeri rad toka dok traje izvedba (neobvezno) |
| Driver | izvor ili odredište podataka | na zahtjev izvješćuje što je napravio |
| Operater | čita zapise | iz zapisa doznaje je li tok obavljen, koliko je obradio i što je pošlo po zlu |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi workflow

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Konfiguracija navodi tok koji treba pokrenuti |
| **Preduvjet** | Workflow Factory je stvorila upravitelje konekcija, drivera i procesora te učitala konfiguraciju |
| **Glavni tijek** | 1. Workflow Factory predaje workflowu konfiguraciju i tri upravitelja. 2. Workflow provjerava da je svaki predani dio ispravne vrste. 3. Workflow preuzima upravitelje. 4. Workflow Factory po potrebi priključuje monitor. |
| **Alternativni tok** | A1. Neki od predanih dijelova nije ispravne vrste: workflow se ne sastavlja, greška se vraća Workflow Factoryju. |
| **Ishod** | Workflow sastavljen i spreman za pokretanje; praćenje uključeno ako je zadano |

Sastavljanje nije dio glavnog procesa: glavni proces počinje pokretanjem (UC2).

#### UC2 — Pokreni tok

| | |
|---|---|
| **Akter** | Pokretač (primarni), Procesor i Monitor (podupirući) |
| **Okidač** | Pokretač zatraži izvedbu toka |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Ako je priključen monitor, workflow ga pokreće i daje mu putanje na koje driveri zapisuju. 2. Workflow bilježi početak izvedbe s imenima procesora koje će pokrenuti. 3. Workflow redom pokreće svaki procesor i čeka da završi. 4. Workflow bilježi završetak izvedbe s brojem procesora i ukupnim brojem obrađenih ciklusa. 5. Ako je priključen monitor, workflow ga zaustavlja. 6. Pokretač dobiva ishod. |
| **Alternativni tok** | A1. Procesor javi grešku: to je kardinalna greška. Izvedba se prekida, preostali procesori se ne pokreću, workflow grešku evidentira jednom, zaustavlja monitor i prosljeđuje grešku pokretaču. A2. Monitor se ne može pokrenuti ili zaustaviti, ili se ne može odrediti putanja koju treba mjeriti: to je manja greška. Workflow je evidentira i izvedba se nastavlja, odnosno završava; greška monitora ne prekriva grešku procesora. |
| **Ishod** | Svi procesori završili, s mjerenjem ako je uključeno; ili izvedba prekinuta kardinalnom greškom koja je evidentirana i proslijeđena pokretaču |

#### UC3 — Zabilježi stanje toka

| | |
|---|---|
| **Akter** | Pokretač |
| **Okidač** | Pokretač želi sačuvati stanje toka |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Pokretač traži bilježenje stanja. 2. Workflow od svakog procesora koji se može spremiti traži njegovo stanje. 3. Workflow vraća zajedničku snimku sa stanjima po imenu procesora. |
| **Alternativni tok** | A1. Procesor ne može dati svoje stanje: kardinalna greška za taj zahtjev; workflow je evidentira i prosljeđuje pokretaču. |
| **Ishod** | Snimka sa stanjem svih procesora koji se mogu spremiti; procesori koji se ne mogu spremiti nisu u njoj |

#### UC4 — Vrati stanje toka

| | |
|---|---|
| **Akter** | Pokretač |
| **Okidač** | Pokretač želi vratiti ranije zabilježeno stanje toka |
| **Preduvjet** | Postoji snimka stanja (UC3) |
| **Glavni tijek** | 1. Pokretač predaje snimku. 2. Workflow provjerava snimku i svaki njezin zapis. 3. Svaki procesor priprema povrat svog stanja i pritom ga provjerava. 4. Tek kad su svi procesori spremni, workflow svakom procesoru vraća njegovo stanje. |
| **Alternativni tok** | A1. Predano nije snimka ili u njoj nema stanja procesora: kardinalna greška; povrat se odbija. A2. U snimci je procesor koji workflow nema ili se ne može spremiti: kardinalna greška; povrat se odbija. A3. Zapis za procesor nije ispravna snimka ili ga procesor odbije pri pripremi: kardinalna greška; povrat se odbija. U svim slučajevima workflow grešku evidentira i prosljeđuje pokretaču, a nijedan procesor nije dirnut. A4. Povrat ne uspije nakon što je počeo: kardinalna greška; vidi §6. |
| **Ishod** | Procesori vraćeni u zabilježeno stanje; ako je snimka neispravna, nijedan procesor nije dirnut |

#### UC5 — Izvijesti o driverima

| | |
|---|---|
| **Akter** | Pokretač (primarni), Driver (podupirući) |
| **Okidač** | Pokretač traži izvješće o radu drivera |
| **Preduvjet** | Workflow je sastavljen (UC1) |
| **Glavni tijek** | 1. Pokretač traži izvješće. 2. Workflow redom traži od svakog drivera da izvijesti što je napravio. |
| **Alternativni tok** | A1. Driver ne zna izvijestiti: to nije greška; workflow bilježi da izvješća nema i nastavlja. A2. Izvješće drivera ne uspije: manja greška; workflow je evidentira i nastavlja s ostalim driverima. |
| **Ishod** | Svaki driver koji zna izvijestiti je izvijestio; za ostale je zabilježeno da izvješća nema ili da nije uspjelo |

#### UC6 — Razvij ili održavaj workflow *(ukinut)*

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Treba novi tok ili postojeći treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu workflowa i određuje što se događa pri pokretanju. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta workflowa koju Workflow Factory može sastaviti iz konfiguracije |

*Ukinut 2026-10-11:* workflow je konačan razred (BR-WFL-17), pa vrste workflowa ne postoje. Novi tok nastaje konfiguracijom; programer razvija procesore, pipelineove i drivere (njihovi zapisi). Oznaka UC6 se ne koristi ponovno.

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Workflow: Data flow"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Runner
actor Processor
actor Monitor
actor Driver

rectangle "Workflow" as Boundary <<system boundary>> {
    usecase "Sastavi workflow" as UC1
    usecase "Pokreni tok" as UC2
    usecase "Zabilježi stanje toka" as UC3
    usecase "Vrati stanje toka" as UC4
    usecase "Izvijesti o driverima" as UC5
}

Factory -- UC1
Runner -- UC2
UC2 -- Processor
UC2 -- Monitor
Runner -- UC3
Runner -- UC4
Runner -- UC5
UC5 -- Driver
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Glavni proces počinje pokretanjem i završava ishodom ili kardinalnom greškom. Dijagram prikazuje normalni tok i oba načina na koja workflow postupa s greškom: manju evidentira i nastavlja, kardinalnu evidentira i prosljeđuje.

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
:Runner runs the Workflow;
if (Monitor attached?) then (yes)
  if (Monitor started?) then (no)
    :minor error recorded (WARNING);
  endif
endif
:start of the run recorded (INFO);
:Workflow starts each Processor in turn;
if (Processor failed?) then (yes)
  :remaining Processors not started;
  :cardinal error recorded once (ERROR);
  if (Monitor attached?) then (yes)
    :Monitor stopped\n(its failure recorded as minor);
  endif
  :<b><color:red>error of the Processor</color></b>;
  :error passed on to Runner;
  stop
endif
:end of the run recorded (INFO);
if (Monitor attached?) then (yes)
  :Monitor stopped\n(its failure recorded as minor);
endif
:outcome returned to Runner;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | vrsta | kada nastaje | posljedica |
|---|---|---|---|
| `AttributeException` | — (sastavljanje) | pri sastavljanju neki predani dio nije ispravne vrste | workflow se ne sastavlja; greška se vraća Workflow Factoryju, koji je prosljeđuje pokretaču |
| greška procesora | kardinalna | procesor pri pokretanju javi grešku koju nije sam obradio (npr. `ProcessorException`, `PipelineException`) | izvedba se prekida; workflow grešku evidentira jednom, zaustavlja monitor i prosljeđuje je pokretaču |
| `WorkflowException` | kardinalna | pri bilježenju ili vraćanju stanja zahtjev ne može biti obavljen (neispravna snimka, nepoznat procesor, procesor odbije povrat) | zahtjev se odbija prije ikakve promjene; workflow grešku evidentira i prosljeđuje pokretaču |
| greška monitora | manja | monitor se ne može pokrenuti ili zaustaviti, ili putanja za mjerenje nije određena | workflow je evidentira; izvedba se nastavlja; ne prekriva grešku procesora |
| neuspjelo izvješće drivera | manja | driver pri izvješćivanju javi grešku | workflow je evidentira i nastavlja s ostalim driverima |
| izvješće nije dano | nije greška | driver ne zna izvijestiti | bilježi se da izvješća nema; rad se nastavlja |

Workflow bilježi početak izvedbe (s imenima procesora) i završetak (s brojem procesora i obrađenih ciklusa). Svaku grešku koja do njega stigne evidentira jednom: kardinalnu kao grešku, manju kao upozorenje. Podprocesi grešku koju predaju workflowu ne bilježe ponovno kao grešku, nego ostavljaju samo trag za detaljnu analizu.

### 2.6 Pretpostavke i ograničenja

- Workflow je konačan razred: nema podvrsta, a tokovi se razlikuju samo konfiguracijom.
- Workflow sam ne čita konfiguraciju i ne stvara dijelove: to radi Workflow Factory.
- Procesori se pokreću jedan za drugim, redom u kojem su upisani u upravitelja; workflow ne pokreće dva istodobno.
- Kardinalna greška prekida izvedbu cijelog workflowa; workflow ne pokušava ponoviti posao.
- Workflow nikad ne ovisi o praćenju: neuspjeh monitora ili izlaza nikad nije kardinalna greška.
- Greške koje procesor sam obradi (npr. preskočeni dokument) evidentira procesor; do workflowa stižu samo greške koje procesor ne obradi.
- Stanje toka sadrži samo procesore koji se mogu spremiti; konekcije i driveri nisu u njemu.
- Praćenje rada neobvezno je i priključuje ga Workflow Factory.
- Workflow ne čisti upravitelje: oni oslobađaju svoje objekte kad prestanu postojati.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-WFL-01** | Workflow se sastavlja iz konfiguracije i triju upravitelja (konekcije, driveri, procesori); bez njih ispravne vrste se ne sastavlja. |
| **BR-WFL-02** | Workflow sam ne stvara konekcije, drivere ni procesore: dobiva ih od Workflow Factory. |
| **BR-WFL-03** | Workflow pokreće procesore jedan po jedan, redom upisa, i svaki mora završiti prije sljedećeg. |
| **BR-WFL-04** | Greška procesora je kardinalna: prekida izvedbu cijelog workflowa, a preostali procesori se ne pokreću. |
| **BR-WFL-05** | Ako je priključen monitor, pokreće se prije izvedbe i zaustavlja nakon nje, i kad izvedba završi greškom. |
| **BR-WFL-06** | Workflow bilježi početak izvedbe i završetak (broj procesora i ukupan broj obrađenih ciklusa); to su granice glavnog procesa. |
| **BR-WFL-07** | Stanje toka obuhvaća samo procesore koji se mogu spremiti. |
| **BR-WFL-08** | Vraćanje stanja provjerava cijelu snimku i priprema povrat svakog procesora prije nego što išta vrati: neispravna snimka ne mijenja nijedan procesor. |
| **BR-WFL-09** | Snimka stanja smije sadržavati samo procesore koje workflow ima i koji se mogu vratiti. |
| **BR-WFL-10** | Izvješće o driverima traži se od svakog drivera; driver koji ne zna izvijestiti ili čije izvješće ne uspije ne prekida izvješćivanje ostalih. |
| **BR-WFL-11** | Workflow je vlasnik glavnog procesa i svih podprocesa: svaku grešku koja do njega stigne evidentira točno jednom. |
| **BR-WFL-12** | Kardinalna greška je ona zbog koje glavni proces ili zatraženi zahtjev ne može biti obavljen; workflow je evidentira kao grešku i prosljeđuje pokretaču. |
| **BR-WFL-13** | Manja greška je ona uz koju posao ostaje obavljen, s lošijim ishodom; workflow je evidentira kao upozorenje i nastavlja. Pokretač je ne dobiva kao grešku. |
| **BR-WFL-14** | Neuspjeh monitora ili izlaza uvijek je manja greška: workflow ne ovisi o praćenju, a greška praćenja ne prekriva grešku procesora. |
| **BR-WFL-15** | Podproces grešku koju predaje workflowu ne evidentira kao grešku; ostavlja samo trag za detaljnu analizu. Greška ulazi u zapise frameworka kod workflowa, jednom. |
| **BR-WFL-16** | Pokretač kardinalnu grešku ne evidentira ponovno u zapise frameworka: ona je u njima već kod workflowa. |
| **BR-WFL-17** | Workflow je konačan razred: ne smije se nasljeđivati. Tokovi se razlikuju samo konfiguracijom, a pravila o pokretanju i greškama (BR-WFL-03 do BR-WFL-16) ne može promijeniti nijedna podvrsta. |

## 4. Nefunkcionalni zahtjevi

| kakvoća | što znači za workflow | NFRQ |
|---|---|---|
| ujednačenost | svi tokovi se pokreću, prate i vraćaju na isti način | — |
| čitljivost zapisa | jedna greška daje jedan zapis greške; razina zapisa kaže je li posao obavljen (BR-WFL-11 do BR-WFL-15) | [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md), [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) |
| vlasništvo i opseg zapisa | workflow kao vlasnik glavnog procesa bilježi njegove granice s brojačima (BR-WFL-06) | [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) |
| otpornost | neispravna snimka ne mijenja nijedan procesor (BR-WFL-08); prekinuta izvedba ne ostavlja monitor uključen (BR-WFL-05); praćenje ne može srušiti tok (BR-WFL-14) | [`NFRQ-SCR-12`](../03-NFRQ/NFRQ-SCR-12-instance-availability.md) |
| razdvajanje odgovornosti | workflow provjerava samo vlastite ulaze (UC1); greške koje procesor obradi ostaju procesorove | [`NFRQ-ORG-15`](../03-NFRQ/NFRQ-ORG-15-separation-of-concerns.md) |
| povjerljivost zapisa | zapisi o greškama nose identitet i uzrok, ne sadržaj dokumenata ni vjerodajnice | [`NFRQ-SCR-06`](../03-NFRQ/NFRQ-SCR-06-audit-confidentiality.md), [`NFRQ-SCR-10`](../03-NFRQ/NFRQ-SCR-10-secrets-enter-at-the-process-boundary.md) |
| proširivost | novi tok nastaje konfiguracijom; programer proširuje procesore, pipelineove i drivere, a ne workflow | — |
| cjelovitost ponašanja | workflow je konačan razred, pa nijedna podvrsta ne može zaobići pravila o greškama (BR-WFL-17); zabrana je zaštita od pogreške, ne sigurnosna granica | [`NFRQ-SCR-05`](../03-NFRQ/NFRQ-SCR-05-adversary-model.md) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

**Odluke**

1. Djeca ovog zapisa (SyRS) nisu određena. Naziv sloja u naslovima dijagrama („Data flow") je prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. **Iznimka granice frameworka u NFRQ-OBS-01.** OBS-01 c.1 zabranjuje da isti sloj evidentira grešku i proslijedi je. Workflow je zadnji sloj frameworka prije pokretača, pa evidentira kardinalnu grešku i prosljeđuje je (BR-WFL-12); jedan uzrok i dalje daje jedan zapis greške (BR-WFL-15, BR-WFL-16). Iznimku treba unijeti u OBS-01 dokumentiranom izmjenom.
4. **Tko traži stanje i izvješće.** Pokretač je vanjski program (npr. cron), u budućnosti i proces frameworka. Nije određeno tko traži bilježenje i vraćanje stanja te izvješće o driverima, ni kada.
5. **Povrat koji ne uspije nakon početka** (UC4 A4): ako primjena povrata padne usred niza, dio procesora je već vraćen. Treba odlučiti kako se postupa (npr. vraćanje prethodnog stanja ili oznaka da je stanje toka nedosljedno).
6. Stanje toka (UC3, UC4) odvojeno je od kontrolne točke koju svaki procesor može zapisivati sam ([`StRS-PROCESSOR`](StRS-PROCESSOR.md), [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)). Treba potvrditi odnos ta dva mehanizma i čemu služi stanje toka.
7. Praćenje rada (monitor) u ovom je zapisu opisano samo koliko ga workflow pokreće i zaustavlja. Treba odlučiti treba li zaseban zapis za monitor.
8. **Razgraničenje s procesorom.** Koje greške procesor obradi sam (manje greške na razini dokumenta), a koje predaje workflowu, propisuje [`StRS-PROCESSOR`](StRS-PROCESSOR.md); taj zapis treba uskladiti s BR-WFL-11 i BR-WFL-15.
9. **Zamrzavanje sastava.** Konačan razred sprječava podvrste, ali ne i zamjenu dijelova postojećeg workflowa (upravitelja, monitora) nakon sastavljanja. Treba odlučiti zamrzava li se i sastav: monitor bi se tada predavao pri sastavljanju ili priključivao najviše jednom, prije prvog pokretanja; zaključavanje sadržaja upravitelja odluka je za [`StRS-MANAGER`](StRS-MANAGER.md).
10. **Vrsta workflowa u konfiguraciji.** Konfiguracija toka navodi `type`, a Workflow Factory po njemu bira razred. Uz konačan razred `type` može imenovati samo `Workflow`; treba odlučiti ostaje li ključ (radi dosljednosti s ostalim unosima) ili se uklanja.

**Neostvareno ili neusklađeno** (razlika između koda i ovog dizajna; izrada zapisa kod ne mijenja)

11. **Evidencija kardinalne greške.** Kod (`execute`, `run`) grešku procesora ne hvata i ne evidentira, nego je samo prosljeđuje; BR-WFL-11 i BR-WFL-12 traže jedan zapis greške kod workflowa.
12. **Monitor izvan zaštite.** Pokretanje monitora je izvan bloka koji štiti izvedbu, pa njegov neuspjeh prekida tok; zaustavljanje je u završnom bloku, pa njegova greška može prekriti grešku procesora ili srušiti uspješnu izvedbu (BR-WFL-14).
13. **Putanja za mjerenje.** Neuspjelo određivanje putanje koju monitor mjeri guta se bez traga (BR-WFL-13: manja greška se evidentira).
14. **Izvješće drivera.** Driver bez izvješća bilježi se na razini koja je rezervirana za granice radnih jedinica i ulaz u lanac; neuspjelo izvješće prekida izvješćivanje ostalih drivera (BR-WFL-10, BR-WFL-13).
15. **Podprocesi.** Treba provjeriti bilježe li procesor, pipeline i ostali slojevi grešku kao grešku prije nego što je predaju workflowu (BR-WFL-15).
16. **Workflow nije konačan razred.** U kodu je `Workflow` apstraktni razred (`ABC`) bez dekoratora `@final`, pa se može nasljeđivati (BR-WFL-17).

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
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md), [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md), [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md), [`NFRQ-SCR-06`](../03-NFRQ/NFRQ-SCR-06-audit-confidentiality.md), [`NFRQ-SCR-10`](../03-NFRQ/NFRQ-SCR-10-secrets-enter-at-the-process-boundary.md), [`NFRQ-SCR-12`](../03-NFRQ/NFRQ-SCR-12-instance-availability.md), [`NFRQ-ORG-15`](../03-NFRQ/NFRQ-ORG-15-separation-of-concerns.md) | kakvoće |
| `concrete/workflow.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.4 | 2026-10-11 | Workflow je konačan razred (`@final`, BR-WFL-17): nema podvrsta, tokovi se razlikuju samo konfiguracijom. UC6 ukinut; programer razvija procesore, pipelineove i drivere. Nova kakvoća *cjelovitost ponašanja* (SCR-05: zaštita od pogreške, ne sigurnosna granica). *Otvoreno*: zamrzavanje sastava, ključ `type` u konfiguraciji toka, neusklađenost koda (`ABC` bez `@final`). |
| v0.0.3 | 2026-10-11 | Dizajn prema odluci autora: workflow je vlasnik glavnog procesa i svih podprocesa, svaku grešku koja do njega stigne evidentira jednom i razvrstava na kardinalne (evidentira i prosljeđuje pokretaču) i manje (evidentira i nastavlja). Nova pravila BR-WFL-11 do BR-WFL-16; BR-WFL-04, -06, -08 i -10 preformulirani; UC2 do UC5 dopunjeni alternativnim tokovima; tablica grešaka s vrstom greške; dijagrami ažurirani. Uloga *Runner* (vanjski program, npr. cron) umjesto *Initiator*. Sastavljanje nije dio glavnog procesa. §4 povezan s NFRQ zapisima. Uklonjene poveznice na prethodni zapis. *Otvoreno*: iznimka granice frameworka u OBS-01, djelomičan povrat stanja, razgraničenje s procesorom, pet neusklađenosti koda. |
| v0.0.2 | 2026-10-11 | Obnovljen: datoteka je sadržavala kopiju zapisa `StRS-FACTORY`, pa je tekst o workflowu ponovno napisan prema kodu `concrete/workflow.py`. Nazivlje *Workflow Factory* usklađeno s ostalim zapisima; snimka umjesto bilješke; poveznica na `StRS-MEMENTO` i `StRS-MEMENTO-STORE`. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/workflow.py`. |