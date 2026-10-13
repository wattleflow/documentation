<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-DRIVER — Driver: jedinstven pristup vanjskim resursima

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/driver.py`: `GenericDriver`, `DriverState`, `DriverAction`, `DriverMetadata` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-PROCESSOR`](StRS-PROCESSOR.md), [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — korisnici podataka do kojih driver omogućuje pristup · [`StRS-MANAGER`](StRS-MANAGER.md) — upravitelji drivera i konekcija · [`StRS-FACTORY`](StRS-FACTORY.md) — Workflow Factory predaje driver komponentama · [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) — stroj stanja · [`StRS-REPOSITORY`](StRS-REPOSITORY.md) — repozitorij čije strategije koriste driver · [`StRS-CONNECTION`](StRS-CONNECTION.md) — konekcija preko koje driver radi · [`StRS-STRATEGY`](StRS-STRATEGY.md) — strategije koje koriste driver · [`StRS-WORKFLOW`](StRS-WORKFLOW.md) — workflow koji traži izvješće o driverima |
| **Dijagrami** | inline (§2.2, §2.4, §2.5, §2.6) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi driver, tko ga koristi, kako se spaja na vanjski resurs i odspaja s njega, kako se konekcija može zamijeniti u hodu, u kojim se stanjima driver može nalaziti i što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** sastavljanje drivera, spajanje i odspajanje, čitanje i pisanje podataka, ponašanje drivera pri zamjeni konekcije, stanja drivera i greške koje javlja.

**Izvan opsega:** rad Driver Managera i Connection Managera, način na koji pojedini driver govori „jezikom" svog resursa (baza podataka, kamera, senzor, servis) i rad strategija čitanja i pisanja repozitorija te odgođeno stvaranje drivera (`LazyDriverProxy`), koje služi specijalizacijama u paketu *blackwattle*. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| driver | Driver | univerzalni prevoditelj: jedno zajedničko sučelje za čitanje i pisanje podataka u vanjski resurs |
| resurs | External resource | izvor ili odredište podataka: baza podataka, kamera, senzor, servis — u lokalnoj mreži, na internetu ili u cloudu |
| konekcija | Connection | spoj prema resursu preko kojeg driver radi ([`StRS-CONNECTION`](StRS-CONNECTION.md)) |
| Workflow Factory | Workflow Factory | dio sustava koji prema konfiguraciji stvara i podešava driver i registrira njegove konekcije |
| Driver Manager | Driver Manager | upravitelj drivera: spaja ih i odspaja te ih po imenu daje Workflow Factoryju ([`StRS-MANAGER`](StRS-MANAGER.md)) |
| Connection Manager | Connection Manager | upravitelj konekcija; tehnički pomoćnik koji na zahtjev izvodi zamjenu konekcije u hodu ([`StRS-MANAGER`](StRS-MANAGER.md)) |
| pokretač zamjene | Swap initiator | dio sustava koji zahtijeva zamjenu konekcije; još nije određen (§6) |
| strategija čitanja i pisanja | Read / write strategy | dio repozitorija koji preko drivera čita ili zapisuje podatke |
| metapodaci | metadata | naziv, verzija, protokol i sposobnosti drivera |
| sposobnost | capability | radnja koju driver podržava, iz zatvorenog popisa sposobnosti |
| spajanje | connect / load | priprema drivera i resursa za rad |
| odspajanje | disconnect / unload | zatvaranje resursa i oslobađanje konekcije |
| zamjena konekcije | hot swap | prebacivanje drivera na novu konekciju bez prekida rada sustava |
| stanje | state | trenutna faza rada drivera (§2.5) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Sustav čita i piše podatke iz mnogo različitih izvora: baza podataka, kamera, senzora i servisa, strukturiranih i nestrukturiranih. Kad bi svaki dio sustava morao naučiti „jezik" svakog izvora posebno, isti bi se kod ponavljao, a dodavanje novog izvora zahtijevalo bi izmjene na mnogo mjesta.

**Rješenje** Driver je poput univerzalnog prevoditelja. Umjesto da svaki dio sustava uči jezik svake baze podataka, kamere ili senzora posebno, driver nudi jedno zajedničko sučelje kroz koje se sve to čita i piše, bilo da su podaci strukturirani ili ne. Driverima upravlja Driver Manager, pa više dijelova sustava može dijeliti iste resurse.

**Smjer** Driver ima vlastiti životni ciklus: zna se kad se stvara, kad radi i kad se gasi. Sustav je modularan, pa se lako dodaju novi driveri za nove podsustave, bilo u lokalnoj mreži, na internetu ili u cloudu. Workflow Factory driver stvara i podešava prema konfiguraciji, a korisnici mu pristupaju preko Driver Managera, po imenu. Konekcija iza drivera može se zamijeniti u hodu, i to sigurno: stara konekcija ostaje dok nova ne proradi.

### 2.2 Kontekst

Workflow Factory prema konfiguraciji stvara driver i registrira ga kod Driver Managera. Pri sastavljanju toka Workflow Factory od Driver Managera po imenu dohvaća driver i predaje ga komponenti koja ga je navela: repozitoriju, procesoru ili izlazu monitora. Repozitorij driver dalje predaje svojim strategijama čitanja i pisanja, koje preko njega čitaju i pišu podatke u vanjski resurs. Driver Manager driver spaja i odspaja. Na zahtjev pokretača zamjene Connection Manager zamjenjuje konekciju iza drivera i o tome ga obavještava.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Driver: Data access"
caption "Context Diagram"

actor "Workflow Factory" as Factory
component "Driver Manager" as DM
component "Connection Manager" as CM
actor "Swap initiator" as Swap
component "Repository\n(read / write strategies)" as RW
component "Processors,\nmonitor exporters" as Others
database "External resource\n(database, camera,\nsensor, service)" as Resource

rectangle "Driver" as Boundary <<system boundary>> {
  component Driver
}

Factory ..> Driver : assembles,\nregisters connections
DM ..> Driver : connect / disconnect
Factory ..> DM : registers driver,\ngets it by name
Factory ..> RW : hands over driver
Factory ..> Others : hands over driver
RW ..> Driver : read / write
Others ..> Driver : read / write
Swap ..> CM : requests swap
CM ..> Driver : swap event
Driver --> Resource : reads / writes
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava drivere | izrađuje drivere za nove podsustave i održava postojeće |
| Workflow Factory | prema konfiguraciji sastavlja driver | stvara i podešava driver te registrira njegove konekcije |
| Driver Manager | upravlja driverima | spaja i odspaja drivere; po imenu ih daje Workflow Factoryju |
| Strategije čitanja i pisanja | glavni korisnici drivera | čitaju i zapisuju podatke u resurs preko drivera koji im predaje repozitorij |
| Repozitoriji, procesori i izlazi monitora | dobivaju driver | primaju driver od Workflow Factoryja pri sastavljanju toka |
| Pokretač zamjene | zahtijeva zamjenu konekcije | nalaže zamjenu konekcije u hodu; tko je to, nije određeno (§6) |
| Connection Manager | tehnički pomoćnik drivera | na zahtjev izvodi zamjenu konekcije u hodu i obavještava driver o zamjeni |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi driver

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Pokretanje toka |
| **Preduvjet** | Konfiguracija navodi driver, njegove metapodatke i konekcije |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije čita driver. 2. Stvara ga s metapodacima (naziv, verzija, protokol, sposobnosti). 3. Registrira njegove konekcije. |
| **Alternativni tok** | Metapodaci nisu ispravni (nepoznata, ponovljena ili neispravno zadana sposobnost): greška, driver se ne stvara. |
| **Ishod** | Driver stvoren, u stanju PENDING, registriran kod Driver Managera; Workflow Factory ga predaje komponentama koje ga navode ([`BR-FCT-08`](StRS-FACTORY.md#3-poslovna-pravila)) |

#### UC2 — Spoji driver

| | |
|---|---|
| **Akter** | Driver Manager |
| **Okidač** | Driver Manager zatraži spajanje |
| **Preduvjet** | Driver je u stanju PENDING, UNLOADED, PAUSED ili DEGRADED |
| **Glavni tijek** | 1. Driver prelazi u stanje LOADING. 2. Priprema resurs za rad. 3. Prelazi u stanje LIVE. |
| **Alternativni tok** | A1. Driver je već spojen ili se upravo spaja: zahtjev nema učinka. A2. Stanje ne dopušta spajanje: spajanje se odbija greškom. A3. Priprema resursa ne uspije: driver prelazi u stanje DEGRADED, a izvorna greška prosljeđuje se onome tko je zatražio spajanje. |
| **Ishod** | Driver spojen, u stanju LIVE |

#### UC3 — Čitaj i piši podatke

| | |
|---|---|
| **Akter** | Strategija čitanja i pisanja ili druga komponenta koja je dobila driver |
| **Okidač** | Korisnik treba pročitati podatke iz resursa ili ih zapisati u njega |
| **Preduvjet** | Komponenta je pri sastavljanju dobila driver (UC1), a driver je spojen (UC2) |
| **Glavni tijek** | 1. Korisnik se obraća driveru koji je dobio pri sastavljanju (strategija ga dobiva od svog repozitorija). 2. Driver čita ili zapisuje podatke na zadanoj adresi u resursu. 3. Korisnik dobiva rezultat. |
| **Alternativni tok** | Čitanje ili pisanje ne uspije: korisnik prima grešku. |
| **Ishod** | Podaci pročitani iz resursa ili zapisani u njega |

#### UC4 — Odspoji driver

| | |
|---|---|
| **Akter** | Driver Manager |
| **Okidač** | Zahtjev za odspajanje, ili driver prestaje postojati |
| **Preduvjet** | Driver je u stanju LIVE, PAUSED ili DEGRADED |
| **Glavni tijek** | 1. Driver prelazi u stanje UNLOADING. 2. Zatvara resurs i oslobađa konekciju. 3. Prelazi u stanje UNLOADED. |
| **Alternativni tok** | A1. Driver je već odspojen ili ga stanje ne dopušta odspojiti: zahtjev nema učinka. A2. Zatvaranje ne uspije: driver prelazi u stanje DEGRADED, a greška se prosljeđuje. Kad driver prestaje postojati, greška se ne prosljeđuje. |
| **Ishod** | Driver odspojen, u stanju UNLOADED; može se ponovno spojiti ili vratiti u početno stanje |

#### UC5 — Zamijeni konekciju

| | |
|---|---|
| **Akter** | Pokretač zamjene (primarni), Connection Manager (podupirući) |
| **Okidač** | Pokretač zamjene zahtijeva zamjenu konekcije u hodu |
| **Preduvjet** | Postoji nova konekcija prema resursu |
| **Glavni tijek** | 1. Pokretač zamjene predaje Connection Manageru novu konekciju. 2. Connection Manager spaja novu konekciju. 3. Prebacuje promatrače stare konekcije na novu i obavještava driver o zamjeni. 4. Driver oslobađa staru konekciju. 5. Connection Manager odspaja staru konekciju. 6. Pri sljedećem spajanju (UC2) driver radi nad novom konekcijom. |
| **Alternativni tok** | Nova konekcija ne uspije: stara konekcija ostaje, driver nastavlja raditi i ništa se ne gubi. |
| **Ishod** | Driver radi nad novom konekcijom od sljedećeg spajanja |

#### UC6 — Razvij ili održavaj driver

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Novi podsustav treba driver ili postojeći driver treba prilagodbe |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja driver i navodi njegove metapodatke. 2. Novi ili izmijenjeni driver postaje dostupan konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Driver koji Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Driver: Data access"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor "Read / write strategy" as User
actor "Swap initiator" as Swap
actor "Connection Manager" as CM
actor Programmer
actor "Driver Manager" as DM

rectangle "Driver" as Boundary <<system boundary>> {
    usecase "Sastavi driver" as UC1
    usecase "Spoji driver" as UC2
    usecase "Čitaj i piši podatke" as UC3
    usecase "Odspoji driver" as UC4
    usecase "Zamijeni konekciju" as UC5
    usecase "Razvij ili održavaj\ndriver" as UC6
}

Factory -- UC1
User -- UC3
Swap -- UC5
UC5 -- CM
Programmer -- UC6
UC2 -- DM
UC4 -- DM
@enduml
```

Driver Manager je primarni akter spajanja i odspajanja; tko mu nalaže spajanje prije rada toka, još nije određeno (§6). Zamjenu konekcije pokreće pokretač zamjene, a izvodi Connection Manager; isti slučaj iz perspektive upravitelja opisan je u [`StRS-MANAGER`](StRS-MANAGER.md), UC5.

### 2.5 Stanja drivera

Driver u svakom trenutku ima jedno stanje i prelazi u drugo samo propisanim prijelazom ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md)). Iz stanja DEGRADED nema automatskog oporavka: driver ostaje u njemu dok netko ponovno ne zatraži spajanje ili odspajanje.

| stanje | značenje |
|---|---|
| PENDING | driver je stvoren, ali još nije spojen |
| LOADING | spajanje je u tijeku |
| LIVE | driver je spojen i radi |
| PAUSED | driver je privremeno zaustavljen; u rad se vraća ponovnim spajanjem |
| DEGRADED | spajanje ili odspajanje nije uspjelo; izlaz je ponovni pokušaj spajanja ili odspajanje |
| UNLOADING | odspajanje je u tijeku |
| UNLOADED | driver je odspojen; može se ponovno spojiti ili vratiti u PENDING |

```plantuml
@startuml state-diagram
!include requirements/styles/wattleflow.puml

title "Driver: Data access"
caption "State Diagram"

[*] --> PENDING
PENDING --> LOADING : load
UNLOADED --> LOADING : load
PAUSED --> LOADING : load
DEGRADED --> LOADING : load
LOADING --> LIVE : load ok
LOADING --> DEGRADED : load fail
LIVE --> PAUSED : pause
LIVE --> UNLOADING : unload
PAUSED --> UNLOADING : unload
DEGRADED --> UNLOADING : unload
UNLOADING --> UNLOADED : unload ok
UNLOADING --> DEGRADED : unload fail
UNLOADED --> PENDING : reset
@enduml
```

### 2.6 Tok rada i praćenje grešaka

Dijagram prikazuje životni ciklus drivera u normalnom radu (od sastavljanja preko čitanja i pisanja do odspajanja) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Driver: Data access"
caption "Flow Chart"

start
:Workflow Factory assembles driver\nfrom configuration;
if (metadata valid?) then (no)
  :<b><color:red>metadata error</color></b>;
  stop
endif
:Driver Manager connects driver;
if (load allowed in current state?) then (no)
  :<b><color:red>DriverException</color></b>;
  :error returned to Driver Manager;
  stop
endif
:load resource;
if (load succeeded?) then (no)
  :state DEGRADED;
  :<b><color:red>resource error</color></b>;
  :error returned to Driver Manager;
  stop
endif
:read / write data\n(driver handed over by\nWorkflow Factory at assembly);
:result returned to user;
:Driver Manager disconnects driver;
if (close succeeded?) then (no)
  :state DEGRADED;
  :<b><color:red>resource error</color></b>;
  stop
endif
:state UNLOADED;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| greška metapodataka (`TypeError`, `ValueError`) | sposobnost je nepoznata, ponovljena ili neispravno zadana | driver se ne stvara |
| `DriverException` | spajanje je zatraženo u stanju koje ga ne dopušta | spajanje se odbija; Driver Manager prima grešku |
| greška resursa pri spajanju | resurs se ne može pripremiti za rad | driver prelazi u DEGRADED; izvorna greška prosljeđuje se Driver Manageru |
| greška resursa pri odspajanju | resurs se ne može zatvoriti | driver prelazi u DEGRADED; greška se prosljeđuje, osim kad driver prestaje postojati |
| zamjena konekcije ne uspije | nova konekcija ne proradi | stara konekcija ostaje; driver nastavlja raditi |
| upozorenje | zatražena je radnja koju driver ne podržava | radnja se ne izvodi; događaj se bilježi |
| bez učinka | pauza ili povratak u početno stanje zatraženi u stanju koje ih ne dopušta | ništa se ne mijenja |

Izvorna greška resursa prosljeđuje se nepromijenjena, s uzrokom. Driver bilježi svoje događaje: neuspjelo spajanje i odspajanje, nepodržane radnje, obavijesti o zamjeni konekcije i gašenje.

### 2.7 Pretpostavke i ograničenja

- Driver ne zna tko ga koristi ni zašto; samo čita i piše podatke u svoj resurs.
- Korisnici do drivera dolaze isključivo preko Driver Managera.
- Driver nema automatski oporavak; ponovni pokušaj spajanja uvijek pokreće netko izvana.
- Sposobnosti drivera biraju se iz zatvorenog popisa koji propisuje framework.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-DRV-01** | Driver nudi jedno zajedničko sučelje za čitanje i pisanje, neovisno o vrsti resursa i obliku podataka. |
| **BR-DRV-02** | Workflow Factory pri sastavljanju dohvaća driver od Driver Managera po imenu i predaje ga komponenti koja ga je navela; repozitorij ga predaje svojim strategijama. Više komponenti može dijeliti isti driver i resurs. |
| **BR-DRV-03** | Driver se spaja samo iz stanja PENDING, UNLOADED, PAUSED ili DEGRADED. Zahtjev za spajanje iz drugog stanja odbija se greškom, osim kad je driver već spojen ili se spaja; tada zahtjev nema učinka. |
| **BR-DRV-04** | Driver se spaja na zahtjev Driver Managera (UC2); pri čitanju ili pisanju ne spaja se sam. |
| **BR-DRV-06** | Neuspjelo spajanje ili odspajanje vodi u stanje DEGRADED; izvorna greška prosljeđuje se onome tko je zatražio radnju. |
| **BR-DRV-07** | Iz stanja DEGRADED izlazi se samo ponovnim spajanjem ili odspajanjem; automatskog oporavka nema. |
| **BR-DRV-08** | Pauza je moguća samo iz stanja LIVE; iz pauze se driver vraća ponovnim spajanjem. |
| **BR-DRV-09** | Odspojen driver može se vratiti u početno stanje PENDING. |
| **BR-DRV-10** | Pri zamjeni konekcije nova konekcija mora proraditi prije nego što se stara odspoji. Ako nova konekcija ne uspije, stara ostaje. |
| **BR-DRV-11** | Na obavijest o zamjeni konekcije driver oslobađa staru konekciju i pri sljedećem spajanju radi nad novom. |
| **BR-DRV-12** | Kad driver prestane postojati, sam se odspaja; greška pri tome se ne prosljeđuje. |
| **BR-DRV-13** | Sposobnosti drivera navode se u metapodacima isključivo iz zatvorenog popisa, svaka najviše jednom. |
| **BR-DRV-14** | Radnja koju driver ne podržava ne izvodi se; bilježi se upozorenje. |

## 4. Nefunkcionalni zahtjevi

Povezivanje s NFRQ zapisima je otvoreno (§6). Iz narativa i koda proizlaze ove kakvoće:

| kakvoća | što znači za driver |
|---|---|
| proširivost | novi driveri za nove podsustave dodaju se bez izmjene korisnika (UC6) |
| kompatibilnost | jedno sučelje za resurse u lokalnoj mreži, na internetu i u cloudu (BR-DRV-01) |
| učinkovitost | više korisnika dijeli isti driver i resurs (BR-DRV-02) |
| otpornost | zamjena konekcije ne prekida rad i ne gubi podatke ako nova konekcija ne uspije (BR-DRV-10) |
| praćenje | driver bilježi neuspjela spajanja i odspajanja, nepodržane radnje i zamjene konekcije |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena.
2. Naziv sloja u naslovima dijagrama („Data access") je prijedlog.
3. Narativ (§2.1) sastavljen je iz autorovih odgovora; odlomak *Problem* je prijedlog i treba ga potvrditi.
4. **Zamjena konekcije.** Connection Manager je izvodi, ali je trenutačno ništa ne pokreće. Treba odrediti tko je pokretač zamjene i u kojim situacijama (npr. nove vjerodajnice, drugi poslužitelj).
5. **Pauza.** Treba odrediti tko pauzira driver i zašto.
6. **Povratak u početno stanje.** Treba odrediti tko vraća odspojen driver u stanje PENDING i zašto.
7. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.
8. **Tko spaja drivere.** Workflow Factory drivere stvara i predaje komponentama, ali ih ne spaja ([`StRS-FACTORY`](StRS-FACTORY.md)), a driver se pri čitanju ili pisanju ne spaja sam (BR-DRV-04). Nije određeno tko Driver Manageru nalaže spajanje prije rada toka.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard; njegovi repozitoriji koriste drivere preko strategija čitanja i pisanja |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji driver predaje komponentama |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | stroj stanja |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor, jedan od mogućih korisnika drivera |
| [`StRS-REPOSITORY`](StRS-REPOSITORY.md) | repozitorij čije strategije koriste driver |
| [`StRS-MANAGER`](StRS-MANAGER.md) | upravitelji drivera i konekcija |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | konekcija preko koje driver radi |
| `concrete/driver.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.5 | 2026-10-11 | Prema kodu Workflow Factoryja: driver komponentama (repozitorij, procesor, izlaz monitora) dohvaća Workflow Factory od Driver Managera, a strategije ga dobivaju od repozitorija (BR-DRV-02, UC1, UC3, dionici, kontekst, tok). Uklonjena zaostala rečenica o samostalnom spajanju. *Otvoreno*: pitanje o putu do drivera zamijenjeno pitanjem tko spaja drivere. Poveznice na StRS-FACTORY i StRS-STATE-MACHINE. |
| v0.0.4 | 2026-10-11 | Odgođeno stvaranje drivera (`LazyDriverProxy`) izvan opsega: služi specijalizacijama u paketu *blackwattle*. Uklonjeno pravilo BR-DRV-05 (oznaka se ne koristi ponovno); BR-DRV-04 kaže da driver spaja Driver Manager; UC2, UC3, UC5, dijagram korisničkih slučajeva i dijagram toka usklađeni; uklonjene dvije stavke iz *Otvoreno*. |
| v0.0.3 | 2026-10-11 | Samostalno spajanje pri čitanju ili pisanju vrijedi samo za odgođeni driver (BR-DRV-04, UC3). *Otvoreno*: dodana pitanja o tome kako strategije dolaze do drivera i o odgođenom driveru u repozitoriju; dodane poveznice na StRS-REPOSITORY. |
| v0.0.2 | 2026-10-11 | Usklađeno sa StRS-MANAGER: *veza* je posvuda *konekcija*; driver se dohvaća samo po imenu (ne i po tipu), i u narativu; zamjenu konekcije pokreće pokretač zamjene, a izvodi Connection Manager (UC5, dionici, dijagrami). Dodane poveznice na StRS-MANAGER i StRS-CONNECTION. |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz koda `concrete/driver.py` i autorovih odgovora na ključna pitanja. |