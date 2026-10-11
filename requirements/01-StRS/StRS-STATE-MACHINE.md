<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-STATE-MACHINE — Stroj stanja: dopušteni prijelazi između stanja

| | |
|---|---|
| **Verzija** | v0.0.4 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/state_machine.py`: `StateMachine`, `GuardedStateMachine` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), [`StRS-CONNECTION`](StRS-CONNECTION.md), [`StRS-DRIVER`](StRS-DRIVER.md), [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — vlasnici stroja; svaki opisuje vlastita stanja i prijelaze |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi stroj stanja, tko ga koristi, kako osigurava da se komponenta mijenja samo propisanim redoslijedom i što se događa kad se zatraži nedopuštena promjena.

### 1.2 Opseg

**U opsegu:** zajednički stroj koji drži trenutno stanje i izvodi samo dopuštene prijelaze, provjera smije li se radnja izvesti, pokušaj prijelaza bez greške, jednokratna zaštita prije prvog prijelaza i greške koje stroj javlja.

**Izvan opsega:** koja stanja i prijelaze ima pojedina komponenta. To opisuje svaki vlasnik u svom zapisu ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), [`StRS-CONNECTION`](StRS-CONNECTION.md), [`StRS-DRIVER`](StRS-DRIVER.md), [`StRS-PROCESSOR`](StRS-PROCESSOR.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| stroj stanja | State machine | dio koji drži trenutno stanje komponente i dopušta samo propisane promjene |
| stanje | state | trenutna faza rada komponente, npr. „spojena" ili „neuspjela" |
| radnja | action | događaj koji može promijeniti stanje, npr. „spoji" ili „pokreni" |
| prijelaz | transition | promjena iz jednog stanja u drugo uzrokovana radnjom |
| tablica prijelaza | transition table | popis svih dopuštenih parova (stanje, radnja) i stanja u koje vode |
| početno stanje | initial state | stanje u kojem stroj počinje |
| vlasnik stroja | Owner | komponenta koja svoj stroj stanja sastavlja, čita i mijenja: Blackboard, konekcija, driver ili procesor |
| pokušaj prijelaza | try apply | pokušaj promjene stanja koji, umjesto greške, javlja je li prijelaz izveden |
| zaštita | guard | provjera koja se izvodi jednom, prije prvog prijelaza |
| paralelni tok rada | thread | dio programa koji se izvodi istodobno s drugima |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Komponente prolaze kroz faze rada i ne smiju iz svake faze u svaku: veza se ne smije otvoriti prije nego što je stvorena, a obrada koja je završila ne smije se nastaviti. Ako svaka komponenta sama pazi na redoslijed, pravila se razilaze, a nedopuštena promjena prođe neprimijećeno.

**Rješenje** Stroj stanja je zajednički dio koji drži trenutno stanje komponente i izvodi samo prijelaze iz zadane tablice. Zatražen prijelaz koji tablica ne dopušta ne izvodi se, a stanje ostaje kakvo jest. Isti stroj koriste Blackboard, konekcija, driver i procesor.

**Smjer** Stroj je općenit: ne zna što stanja znače. Znaju to vlasnici, koji mu predaju vlastitu tablicu prijelaza i početno stanje. Provjera i izvedba prijelaza jedan su nedjeljiv korak, pa dva paralelna toka rada ne mogu oba izvesti prijelaz iz istog stanja. Uz stroj postoji i omotač sa zaštitom koja se izvodi jednom, prije prvog prijelaza.

### 2.2 Kontekst

Vlasnik stroja pri sastavljanju predaje stroju tablicu prijelaza i početno stanje. Tijekom rada pita stroj smije li se radnja izvesti, traži prijelaz ili ga pokušava. Stroj mijenja stanje samo kad tablica to dopušta, a inače vraća grešku ili odgovor „ne". Vlasnik čita trenutno stanje i prema njemu odlučuje što smije dalje. Stroj nikoga ne obavještava o promjeni.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "State machine: Transition control"
caption "Context Diagram"

component "Owner\n(Blackboard, Connection,\nDriver, Processor)" as Owner
actor "Programmer" as Programmer

rectangle "State machine" as Boundary <<system boundary>> {
  component "State machine" as Machine
}

Programmer ..> Owner : defines states\nand transitions
Owner ..> Machine : builds with table,\nasks, applies, tries
Machine --> Owner : state / outcome / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava stanja i prijelaze | određuje koja stanja i prijelaze ima pojedina komponenta |
| Vlasnik stroja | koristi stroj | sastavlja ga s tablicom prijelaza, pita smije li se radnja, traži ili pokušava prijelaz i čita stanje |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi stroj stanja

| | |
|---|---|
| **Akter** | Vlasnik stroja |
| **Okidač** | Vlasnik se sastavlja ili vraća na zabilježeno stanje |
| **Preduvjet** | Vlasnik ima tablicu prijelaza i početno stanje |
| **Glavni tijek** | 1. Vlasnik predaje stroju tablicu prijelaza i početno stanje. 2. Stroj prepisuje tablicu i više je ne mijenja. 3. Stroj preuzima početno stanje. |
| **Alternativni tok** | A1. Početno stanje ne postoji u tablici: stroj se ne sastavlja, greška. |
| **Ishod** | Stroj u početnom stanju, s tablicom koju vlasnik više ne može izmijeniti |

#### UC2 — Provjeri smije li se radnja izvesti

| | |
|---|---|
| **Akter** | Vlasnik stroja |
| **Okidač** | Vlasnik treba znati je li radnja u trenutnom stanju dopuštena |
| **Preduvjet** | Stroj je sastavljen (UC1) |
| **Glavni tijek** | 1. Vlasnik pita stroj za radnju. 2. Stroj odgovara „da" ili „ne", bez promjene stanja. |
| **Alternativni tok** | — |
| **Ishod** | Vlasnik zna smije li izvesti radnju; stanje je nepromijenjeno |

Odgovor vrijedi za trenutak pitanja. Ako između pitanja i prijelaza drugi paralelni tok rada promijeni stanje, prijelaz se i dalje provjerava ponovno (UC3), pa nedopušten prijelaz ne može proći.

#### UC3 — Izvedi prijelaz

| | |
|---|---|
| **Akter** | Vlasnik stroja |
| **Okidač** | Vlasnik izvodi radnju koja mijenja njegovo stanje |
| **Preduvjet** | Stroj je sastavljen (UC1) |
| **Glavni tijek** | 1. Vlasnik traži prijelaz za radnju. 2. Stroj provjerava da tablica dopušta radnju u trenutnom stanju. 3. Stroj prelazi u novo stanje. |
| **Alternativni tok** | A1. Tablica radnju u trenutnom stanju ne dopušta: prijelaz se ne izvodi, stanje ostaje, vlasnik prima grešku. A2. Vlasnik umjesto prijelaza traži pokušaj prijelaza: stroj prijelaz izvodi samo ako je dopušten i vraća „da", inače vraća „ne" bez greške. |
| **Ishod** | Stroj u novom stanju, ili nepromijenjen uz grešku odnosno odgovor „ne" |

#### UC4 — Zaštiti prvi prijelaz

| | |
|---|---|
| **Akter** | Vlasnik stroja |
| **Okidač** | Vlasnik želi da se nešto provjeri prije prvog prijelaza |
| **Preduvjet** | Stroj je omotan zaštitom |
| **Glavni tijek** | 1. Vlasnik traži prvi prijelaz ili pokušaj prijelaza. 2. Zaštita se izvodi jednom i dobiva stroj. 3. Prijelaz se izvodi kao u UC3. |
| **Alternativni tok** | A1. Zaštita javi grešku: prijelaz se ne izvodi, vlasnik prima grešku, zaštita se ponavlja pri sljedećem zahtjevu. A2. Više paralelnih tokova rada stigne istodobno: zaštitu izvodi jedan, ostali čekaju. A3. Zaštita prođe, ali tablica prijelaz ne dopušta: vlasnik prima grešku ili odgovor „ne" (UC3), a zaštita se ipak smatra izvedenom i više se ne ponavlja. |
| **Ishod** | Zaštita je izvedena jednom; sljedeći prijelazi idu bez zaštite |

Čitanje stanja i provjera smije li se radnja izvesti ne pokreću zaštitu.

#### UC5 — Razvij ili održavaj stanja i prijelaze

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Komponenta treba novo stanje ili prijelaz ili postojeći treba izmjenu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer određuje stanja, radnje i dopuštene prijelaze komponente. 2. Vlasnik predaje stroju novu tablicu pri sastavljanju. |
| **Alternativni tok** | — |
| **Ishod** | Komponenta mijenja stanje prema novoj tablici |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "State machine: Transition control"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor Owner
actor Programmer

rectangle "State machine" as Boundary <<system boundary>> {
    usecase "Sastavi stroj stanja" as UC1
    usecase "Provjeri smije li se\nradnja izvesti" as UC2
    usecase "Izvedi prijelaz" as UC3
    usecase "Zaštiti prvi prijelaz" as UC4
    usecase "Razvij ili održavaj\nstanja i prijelaze" as UC5
}

Owner -- UC1
Owner -- UC2
Owner -- UC3
Owner -- UC4
Programmer -- UC5
UC4 ..> UC3 : <<extend>>
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Stroj nema vlastita stanja: stanja i prijelazi pripadaju vlasniku i opisani su u njegovom zapisu, pa odjeljak o stanjima izostaje. Dijagram prikazuje normalni tok (od sastavljanja do prijelaza) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "State machine: Transition control"
caption "Flow Chart"

start
:Owner builds machine with\ntransition table and initial state;
if (initial state in table?) then (no)
  :<b><color:red>ValueError</color></b>;
  :error returned to Owner;
  stop
endif
:Owner requests a transition\nor a try;
if (guarded and guard not yet passed?) then (yes)
  :run guard once;
  if (guard failed?) then (yes)
    :<b><color:red>error of the guard</color></b>;
    :error returned to Owner;
    stop
  endif
  :guard marked as passed;
endif
if (table allows action in current state?) then (no)
  if (request was a try?) then (yes)
    :answer "no" returned;
  else (no)
    :<b><color:red>ValueError</color></b>;
    :error returned to Owner;
  endif
  stop
endif
:state changed to the target state;
:answer "yes" / done returned to Owner;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ValueError` | početno stanje ne postoji u tablici; ili je zatražen prijelaz koji tablica u trenutnom stanju ne dopušta | stroj se ne sastavlja, odnosno prijelaz se ne izvodi i stanje ostaje; vlasnik prima grešku |
| greška zaštite | zaštita javi grešku pri prvom prijelazu | prijelaz se ne izvodi; zaštita se ponavlja pri sljedećem zahtjevu; vlasnik prima grešku zaštite |
| odgovor „ne" | pokušaj prijelaza koji tablica ne dopušta | stanje ostaje; nije greška |

Stroj ne bilježi događaje; neuspjeh bilježi vlasnik, a poruka greške navodi radnju i stanje.

### 2.6 Pretpostavke i ograničenja

- Stroj ne zna što stanja i radnje znače; zna samo tablicu koju mu je vlasnik predao.
- Stroj nikoga ne obavještava o promjeni stanja; vlasnik sam čita stanje.
- Stanje se izvana ne može postaviti, samo prijelazom. Za povratak na zabilježeno stanje vlasnik sastavlja novi stroj s tim početnim stanjem.
- Tablica se ne može mijenjati nakon sastavljanja.
- Stroj ne pamti prošla stanja ni broj prijelaza.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-SMC-01** | Stroj stanja u svakom trenutku ima točno jedno stanje. |
| **BR-SMC-02** | Stanje se mijenja samo prijelazom iz tablice: za par (stanje, radnja) tablica određuje novo stanje. |
| **BR-SMC-03** | Prijelaz koji tablica ne dopušta ne izvodi se: stanje ostaje, a vlasnik prima grešku, ili odgovor „ne" ako je prijelaz samo pokušan. |
| **BR-SMC-04** | Početno stanje mora postojati u tablici; inače se stroj ne sastavlja. |
| **BR-SMC-05** | Tablica se pri sastavljanju prepisuje; vlasnik je poslije ne može izmijeniti kroz stroj. |
| **BR-SMC-06** | Provjera i izvedba prijelaza čine jedan nedjeljiv korak: dva paralelna toka rada ne mogu oba izvesti prijelaz iz istog stanja. |
| **BR-SMC-07** | Provjera smije li se radnja izvesti ne mijenja stanje. |
| **BR-SMC-08** | Zaštita se izvodi jednom, prije prvog prijelaza ili pokušaja prijelaza; kad više paralelnih tokova rada stigne istodobno, izvodi je jedan, a ostali čekaju. |
| **BR-SMC-09** | Ako zaštita javi grešku, prijelaz se ne izvodi i zaštita se pri sljedećem zahtjevu izvodi ponovno. |
| **BR-SMC-10** | Zaštita ne utječe na čitanje stanja ni na provjeru smije li se radnja izvesti. |
| **BR-SMC-11** | Zaštita koja je prošla smatra se izvedenom i kad sam prijelaz potom nije dopušten. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; za njih još nema poveznice na zapise zahtjeva na kakvoću (§6).

| kakvoća | što znači za stroj stanja |
|---|---|
| ispravnost redoslijeda | komponenta ne može preskočiti ni ponoviti fazu koju tablica ne dopušta (BR-SMC-03) |
| sigurnost u paralelnom radu | dva paralelna toka rada ne mogu oba izvesti prijelaz iz istog stanja (BR-SMC-06) |
| ponovna uporaba | četiri komponente koriste isti stroj; svaka donosi samo svoju tablicu |
| razumljivost grešaka | poruka greške navodi radnju i stanje u kojem nije dopuštena |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-SMC-*` i naziv sloja u naslovima dijagrama („Transition control") su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Pokušaj prijelaza i omotač sa zaštitom (UC4) u pregledanom kodu ne koristi nijedna komponenta: vlasnici koriste samo provjeru i izvedbu prijelaza. Blackboard ima vlastiti pokušaj prijelaza, sastavljen od provjere i izvedbe, umjesto pokušaja iz stroja. Treba odrediti tko koristi pokušaj prijelaza i zaštitu, ili ih izostaviti iz okvira.
4. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | vlasnik stroja; stanja Blackboarda |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | vlasnik stroja; stanja konekcije |
| [`StRS-DRIVER`](StRS-DRIVER.md) | vlasnik stroja; stanja drivera |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | vlasnik stroja; stanja procesora |
| `concrete/state_machine.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.4 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.3 | 2026-10-11 | Zapisi vlasnika sada upućuju na ovaj zapis; stavka iz *Otvoreno* uklonjena. |
| v0.0.2 | 2026-10-11 | Usklađeno s kodom: zaštita se pokreće i pri pokušaju prijelaza i smatra se izvedenom i kad prijelaz potom nije dopušten (UC4 A3, novo pravilo BR-SMC-11, tok); pravilo o paralelnom radu precizirano (BR-SMC-06, narativ, §4); pojašnjeno da odgovor na provjeru vrijedi za trenutak pitanja (UC2). *Otvoreno*: oznaka zapisa više nije prijedlog; dodana stavka o poveznicama iz zapisa vlasnika; dopunjena stavka o pokušaju prijelaza. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/state_machine.py`. |