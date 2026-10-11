<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-MANAGER — Upravitelji: evidencija konekcija, drivera i procesora po imenu

| | |
|---|---|
| **Verzija** | v0.0.6 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/manager.py`: `ConnectionManager`, `DriverManager`, `ProcessorManager` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-CONNECTION`](StRS-CONNECTION.md) — upravljane konekcije · [`StRS-DRIVER`](StRS-DRIVER.md) — upravljani driveri · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — upravljani procesori · [`StRS-FACTORY`](StRS-FACTORY.md) — puni upravitelje |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služe upravitelji, tko ih koristi, kako evidentiraju i oslobađaju konekcije, drivere i procesore te što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** evidencija upravljanih objekata po imenu, njihovo dohvaćanje, izvođenje operacija nad njima po imenu, uklanjanje i oslobađanje, zamjena konekcije u hodu i greške koje upravitelji javljaju.

**Izvan opsega:** što upravljani objekti sami rade i u kojim su stanjima (opisano u [`StRS-CONNECTION`](StRS-CONNECTION.md), [`StRS-DRIVER`](StRS-DRIVER.md) i [`StRS-PROCESSOR`](StRS-PROCESSOR.md)) te kako ih Workflow Factory sastavlja iz konfiguracije. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| upravitelj | Manager | dio sustava koji vodi evidenciju objekata jedne vrste po imenu; postoje tri: za konekcije, drivere i procesore |
| upravljani objekt | managed object | konekcija, driver ili procesor koji je upisan u evidenciju upravitelja |
| ime | name | jedinstvena oznaka kojom se upravljani objekt upisuje i traži u evidenciji |
| registracija | register | upis objekta u evidenciju pod njegovim imenom |
| uklanjanje | unregister | brisanje objekta iz evidencije, uz oslobađanje onoga što je objekt držao |
| oslobađanje | release | zatvaranje konekcije ili odspajanje drivera, kako ne ostanu otvoreni resursi |
| operacija | operation | zahtjev upravitelja objektu po imenu: povezati, odspojiti, pokrenuti |
| zamjena u hodu | hot swap | zamjena registrirane konekcije drugom bez gubitka njezinih promatrača |
| promatrač | Observer | dio sustava koji se pretplati na konekciju i prima obavijesti o njezinim događajima |
| konekcija | Connection | jedina granica prema vanjskom sustavu ([`StRS-CONNECTION`](StRS-CONNECTION.md)) |
| driver | Driver | dio sustava koji kroz konekciju radi s vanjskim sustavom ([`StRS-DRIVER`](StRS-DRIVER.md)) |
| procesor | Processor | motor koji provodi zapise kroz pipelineove ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja konekcije, drivere i procesore, upisuje ih u upravitelje i dohvaća drivere za komponente koje ih navode ([`StRS-FACTORY`](StRS-FACTORY.md)) |
| Workflow | Workflow | dio sustava koji izvršava tok: od upravitelja procesora dohvaća procesore i pokreće ih |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Konekcije, driveri i procesori moraju se naći po imenu, pokrenuti i na kraju uredno ugasiti, a rade ih različiti dijelovi toka. Ako svaki dio sam pamti što je napravio, objekti ostaju neugašeni, isto ime znači različite stvari, a zamjena konekcije usred rada gubi sve što je na nju bilo vezano.

**Rješenje** Upravitelj je evidencija objekata jedne vrste po imenu. Tok objekt ne drži sam nego ga traži od upravitelja po imenu i po imenu mu šalje zahtjev. Kad se objekt ukloni ili upravitelj prestane postojati, upravitelj objekt oslobađa. Upravitelj konekcija dodatno zna zamijeniti konekciju drugom, uz prijenos promatrača.

**Smjer** Tri upravitelja dijele isti oblik, a razlikuju se po onome što drže i po radnjama koje dopuštaju. Upravitelj ne zna što zahtjev znači objektu: samo ga prosljeđuje. Upravitelji nisu vlasnici toka; objekte upisuje Workflow Factory prema konfiguraciji. Iako su upravitelji promatrači, zasad ništa ne promatraju.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja konekcije, drivere i procesore te ih upisuje u odgovarajućeg upravitelja. Driver od upravitelja konekcija dohvaća svoju konekciju po imenu. Workflow Factory od upravitelja drivera po imenu dohvaća driver za komponentu koja ga navodi, a Workflow od upravitelja procesora dohvaća procesore i pokreće ih. Kad se objekt ukloni ili upravitelj prestane postojati, upravitelj objekt oslobađa. Greške se vraćaju onome tko je radnju zatražio.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Managers: Component registry"
caption "Context Diagram"

actor "Workflow Factory" as Factory
actor "Workflow" as Workflow
component "Driver" as Driver
component "Connection" as Connection
component "Processor" as Processor

rectangle "Managers" as Boundary <<system boundary>> {
  component "ConnectionManager" as CM
  component "DriverManager" as DM
  component "ProcessorManager" as PM
}

Factory ..> CM : registers
Factory ..> DM : registers
Factory ..> PM : registers
Workflow ..> PM : asks for processors,\nstarts them
Driver ..> CM : asks for connection
CM --> Driver : connection
Factory ..> DM : asks for driver
DM --> Factory : driver
DM ..> Driver : loads, releases
CM ..> Connection : connects, disconnects,\nreleases, swaps
PM ..> Processor : starts
PM --> Workflow : processor / outcome
@enduml
```

Driver se na dijagramu pojavljuje jednom, iako ima dvije uloge: korisnik je upravitelja konekcija i upravljani objekt upravitelja drivera.

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Workflow Factory | sastavlja objekte iz konfiguracije | upisuje konekcije, drivere i procesore u upravitelje i po imenu dohvaća drivere za komponente koje ih navode |
| Workflow | izvršava tok | od upravitelja procesora dohvaća procesore i pokreće ih |
| Driver | koristi konekciju; ujedno je upravljani objekt | dohvaća svoju konekciju od upravitelja konekcija po imenu; odgovara na zahtjeve spajanja i odspajanja |
| Pokretač zamjene | zahtijeva zamjenu konekcije | nalaže zamjenu registrirane konekcije drugom; tko je to, nije određeno (§6) |
| Konekcija | upravljani objekt | odgovara na zahtjeve povezivanja i odspajanja te na zamjenu |
| Procesor | upravljani objekt | odgovara na zahtjev pokretanja |
| Promatrač | prati konekciju | pri zamjeni konekcije prelazi na novu konekciju i prima obavijest o zamjeni |

### 2.4 Korisnički slučajevi

#### UC1 — Registriraj objekt

| | |
|---|---|
| **Akter** | Workflow Factory (primarni), upravljani objekt (podupirući) |
| **Okidač** | Workflow Factory je sastavio konekciju, driver ili procesor prema konfiguraciji |
| **Preduvjet** | Objekt ima ime |
| **Glavni tijek** | 1. Workflow Factory predaje objekt upravitelju njegove vrste. 2. Upravitelj upisuje objekt pod njegovim imenom. |
| **Alternativni tok** | A1. Objekt s tim imenom već je upisan: novi se ne upisuje, stari ostaje, zapisuje se upozorenje. A2. Konekcija nema ime: registracija se odbija, greška upravitelja. |
| **Ishod** | Objekt je u evidenciji i dostupan po imenu |

#### UC2 — Dohvati objekt po imenu

| | |
|---|---|
| **Akter** | Driver (konekcija), Workflow Factory (driver, pri sastavljanju), Workflow (procesor) |
| **Okidač** | Dio toka treba konekciju, driver ili procesor |
| **Preduvjet** | Objekt je registriran (UC1) |
| **Glavni tijek** | 1. Pozivatelj traži objekt po imenu. 2. Upravitelj ga vraća. |
| **Alternativni tok** | A1. Ime nije upisano: greška upravitelja. |
| **Ishod** | Pozivatelj ima traženi objekt |

#### UC3 — Izvedi operaciju nad objektom

| | |
|---|---|
| **Akter** | Pozivatelj koji nije određen (konekcije i driveri, §6), Workflow (pokretanje procesora) |
| **Okidač** | Tok treba povezati ili odspojiti konekciju, spojiti ili odspojiti driver, ili pokrenuti procesor |
| **Preduvjet** | Objekt je registriran (UC1) |
| **Glavni tijek** | 1. Pozivatelj šalje upravitelju ime i radnju. 2. Upravitelj prosljeđuje radnju objektu. 3. Objekt je izvodi i javlja ishod upravitelju. 4. Upravitelj javlja ishod pozivatelju; pri spajanju drivera i pokretanju procesora vraća i sam objekt. |
| **Alternativni tok** | A1. Ime nije upisano: greška upravitelja. A2. Procesoru je zatražena radnja različita od pokretanja: zahtjev se odbija odgovorom „ne" i upozorenjem. A3. Objekt u izvođenju radnje javi grešku (npr. povezivanje ili odspajanje ne uspije): greška se vraća pozivatelju. |
| **Ishod** | Radnja izvedena ili odbijena; pozivatelj zna ishod |

#### UC4 — Ukloni objekt

| | |
|---|---|
| **Akter** | Pozivatelj koji nije određen (primarni, §6), upravljani objekt (podupirući) |
| **Okidač** | Objekt više nije potreban toku |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Pozivatelj traži uklanjanje po imenu ili objektu. 2. Upravitelj briše objekt iz evidencije. 3. Upravitelj oslobađa objekt: konekciju zatvara, bez obzira na to je li sesija otvorena, a driver odspaja. Procesoru se samo briše unos. |
| **Alternativni tok** | A1. Ime nije upisano: za konekcije i drivere zapisuje se upozorenje, za procesore se zahtjev tiho preskače; ništa se ne briše. A2. Oslobađanje ne uspije: zapisuje se greška, a objekt je iz evidencije ipak uklonjen. |
| **Ishod** | Objekt nije u evidenciji; konekcija i driver nemaju otvorenih resursa. Procesor svoj Blackboard čisti sam, kad prestane postojati. |

#### UC5 — Zamijeni konekciju u hodu

| | |
|---|---|
| **Akter** | Pokretač zamjene (primarni), Konekcija i Promatrač (podupirući) |
| **Okidač** | Postoji nova konekcija koja treba zamijeniti registriranu pod istim imenom |
| **Preduvjet** | Stara konekcija je registrirana pod tim imenom |
| **Glavni tijek** | 1. Pokretač predaje upravitelju ime i novu konekciju. 2. Upravitelj povezuje novu konekciju. 3. Promatrači stare prelaze na novu. 4. Nova zamjenjuje staru u evidenciji. 5. Nova konekcija šalje obavijest o zamjeni. 6. Upravitelj odspaja staru konekciju. |
| **Alternativni tok** | A1. Ime nije upisano: greška upravitelja. A2. Nova konekcija se ne uspije povezati: zamjena se prekida, stara ostaje registrirana i aktivna, greška upravitelja. A3. Odspajanje stare ne uspije: zamjena ostaje, zapisuje se upozorenje. |
| **Ishod** | Pod istim imenom je nova konekcija koja ima promatrače stare; stara je odspojena |

#### UC6 — Oslobodi objekte pri gašenju

| | |
|---|---|
| **Akter** | — |
| **Okidač** | Upravitelj prestaje postojati |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Upravitelj redom oslobađa svaki upisani objekt: konekcije zatvara, drivere odspaja, a procesore uklanja iz evidencije. 2. Evidencija se prazni. |
| **Alternativni tok** | A1. Oslobađanje pojedinog objekta ne uspije: kvar se bilježi, ostali objekti svejedno se oslobađaju. |
| **Ishod** | Nema upisanih objekata; konekcije i driveri nemaju otvorenih resursa |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Managers: Component registry"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Workflow
actor "Caller\n(not determined)" as Caller
actor Driver
actor "Swap initiator" as Swap
actor Connection
actor Observer

rectangle "Managers" as Boundary <<system boundary>> {
    usecase "Registriraj objekt" as UC1
    usecase "Dohvati objekt\npo imenu" as UC2
    usecase "Izvedi operaciju\nnad objektom" as UC3
    usecase "Ukloni objekt" as UC4
    usecase "Zamijeni konekciju\nu hodu" as UC5
    usecase "Oslobodi objekte\npri gašenju" as UC6
}

Factory -- UC1
Workflow -- UC2
Factory -- UC2
Driver -- UC2
Caller -- UC3
Workflow -- UC3
Caller -- UC4
Swap -- UC5
UC5 -- Connection
UC5 -- Observer
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Upravitelji nemaju vlastita stanja; stanja imaju upravljani objekti. Dijagram prikazuje normalni tok (od registracije do uklanjanja) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Managers: Component registry"
caption "Flow Chart"

start
:Workflow Factory registers object;
if (connection without name?) then (yes)
  :<b><color:red>ManagerException</color></b>;
  :error returned to Workflow Factory;
  stop
endif
if (name already registered?) then (yes)
  :warning logged, old object kept;
else (no)
  :object stored under its name;
endif
:caller requests object\nor operation by name;
if (name registered?) then (no)
  :<b><color:red>ManagerException</color></b>;
  :error returned to caller;
  stop
endif
if (processor and action is not start?) then (yes)
  :warning logged;
  :answer "no" returned to caller;
  stop
endif
:operation forwarded to object;
if (connection swap requested?) then (yes)
  if (new connection connected?) then (no)
    :<b><color:red>ManagerException</color></b>;
    :old connection stays active;
    :error returned to initiator;
    stop
  endif
  :observers moved to new connection,\nswap announced;
  :old connection disconnected\n(warning if this fails);
endif
:object removed\n(caller not determined);
:object released\n(error logged if this fails);
:entry deleted;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ManagerException` | ime nije upisano (dohvat, operacija, zamjena); konekcija se registrira bez imena; nova konekcija pri zamjeni se ne poveže | radnja se odbija; pri zamjeni stara konekcija ostaje registrirana i aktivna; onaj tko ju je zatražio prima grešku |
| upozorenje | ime je već upisano; uklanjanje konekcije ili drivera koji nije upisan; procesoru je zatražena radnja različita od pokretanja (uz odgovor „ne"); odspajanje stare konekcije pri zamjeni ne uspije | rad se nastavlja; događaj se bilježi |
| bez učinka | uklanjanje procesora koji nije upisan | ništa se ne događa i ništa se ne bilježi |
| zabilježena greška | oslobađanje pri uklanjanju ili gašenju ne uspije | objekt je ipak uklonjen iz evidencije; ostali objekti se oslobađaju; greška se bilježi, ali ne vraća pozivatelju |

Greške koje javi sam upravljani objekt (konekcija, driver, procesor) prosljeđuju se pozivatelju kakve jesu. Upravitelji bilježe svoje događaje: registraciju, uklanjanje, operacije, zamjenu i greške.

### 2.6 Pretpostavke i ograničenja

- Upravitelj ne zna što upravljani objekt radi: samo ga evidentira i prosljeđuje mu zahtjeve.
- Ime je jedinstveno unutar jednog upravitelja; isto ime u različitim upraviteljima nije povezano.
- Upravitelj procesora pri uklanjanju samo briše unos: procesor se ne zaustavlja. Procesor svoj Blackboard čisti sam, kad prestane postojati ([`BR-PRC-12`](StRS-PROCESSOR.md#3-poslovna-pravila)); to se događa tek kad ga više nitko ne drži.
- Upravitelj ne bira kad zamijeniti konekciju: zamjenu mora zatražiti netko izvana.
- Upravitelj ništa ne promatra: obavijesti koje primi zanemaruje.
- Evidencija je u memoriji: nestaje s upraviteljem.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-MGR-01** | Upravitelj drži objekte jedne vrste (konekcije, drivere ili procesore) po imenu; ime je jedinstveno unutar upravitelja. |
| **BR-MGR-02** | Ponovna registracija već upisanog imena ne zamjenjuje stari objekt: novi se zanemaruje uz upozorenje. |
| **BR-MGR-03** | Konekcija se ne registrira bez imena. |
| **BR-MGR-04** | Objekt se dohvaća i na njemu se izvodi operacija samo po upisanom imenu; nepoznato ime je greška. |
| **BR-MGR-05** | Upravitelj prosljeđuje operaciju objektu i ne zna što ona za njega znači. |
| **BR-MGR-06** | Procesoru se preko upravitelja može zatražiti samo pokretanje; svaka druga radnja odbija se upozorenjem i odgovorom „ne". |
| **BR-MGR-07** | Pri uklanjanju upravitelj konekciju zatvara, bez obzira na to je li sesija otvorena ([`BR-CON-17`](StRS-CONNECTION.md#3-poslovna-pravila)), a driver odspaja; unos nestaje i kad oslobađanje ne uspije, a kvar se bilježi. |
| **BR-MGR-08** | Uklanjanje nepoznatog imena nije greška: za konekcije i drivere bilježi se upozorenje, a za procesore se zahtjev tiho preskače. |
| **BR-MGR-09** | Kad upravitelj prestane postojati, oslobađa sve upravljane objekte (konekcije zatvara kao u BR-MGR-07); kvar jednog ne sprječava oslobađanje ostalih. |
| **BR-MGR-10** | Pri zamjeni konekcije nova se najprije povezuje; ako ne uspije, stara ostaje registrirana i aktivna. |
| **BR-MGR-11** | Pri zamjeni nova konekcija preuzima promatrače stare i šalje obavijest o zamjeni; stara se odspaja tek posljednja. |
| **BR-MGR-12** | Kvar odspajanja stare konekcije ne poništava zamjenu: bilježi se upozorenje. |
| **BR-MGR-13** | Povezivanje i odspajanje po imenu ne skrivaju grešku: greška objekta ili nepoznato ime stižu pozivatelju. Samo oslobađanje pri uklanjanju i gašenju (BR-MGR-07, BR-MGR-09) bilježi grešku umjesto da je javi pozivatelju. |
| **BR-MGR-14** | Upravitelj prima obavijesti, ali na njih ne reagira. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; za njih još nema poveznice na zapise zahtjeva na kakvoću (§6).

| kakvoća | što znači za upravitelje |
|---|---|
| urednost | resursi se oslobađaju pri uklanjanju i gašenju, pa nema zaboravljenih konekcija i drivera (BR-MGR-07, BR-MGR-09) |
| otpornost | kvar oslobađanja jednog objekta ne zaustavlja ostale (BR-MGR-09); neuspjela zamjena ne ruši staru konekciju (BR-MGR-10) |
| praćenje | upravitelji bilježe registraciju, uklanjanje, operacije, zamjenu i greške |
| odvojenost | tok ne drži objekte sam: traži ih po imenu, a o njihovu oslobađanju brine upravitelj |
| ujednačenost | sva tri upravitelja registriraju, dohvaćaju, izvode operacije i uklanjaju objekte po imenu na isti način; razlike su navedene u §6 |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-MGR-*` i naziv sloja u naslovima dijagrama („Component registry") su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Zamjena konekcije u hodu: provodi je upravitelj, ali je u pregledanom kodu nitko ne poziva. Nije određeno tko je i zašto pokreće (akter „Pokretač zamjene", UC5).
4. Sloj: nije odlučeno kojem sloju pripadaju upravitelji. Isto pitanje otvoreno je za konekciju ([`StRS-CONNECTION`](StRS-CONNECTION.md)).
5. Procesor se pri uklanjanju ne zaustavlja, dok se konekcija i driver oslobađaju. Treba potvrditi da je ta razlika željena.
6. U kodu postoji orkestrator, dio sustava koji drži upravitelja konekcija, ali ga u pregledanom kodu ne koristi. Treba odlučiti pripada li to ovom zapisu.
7. **Razlike među upraviteljima.** Narativ kaže da tri upravitelja dijele isti oblik, a razlikuju se u ovome: upravitelj konekcija nema popis svih objekata koji druga dva imaju; upravitelj drivera nema prečace za spajanje i odspajanje po imenu; upravitelj procesora ne bilježi upozorenje pri uklanjanju nepoznatog imena (BR-MGR-08). Treba odlučiti jesu li razlike namjerne.
8. **Tko traži operacije i uklanjanje.** Workflow Factory upravitelje samo puni i iz njih dohvaća drivere; spajanje i odspajanje konekcija i drivera te uklanjanje objekata ne poziva. Nije određeno tko to radi (UC3, UC4), pa ni tko spaja drivere prije rada toka.
9. Kakvoće treba povezati s postojećim NFRQ zapisima ili otvoriti nove.
10. **Zatvaranje konekcija.** BR-MGR-07 i BR-MGR-09 traže da upravitelj zatvori svaku konekciju, a kod trenutačno zatvara samo konekcije s otvorenom sesijom. Kod treba uskladiti s pravilom (SyRS).

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | konekcije koje upravlja upravitelj konekcija |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driveri koje upravlja upravitelj drivera |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesori koje upravlja upravitelj procesora |
| `concrete/manager.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.6 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.5 | 2026-10-11 | Prema kodu Workflow Factoryja: driver od upravitelja drivera dohvaća Workflow Factory, ne strategije čitanja i pisanja (pojmovi, kontekst, dionici, UC2). Workflow Factory ne izvodi operacije ni uklanjanje, pa je akter UC3 (konekcije i driveri) i UC4 neodređen, a UC6 nastaje gašenjem upravitelja bez aktera. *Otvoreno*: tko traži operacije i uklanjanje. |
| v0.0.4 | 2026-10-11 | Upravitelj pri uklanjanju i gašenju zatvara svaku konekciju, bez obzira na to je li sesija otvorena (BR-MGR-07, BR-MGR-09, UC4, UC6; usklađeno s BR-CON-17). Razlika prema trenutačnom kodu zabilježena u *Otvoreno*. |
| v0.0.3 | 2026-10-11 | Nazivi aktera usklađeni s ostalim StRS-ovima: objekte sastavlja, upisuje, uklanja i njima upravlja Workflow Factory, a procesore dohvaća i pokreće Workflow (pojmovi, dionici, UC1–UC4, UC6, dijagrami). Uklonjena stavka o usklađivanju sa StRS-DRIVER, koji je ispravljen. |
| v0.0.2 | 2026-10-10 | Usklađeno s kodom i povezanim StRS-ovima: uklanjanje nepoznatog procesora tiho se preskače (BR-MGR-08, UC4); procesor svoj Blackboard čisti sam; driver nije akter operacija nad konekcijom (UC3); strategije čitanja i pisanja dodane kao korisnici upravitelja drivera; driver se *odspaja*, ne *prazni*; kontekstni dijagram s jednim driverom; otvorena pitanja dopunjena (razlike među upraviteljima, usklađivanje sa StRS-DRIVER, stara oznaka FRQ-MGR). |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz koda `concrete/manager.py`. |