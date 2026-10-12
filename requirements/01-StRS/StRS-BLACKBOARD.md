<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-BLACKBOARD — Blackboard: privremena pohrana dokumenata tijekom obrade

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/blackboard.py`: `GenericBlackboard`, `BlackboardState`, `BlackboardAction` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — procesor je vlasnik Blackboarda · [`StRS-PIPELINE`](StRS-PIPELINE.md) — pipelineovi čitaju i pišu dokumente · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja Blackboard · [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) — stroj stanja · [`StRS-DOCUMENT`](StRS-DOCUMENT.md) — dokument koji Blackboard drži · [`StRS-DRIVER`](StRS-DRIVER.md) — driver koji koriste strategije repozitorija · [`StRS-REPOSITORY`](StRS-REPOSITORY.md) — repozitoriji kojima Blackboard predaje dokumente · [`StRS-STRATEGY`](StRS-STRATEGY.md) — strategija stvaranja koju Blackboard izvodi |
| **Dijagrami** | inline (§2.2, §2.4, §2.5, §2.6) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi Blackboard, tko ga koristi, kako čuva dokumente tijekom obrade i predaje ih na trajnu pohranu, u kojim se stanjima može nalaziti i što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** privremeno čuvanje dokumenata tijekom obrade, stvaranje, čitanje i pisanje dokumenata, predaja dokumenata repozitorijima, stanja Blackboarda i greške koje javlja.

**Izvan opsega:** kako strategija stvaranja stvara dokumente, kako repozitoriji trajno pohranjuju podatke (uključujući njihove strategije čitanja i pisanja, opisane u dokumentaciji repozitorija) i što pipelineovi rade s dokumentima. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja Blackboard i registrira ga kod procesora |
| Blackboard | Blackboard | zajednički međuspremnik u kojem dokumenti žive dok traje obrada |
| dokument | document | strukturirana ili nestrukturirana jedinica podataka na Blackboardu |
| platno | canvas | sadržaj Blackboarda: dokumenti koji još nisu predani |
| sesija | session | razdoblje jedne obrade, od pokretanja do završetka |
| procesor | Processor | motor obrade; vlasnik Blackboarda ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| pipeline | Pipeline | niz transformacija koji čita i piše dokumente na Blackboardu |
| strategija stvaranja | Create strategy | strategija registrirana na Blackboardu koja stvara nove dokumente; izvodi se kad procesor zatraži novi dokument, a Blackboard ne zna kako radi |
| repozitorij | Repository | mjesto trajne pohrane; prima dokumente pri predaji |
| predaja | flush | slanje svih dokumenata s platna svim repozitorijima i pražnjenje platna |
| stanje | state | trenutna faza rada Blackboarda (§2.5) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Pipelineovi tijekom obrade trebaju dijeliti dokumente: jedan ih stvara ili mijenja, drugi nastavlja raditi s njima. Kad bi svaki pipeline sam čitao iz trajne pohrane i pisao u nju, transformacije bi bile složenije i sporije, a pohrana raspršena po pipelineovima.

**Rješenje** Blackboard je dio mehanizma postojanosti podataka (*data persistence*). Služi privremenoj pohrani dokumenata radi jednostavnijih transformacija podataka: pipelineovi kroz njega dijele strukturirane i nestrukturirane dokumente tijekom trajanja sesije, odnosno procesa.

**Smjer** Blackboard se podešava konfiguracijom, a njegov rad nadzire vlastiti stroj stanja (*state machine*). Ima jednu registriranu strategiju stvaranja, o kojoj ne zna ništa; procesor je poziva kroz Blackboard kad treba novi dokument. Ima jedan ili više registriranih repozitorija u koje se podaci trajno pohranjuju, a ovisno o konfiguraciji pohrana se obavlja tijekom ili nakon obrade. Blackboard osigurava čitanje i pisanje dokumenata tijekom sesije te njihovu predaju repozitorijima.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja zadani Blackboard i registrira ga kod procesora. Procesor je vlasnik Blackboarda: poziva ga da stvori dokument, nalaže predaju i na kraju ga čisti. Pipelineovi tijekom obrade čitaju i pišu dokumente. Na poziv procesora Blackboard izvodi registriranu strategiju stvaranja, a pri predaji šalje dokumente repozitorijima i procesoru javlja jesu li ih svi potvrdili.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Blackboard: Data persistence"
caption "Context Diagram"

actor "Workflow Factory" as Factory
component "Processor" as Processor
component "Pipelines" as Pipelines
component "Create strategy" as CS
database "Repositories" as Repo

rectangle "Blackboard" as Boundary <<system boundary>> {
  component Blackboard
}

Factory ..> Blackboard : configures
Factory ..> Processor : registers Blackboard
Processor ..> Blackboard : owns;\ncreate, flush, clean
Pipelines ..> Blackboard : read / write / delete
Blackboard ..> CS : executes on create
Blackboard --> Repo : flush documents
Blackboard --> Processor : flush outcome
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava Blackboarde | izrađuje nove vrste Blackboarda i održava postojeće |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja Blackboard | čita zadani Blackboard iz konfiguracije i registrira ga kod procesora |
| Procesor | postaje vlasnik Blackboarda nakon registracije | poziva Blackboard da stvori dokumente, obrađuje ih kroz Blackboard i na kraju ga čisti |
| Pipeline | transformira dokumente tijekom sesije | čita, mijenja i zapisuje dokumente na platno |
| Repozitoriji | osiguravaju trajnu pohranu na unaprijed određena odredišta | pri predaji primaju dokumente od Blackboarda i potvrđuju ih; način pohrane opisuje dokumentacija repozitorija |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi Blackboard

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Pokretanje toka |
| **Preduvjet** | Konfiguracija navodi zadani procesor i njegov Blackboard |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije čita Blackboard. 2. Blackboard se stvara s konfiguriranom strategijom stvaranja (stanje IDLE). 3. Nakon registracije kod procesora (UC2) Workflow Factory Blackboardu registrira repozitorije (stanje READY). |
| **Alternativni tok** | Strategija stvaranja ili repozitorij nisu ispravni: greška, Blackboard se ne sastavlja ili repozitorij nije dodan. Repozitorij je već registriran: upozorenje, ponovna registracija se zanemaruje. Konfiguracija ne navodi repozitorije: Blackboard ostaje u stanju IDLE i ne prihvaća pisanje (BR-BB-04). |
| **Ishod** | Blackboard s repozitorijima, registriran kod procesora, spreman za obradu (stanje READY) |

#### UC2 — Registriraj Blackboard

| | |
|---|---|
| **Akter** | Workflow Factory (primarni), Procesor (podupirući) |
| **Okidač** | Workflow Factory je sastavio Blackboard prema konfiguraciji toka (UC1) |
| **Preduvjet** | Blackboard je stvoren sa strategijom stvaranja (UC1, korak 2) i procesor postoji |
| **Glavni tijek** | 1. Workflow Factory predaje Blackboard procesoru na registraciju. 2. Procesor provjerava da je predan Blackboard. 3. Procesor ga preuzima kao vlasnik. |
| **Alternativni tok** | Predano nije Blackboard: procesor odbija registraciju i javlja grešku. |
| **Ishod** | Procesor je vlasnik Blackboarda i može pokrenuti obradu |

Procesor ima jedan Blackboard: nova registracija zamjenjuje prethodni. Bez registriranog Blackboarda procesor ne počinje obradu ([`BR-PRC-02`](StRS-PROCESSOR.md#3-poslovna-pravila)).

#### UC3 — Stvori dokument

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Obrada treba novi dokument |
| **Preduvjet** | Blackboard je spreman |
| **Glavni tijek** | 1. Procesor poziva Blackboard da stvori dokument. 2. Blackboard izvodi registriranu strategiju stvaranja. 3. Dokument je na platnu i dostupan pipelineovima. |
| **Alternativni tok** | Radnja nije dopuštena u trenutnom stanju: greška, sadržaj Blackboarda ostaje nepromijenjen. |
| **Ishod** | Novi dokument na platnu |

#### UC4 — Čitaj i piši dokumente

| | |
|---|---|
| **Akter** | Pipeline |
| **Okidač** | Pipeline obrađuje dokument i zapisuje ga na Blackboard |
| **Preduvjet** | Barem jedan repozitorij je registriran |
| **Glavni tijek** | 1. Pipeline čita dokument. 2. Pipeline zapisuje izmijenjeni ili novi dokument, ili ga briše. 3. Blackboard bilježi da ima nepredanih dokumenata (stanje DIRTY). |
| **Alternativni tok** | Pisanje nije dopušteno u trenutnom stanju (npr. prije registracije repozitorija ili nakon čišćenja): greška, sadržaj Blackboarda ostaje nepromijenjen. |
| **Ishod** | Dokument na platnu ažuriran |

#### UC5 — Predaj dokumente repozitorijima

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Procesor nalaže predaju, tijekom ili nakon obrade, ovisno o konfiguraciji |
| **Preduvjet** | Barem jedan repozitorij je registriran |
| **Glavni tijek** | 1. Blackboard šalje sve dokumente s platna svakom repozitoriju. 2. Repozitoriji potvrđuju primljene dokumente. 3. Blackboard prazni platno (stanje READY). 4. Blackboard javlja procesoru da je predaja potvrđena. |
| **Alternativni tok** | A1. Platno je prazno ili neki repozitorij nije potvrdio sve dokumente: procesoru se javlja da predaja nije potvrđena. A2. Predaja ne uspije: Blackboard prelazi u stanje FAILED, iz kojeg se vraća učitavanjem zabilježenog stanja. |
| **Ishod** | Dokumenti predani repozitorijima; platno prazno |

#### UC6 — Očisti Blackboard

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Obrada je završena ili procesor prestaje postojati |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Blackboard uklanja sadržaj. 2. Prelazi u završno stanje CLEARED. |
| **Alternativni tok** | — (čišćenje nikad ne javlja grešku i smije se ponoviti) |
| **Ishod** | Blackboard prazan; ne prihvaća nove radnje |

#### UC7 — Razvij ili održavaj Blackboard

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Tok treba novu vrstu Blackboarda ili postojeća treba prilagodbe |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu Blackboarda prema zadanom dizajnu ili zahtjevu za izmjenu. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta Blackboarda koju Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Blackboard: Data persistence"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Processor
actor Pipeline
actor Repository
actor Programmer

rectangle "Blackboard" as Boundary <<system boundary>> {
    usecase "Sastavi Blackboard" as UC1
    usecase "Registriraj Blackboard" as UC2
    usecase "Stvori dokument" as UC3
    usecase "Čitaj i piši dokumente" as UC4
    usecase "Predaj dokumente\nrepozitorijima" as UC5
    usecase "Očisti Blackboard" as UC6
    usecase "Razvij ili održavaj\nBlackboard" as UC7
}

Factory -- UC1
Factory -- UC2
UC2 -- Processor
UC3 -- Processor
UC5 -- Processor
UC6 -- Processor
UC4 -- Pipeline
Repository -- UC5
Programmer -- UC7
@enduml
```

### 2.5 Stanja Blackboarda

**Blackboard u svakom trenutku ima jedno stanje i izvodi samo radnje koje to stanje dopušta. Nedopuštena radnja odbija se prije nego što promijeni sadržaj.** Prijelaze nadzire zajednički stroj stanja ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md)).

| stanje | značenje |
|---|---|
| IDLE | Blackboard je sastavljen, ali još nema repozitorija |
| READY | repozitoriji su registrirani; nema nepredanih dokumenata |
| DIRTY | na platnu postoje nepredani dokumenti |
| FAILED | rad je prekinut greškom; vraća se učitavanjem zabilježenog stanja |
| CLEARED | Blackboard je očišćen; završno stanje |

```plantuml
@startuml state-diagram
!include requirements/styles/wattleflow.puml

title "Blackboard: Data persistence"
caption "State Diagram"

[*] --> IDLE
IDLE --> READY : register / load
READY --> READY : register / flush
READY --> DIRTY : write
DIRTY --> DIRTY : write
DIRTY --> READY : flush
IDLE --> FAILED : fail
READY --> FAILED : fail
DIRTY --> FAILED : fail
FAILED --> READY : load
IDLE --> CLEARED : clean
READY --> CLEARED : clean
DIRTY --> CLEARED : clean
FAILED --> CLEARED : clean
CLEARED --> [*]
@enduml
```

### 2.6 Tok rada i praćenje grešaka

Dijagram prikazuje normalni tok jedne sesije (od sastavljanja do čišćenja) i alternativne tokove.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Blackboard: Data persistence"
caption "Flow Chart"

start
:Workflow Factory reads configuration;
:assemble Blackboard\nwith create strategy;
if (create strategy valid?) then (no)
  :BlackboardException; <<#red>>
  stop
endif
:Workflow Factory registers Blackboard\nwith Processor;
if (Processor accepts Blackboard?) then (no)
  :ProcessorException; <<#red>>
  stop
endif
:Workflow Factory registers repositories;
if (repository valid?) then (no)
  :BlackboardException; <<#red>>
  stop
endif
repeat
  :Processor calls Blackboard\nto create document;
  :pipelines read / write documents;
  if (action allowed in current state?) then (no)
    :BlackboardException; <<#red>>
    :content unchanged;
    stop
  endif
  if (flush during processing?) then (yes)
    :flush to repositories;
    :report outcome to Processor;
  endif
repeat while (more documents?) is (yes)
-> no;
:flush to repositories (if deferred); <<#green>>
:report outcome to Processor; <<#green>>
:clean Blackboard; <<#green>>
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `BlackboardException` | strategija stvaranja ili repozitorij nisu ispravni; radnja nije dopuštena u trenutnom stanju | radnja se odbija; sadržaj Blackboarda ostaje nepromijenjen |
| `ProcessorException` | pri registraciji procesoru nije predan Blackboard | registracija se odbija; procesor ne može pokrenuti obradu |
| upozorenje | repozitorij je već registriran | ponovna registracija se zanemaruje; događaj se bilježi |
| predaja nije potvrđena | platno je prazno ili neki repozitorij nije potvrdio sve dokumente | procesor dobiva taj ishod i odlučuje o daljnjem radu |
| greška pri gašenju | čišćenje pri gašenju ne uspije | greška se bilježi, ne prosljeđuje se dalje |

Svaka greška nosi uzrok. Blackboard bilježi svoje događaje: sastavljanje, registraciju repozitorija, greške i gašenje.

### 2.7 Pretpostavke i ograničenja

- Blackboard čuva dokumente samo dok traje sesija; trajna pohrana posao je repozitorija.
- Blackboard ne zna kako strategija stvaranja stvara dokumente ni kako repozitoriji pohranjuju podatke.
- Generički sloj ne propisuje koliko dokumenata Blackboard drži odjednom; to određuje specijalizacija.
- Izvana se sadržaj platna može samo čitati.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-BB-01** | Blackboard čuva dokumente samo za trajanja sesije; trajnu pohranu obavljaju repozitoriji. |
| **BR-BB-02** | Blackboard ima točno jednu strategiju stvaranja dokumenata i ne zna kako ona radi. Stvaranje pokreće procesor; Blackboard ga sam ne pokreće. |
| **BR-BB-03** | Blackboard ima jedan ili više repozitorija. Isti repozitorij ne registrira se dvaput. |
| **BR-BB-04** | Dokumenti se ne mogu pisati dok nije registriran barem jedan repozitorij. |
| **BR-BB-05** | Predaja šalje sve dokumente s platna svim repozitorijima i prazni platno. |
| **BR-BB-06** | Predaja je potvrđena samo ako je platno imalo barem jedan dokument i ako je svaki repozitorij potvrdio svaki dokument. Ishod se javlja procesoru. |
| **BR-BB-07** | Kada se predaja obavlja, tijekom ili nakon obrade, određuje konfiguracija. Ako konfiguracija ništa ne kaže, predaja se odgađa. |
| **BR-BB-08** | Blackboard izvodi samo radnje dopuštene u trenutnom stanju. Nedopuštena radnja odbija se prije nego što promijeni sadržaj. |
| **BR-BB-09** | Očišćen Blackboard ne prihvaća nove radnje. |
| **BR-BB-10** | Prijelaz u stanje neuspjeha i čišćenje nikad ne javljaju grešku i smiju se ponoviti. |
| **BR-BB-11** | Izvana se sadržaj platna može samo čitati; mijenja se isključivo radnjama Blackboarda. |
| **BR-BB-12** | Pravila pohrane koja objavi procesor Blackboard prosljeđuje repozitorijima pri predaji. |
| **BR-BB-13** | Kad Blackboard prestane postojati, čisti svoj sadržaj. |
| **BR-BB-14** | Blackboard kod procesora registrira Workflow Factory. Procesor ima jedan Blackboard; nova registracija zamjenjuje prethodni. |

## 4. Nefunkcionalni zahtjevi

Povezivanje s NFRQ zapisima je otvoreno (§6). Iz narativa i koda proizlaze ove kakvoće:

| kakvoća | što znači za Blackboard |
|---|---|
| dosljednost | nedopuštena radnja ne mijenja sadržaj (BR-BB-08) |
| sigurnost gašenja | čišćenje i prijelaz u neuspjeh smiju se ponavljati bez greške (BR-BB-10) |
| praćenje | Blackboard bilježi sastavljanje, registraciju, greške i gašenje |
| modularnost | Blackboard može registrirati jedan ili više različitih repozitorija |
| proširivost | programer može pisati nove ili proširivati postojeće Blackboarde za specifične potrebe tokova podataka |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-BB-*` su prijedlog.
2. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.
3. Generički sloj propisuje stanje neuspjeha i povratak iz njega, ali ne i što Blackboard dovodi u neuspjeh. Treba odlučiti opisuje li to ovaj zapis ili specijalizacije.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | generički procesor, vlasnik Blackboarda |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja Blackboard |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | stroj stanja |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | pipeline koji čita i piše dokumente |
| `concrete/blackboard.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.5 | 2026-10-11 | Redoslijed sastavljanja usklađen s kodom Workflow Factoryja: Blackboard se stvara sa strategijom stvaranja (IDLE), registrira kod procesora, pa dobiva repozitorije (READY); bez repozitorija ostaje u IDLE (UC1, UC2, tok). Poveznice na StRS-FACTORY i StRS-STATE-MACHINE. |
| v0.0.4 | 2026-10-11 | Oznaka bez broja (`StRS-BLACKBOARD`). Blackboard sastavlja i registrira Workflow Factory (UC1, UC2, BR-BB-14, dijagrami). Granice sustava u dijagramima dobile alias (sukob naziva s komponentom). Dodane poveznice na StRS-PIPELINE. |
| v0.0.3 | 2026-10-10 | Registracija Blackboarda izdvojena u zaseban korisnički slučaj UC2 (Workflow primarni, Procesor podupirući akter); dodano pravilo BR-BB-14; ostali slučajevi prenumerirani (UC2–UC6 → UC3–UC7). |
| v0.0.2 | 2026-10-10 | Workflow sastavlja Blackboard iz konfiguracije i registrira ga kod procesora (UC1, kontekst, tok); programer razvija i održava Blackboarde (novi UC6). |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz narativa autora i koda `concrete/blackboard.py`. |