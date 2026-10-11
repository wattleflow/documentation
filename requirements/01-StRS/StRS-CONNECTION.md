<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-CONNECTION — Generička konekcija: jedina granica prema vanjskom sustavu

| | |
|---|---|
| **Verzija** | v0.0.4 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/connection.py`: `GenericConnection`, `ConnectionState`, `ConnectionAction`, `ConnectionObserverInterface` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-DRIVER`](StRS-DRIVER.md) — driver radi kroz konekciju · [`StRS-MANAGER`](StRS-MANAGER.md) — upravitelj konekcija · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja konekciju · [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) — stroj stanja |
| **Dijagrami** | inline (§2.2, §2.4, §2.5, §2.6) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi generička konekcija, tko je koristi, kako se uspostavlja i zatvara veza prema vanjskom sustavu, u kojim se stanjima konekcija može nalaziti i što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** sastavljanje konekcije, stvaranje i zatvaranje veze prema vanjskom sustavu, otvaranje i zatvaranje sesija, stanja konekcije, obavještavanje promatrača i greške koje konekcija javlja.

**Izvan opsega:** protokol i vrsta pojedinog vanjskog sustava (baza, datotečni poslužitelj, poštanski sustav i sl.), upravljanje skupom imenovanih konekcija i njihova zamjena ([`StRS-MANAGER`](StRS-MANAGER.md)) te rad drivera koji konekciju koristi ([`StRS-DRIVER`](StRS-DRIVER.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| konekcija | Connection | dio sustava koji jedini zna kako doći do vanjskog sustava i u kojem je stanju veza prema njemu |
| vanjski sustav | External system | sustav izvan wattleflowa kojem se pristupa kroz konekciju, npr. baza ili poslužitelj |
| veza | engine / pool | dugotrajan resurs prema vanjskom sustavu koji se stvara jednom i zatvara kad više nije potreban; služi za više sesija. Stvaranje i zatvaranje veze nije isto što i otvaranje i zatvaranje sesije. U ostalim zapisima riječ *veza* ne koristi se kao drugi naziv za konekciju |
| sesija | session | jedan zahvat unutar već stvorene veze: otvara se kad je potreban pristup vanjskom sustavu, a zatvara čim je posao gotov; veza pritom ostaje |
| driver | Driver | dio sustava koji kroz konekciju razgovara s vanjskim sustavom ([`StRS-DRIVER`](StRS-DRIVER.md)) |
| upravitelj konekcija | ConnectionManager | dio sustava koji drži imenovane konekcije, prosljeđuje im zahtjeve za povezivanje i odspajanje te ih zatvara pri uklanjanju i gašenju ([`StRS-MANAGER`](StRS-MANAGER.md)) |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja konekciju i registrira je kod upravitelja konekcija |
| promatrač | Observer | dio sustava koji se pretplati na konekciju i dobiva obavijesti o njezinim događajima |
| odgođeno stvaranje | lazy loading | postavka kojom se veza ne stvara pri sastavljanju konekcije, nego tek kad zatreba |
| stanje | state | trenutna faza rada konekcije (§2.5) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Tok podataka mora pristupati vanjskim sustavima, a svaki od njih ima svoj način povezivanja, svoje kvarove i svoje vrijeme života veze. Ako svaki dio toka sam otvara i zatvara vezu, veze ostaju otvorene, kvarovi se prijavljuju na različite načine, a nitko ne zna je li veza uopće ispravna.

**Rješenje** Generička konekcija jedina je granica prema vanjskom sustavu. Svi ostali dijelovi sustava poznaju samo njezino ponašanje, nikad protokol vanjskog sustava. Konekcija razlikuje dugotrajnu vezu, koja se stvara jednom, i kratkotrajne sesije, koje se otvaraju i zatvaraju prema potrebi, te u svakom trenutku zna u kojem je stanju.

**Smjer** Konekcija je zamišljena kao općenita ljuska: određuje redoslijed rada, stanja i prijavljivanje grešaka, a konkretna vrsta konekcije (za određeni vanjski sustav) dodaje samo ono što je specifično za taj sustav. Konkretna vrsta sama otvara i zatvara sesiju, jer jedino ona zna kad sesija počinje i završava. Konekcija se sastavlja prema konfiguraciji, ima jedinstveno ime, a promatrače obavještava o događajima bez ikakvog znanja o tome što oni s obavijestima rade.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja konekciju i registrira je kod upravitelja konekcija; konekcija prema postavci odmah ili tek na prvi zahtjev stvara vezu prema vanjskom sustavu. Driver konekciju dohvaća od upravitelja po imenu. Ako veza još ne postoji, driver povezivanje traži izravno od konekcije, a zatim kroz nju otvara sesiju, razgovara s vanjskim sustavom i sesiju zatvara. Upravitelj konekcija prosljeđuje konekciji zahtjeve za povezivanje i odspajanje, a pri uklanjanju i gašenju zatvara svoje konekcije. Promatrači dobivaju obavijesti o događajima konekcije. Greške se vraćaju onome tko je radnju zatražio.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Connection: External access"
caption "Context Diagram"

actor "Workflow Factory" as Factory
component "Driver" as Driver
component "ConnectionManager" as Manager
component "Observer" as Observer
database "External system" as External

rectangle "Connection" as Boundary <<system boundary>> {
  component Connection
}

Factory ..> Connection : assembles
Manager ..> Connection : requests connect,\ndisconnect, closes
Driver ..> Connection : requests connect,\nopens, closes session
Connection --> Driver : session
Connection --> Observer : notifies
Connection --> External : creates, closes\nlink
Driver --> External : sends, receives
Connection --> Manager : outcome / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste konekcija | izrađuje konekciju za novi vanjski sustav i održava postojeće |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja konekciju | zadaje joj ime i postavke te je registrira kod upravitelja konekcija, čime je stavlja na raspolaganje ostatku toka |
| Upravitelj konekcija | drži imenovane konekcije | daje konekciju po imenu, prosljeđuje zahtjeve za povezivanje i odspajanje, a pri uklanjanju i gašenju zatvara konekcije |
| Driver | koristi konekciju | dohvaća je od upravitelja po imenu, po potrebi od nje izravno traži povezivanje, otvara sesiju, razgovara s vanjskim sustavom i zatvara sesiju |
| Promatrač | prati događaje | prima obavijesti o događajima konekcije |
| Vanjski sustav | izvor ili odredište podataka | prima vezu i odgovara na zahtjeve |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi konekciju

| | |
|---|---|
| **Akter** | Workflow Factory (primarni), Upravitelj konekcija i Vanjski sustav (podupirući) |
| **Okidač** | Pokretanje toka čija konfiguracija navodi konekciju |
| **Preduvjet** | Konfiguracija za konekciju sadrži ime |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije sastavlja konekciju i daje joj ime. 2. Konekcija preuzima zadane postavke. 3. Ako odgođeno stvaranje nije uključeno, konekcija odmah stvara vezu prema vanjskom sustavu. 4. Workflow Factory registrira konekciju kod upravitelja konekcija. |
| **Alternativni tok** | A1. Ime nije zadano ili je prazno: greška konekcije, konekcija se ne sastavlja. A2. Stvaranje veze ne uspije: konekcija se ne sastavlja, greška se vraća Workflow Factoryju. |
| **Ishod** | Konekcija sastavljena i registrirana; u stanju CREATED, ili NEW ako je uključeno odgođeno stvaranje |

#### UC2 — Koristi vanjski sustav kroz sesiju

| | |
|---|---|
| **Akter** | Driver (primarni), Vanjski sustav (podupirući) |
| **Okidač** | Driver treba pročitati ili zapisati podatke u vanjskom sustavu |
| **Preduvjet** | Driver je od upravitelja dobio konekciju po imenu; veza je stvorena (konekcija u stanju CREATED) |
| **Glavni tijek** | 1. Driver otvara sesiju. 2. Konekcija prelazi u CONNECTING, a po uspjehu u CONNECTED. 3. Driver razgovara s vanjskim sustavom. 4. Driver zatvara sesiju. 5. Konekcija se vraća u CREATED. |
| **Alternativni tok** | A1. Sesija se ne može otvoriti: konekcija se vraća u CREATED, veza ostaje, greška se vraća driveru. A2. Veza nije stvorena (npr. odgođeno stvaranje): driver najprije izravno od konekcije traži povezivanje (UC3), pa tek onda otvara sesiju. |
| **Ishod** | Posao obavljen i sesija zatvorena; veza dostupna za sljedeću sesiju |

Sesija je zahvat unutar veze: njezino zatvaranje ne zatvara vezu. Otvorenu sesiju moguće je dohvatiti samo dok traje; izvan nje konekcija ne daje ništa.

#### UC3 — Zatraži povezivanje ili odspajanje

| | |
|---|---|
| **Akter** | Upravitelj konekcija ili Driver (primarni) |
| **Okidač** | Upravitelj prosljeđuje zahtjev za povezivanje ili odspajanje, ili driver treba vezu koja još ne postoji |
| **Preduvjet** | Konekcija je registrirana u upravitelju |
| **Glavni tijek** | 1. Pozivatelj konekciji predaje zahtjev za povezivanje ili odspajanje. 2. Za povezivanje konekcija osigurava da veza postoji. 3. Za odspajanje konekcija zatvara vezu. 4. Konekcija javlja da je zahtjev prihvaćen. |
| **Alternativni tok** | A1. Preko upravitelja je zatražena radnja koja nije povezivanje ni odspajanje: zahtjev se odbija odgovorom „ne" i upozorenjem; izravno zatražena nepoznata radnja odbija se greškom konekcije. A2. Radnja nije dopuštena u trenutnom stanju (npr. povezivanje iz stanja FAILED): greška; prije novog stvaranja veza se mora zatvoriti ili vratiti u početno stanje (§2.5). |
| **Ishod** | Veza stvorena ili zatvorena prema zahtjevu |

Zahtjev za povezivanje ne otvara sesiju: osigurava samo vezu. Ako ona već postoji, zahtjev se samo bilježi.

#### UC4 — Zatvori konekciju

| | |
|---|---|
| **Akter** | Upravitelj konekcija (primarni), Vanjski sustav (podupirući) |
| **Okidač** | Konekcija se uklanja iz upravitelja, upravitelj se gasi, ili konekcija prestaje postojati |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Konekcija prelazi u CLOSING. 2. Veza se zatvara. 3. Konekcija prelazi u CLOSED. |
| **Alternativni tok** | A1. Konekcija je već zatvorena ili veza nikad nije stvorena: ništa se ne događa. A2. Zatvaranje veze ne uspije: konekcija prelazi u FAILED, greška se vraća onome tko je zatvaranje zatražio. |
| **Ishod** | Konekcija u stanju CLOSED; iz njega se veza može ponovno stvoriti |

Pri uklanjanju i gašenju upravitelj zatvara svaku konekciju, bez obzira na to je li sesija u tom trenutku otvorena (BR-CON-17). Zatvaranje je moguće iz svakog stanja osim NEW i CLOSED, pa i iz FAILED. Kad konekcija prestane postojati, zatvara se sama; kvar pri tom zatvaranju ne prekida rad nikoga drugog.

#### UC5 — Prati događaje konekcije

| | |
|---|---|
| **Akter** | Promatrač |
| **Okidač** | Promatrač treba znati što se događa s konekcijom |
| **Preduvjet** | Promatrač je prepoznat kao promatrač i ima ime |
| **Glavni tijek** | 1. Promatrač se pretplaćuje na konekciju. 2. Konekcija pri svakom događaju obavještava sve promatrače. |
| **Alternativni tok** | A1. Pretplaćuje se nešto što nije promatrač: pretplata se odbija greškom. A2. Promatrač s istim imenom već je pretplaćen: pretplata se zanemaruje. A3. Promatrač javi grešku pri obavijesti: bilježi se upozorenje, ostali promatrači nastavljaju. |
| **Ishod** | Promatrač prima obavijesti |

Pri zamjeni konekcije u hodu upravitelj konekcija prenosi promatrače stare konekcije na novu, a nova konekcija šalje obavijest o zamjeni. Zamjenu pokreće pokretač zamjene, a provodi upravitelj ([`StRS-MANAGER`](StRS-MANAGER.md), UC5); ovdje je opisan samo udio konekcije.

#### UC6 — Razvij ili održavaj konekciju

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Tok treba pristup novom vanjskom sustavu ili postojeća konekcija treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu konekcije i određuje kako se stvara i zatvara veza te kako se otvara sesija. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta konekcije koju Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Connection: External access"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Driver
actor "ConnectionManager" as Manager
actor Observer
actor Programmer
actor "External system" as External

rectangle "Connection" as Boundary <<system boundary>> {
    usecase "Sastavi konekciju" as UC1
    usecase "Koristi vanjski sustav\nkroz sesiju" as UC2
    usecase "Zatraži povezivanje\nili odspajanje" as UC3
    usecase "Zatvori konekciju" as UC4
    usecase "Prati događaje\nkonekcije" as UC5
    usecase "Razvij ili održavaj\nkonekciju" as UC6
}

Factory -- UC1
Driver -- UC2
Driver -- UC3
Manager -- UC3
Manager -- UC4
Observer -- UC5
Programmer -- UC6
UC1 -- External
UC2 -- External
UC4 -- External
@enduml
```

### 2.5 Stanja konekcije

Konekcija u svakom trenutku ima jedno stanje i prelazi u drugo samo na propisani događaj ([`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md)). Time se sprječava, na primjer, otvaranje sesije na vezi koja nije stvorena. Stanje CONNECTED znači da je sesija otvorena; kad se sesija zatvori, konekcija se vraća u CREATED jer veza i dalje postoji. Iz stanja FAILED ne može se izravno stvoriti nova veza: najprije se konekcija zatvori ili vrati u NEW, pa se veza stvara ponovno. Ponovni pokušaj ne događa se sam od sebe.

| stanje | značenje |
|---|---|
| NEW | konekcija je sastavljena, ali veza još nije stvorena |
| CREATING | veza se stvara |
| CREATED | veza postoji, sesija nije otvorena |
| CONNECTING | sesija se otvara |
| CONNECTED | sesija je otvorena i vanjski sustav je dostupan |
| CLOSING | veza se zatvara |
| CLOSED | veza je zatvorena |
| FAILED | stvaranje ili zatvaranje veze nije uspjelo; iz njega je moguće samo zatvaranje ili povratak u NEW |

```plantuml
@startuml state-diagram
!include requirements/styles/wattleflow.puml

title "Connection: External access"
caption "State Diagram"

[*] --> NEW
NEW --> CREATING : create
CLOSED --> CREATING : create
CREATING --> CREATED : create ok
CREATING --> FAILED : create fail
CREATED --> CONNECTING : connect
CONNECTING --> CONNECTED : connect ok
CONNECTING --> CREATED : connect fail
CONNECTED --> CREATED : disconnect
CREATING --> CLOSING : close
CREATED --> CLOSING : close
CONNECTING --> CLOSING : close
CONNECTED --> CLOSING : close
FAILED --> CLOSING : close
CLOSING --> CLOSED : close ok
CLOSING --> FAILED : close fail
CLOSED --> NEW : reset
FAILED --> NEW : reset
@enduml
```

### 2.6 Tok rada i praćenje grešaka

Dijagram prikazuje normalni tok (od sastavljanja konekcije do zatvaranja) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Connection: External access"
caption "Flow Chart"

start
:Workflow Factory assembles connection\nfrom configuration;
if (name given?) then (no)
  :<b><color:red>ConnectionException</color></b>;
  :error returned to Workflow Factory;
  stop
endif
if (lazy loading off?) then (yes)
  :create link;
  if (link built?) then (no)
    :<b><color:red>error of the connection type</color></b>;
    :connection not assembled,\nerror returned to Workflow Factory;
    stop
  endif
endif
:Driver gets connection from manager;
:Driver requests connect directly\n(only if link is missing);
if (creation allowed in current state?) then (no)
  :<b><color:red>ManagerException</color></b>;
  :error returned to Driver;
  stop
endif
:open session;
if (session opened?) then (no)
  :state CREATED;
  :<b><color:red>error of the connection type</color></b>;
  :error returned to Driver;
  stop
endif
:Driver works with external system;
:close session, state CREATED;
:manager closes connection\n(on removal or shutdown);
if (link torn down?) then (no)
  :state FAILED;
  :<b><color:red>error of the connection type</color></b>;
  :error returned to caller;
  stop
endif
:state CLOSED;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ConnectionException` | ime konekcije nije zadano ili je prazno; izravno je zatražena nepoznata radnja | konekcija se ne sastavlja ili se zahtjev odbija; onaj tko ga je zatražio prima grešku |
| `ManagerException` | stvaranje veze zatraženo je u stanju u kojem nije dopušteno (npr. povezivanje iz stanja FAILED ili dok se sesija otvara) | veza se ne stvara; onaj tko ju je zatražio prima grešku |
| greška vrste konekcije pri stvaranju veze | stvaranje veze ne uspije | konekcija prelazi u FAILED; izvorna greška vraća se onome tko je stvaranje zatražio; pri sastavljanju se konekcija ne sastavlja |
| greška vrste konekcije pri otvaranju sesije | sesija se ne može otvoriti | konekcija se vraća u CREATED, veza ostaje; driver prima grešku |
| greška vrste konekcije pri zatvaranju veze | zatvaranje veze ne uspije | konekcija prelazi u FAILED; izvorna greška vraća se onome tko je zatvaranje zatražio, osim pri samozatvaranju (tada se zanemaruje) |
| `TypeError` | pretplaćuje se nešto što nije promatrač | pretplata se odbija |
| upozorenje | promatrač javi grešku pri obavijesti; preko upravitelja je zatražena radnja koju konekcija ne podržava (odgovor „ne") | rad se nastavlja; događaj se bilježi |

Konekcija bilježi svoje događaje: sastavljanje, stvaranje i zatvaranje veze, otvaranje i zatvaranje sesije te greške.

### 2.7 Pretpostavke i ograničenja

- Generička konekcija je samo šablona: konkretne vrste konekcija (za pojedine baze, poslužitelje ili redove poruka) u ovom modulu ne postoje i izrađuju se zasebno (UC6).
- Konekcija nema stanje pauze. Najbliže tome je neuspjelo otvaranje sesije, nakon kojeg se konekcija vraća u CREATED i veza ostaje.
- Konekcija sama ne zna ništa o protokolu vanjskog sustava: to znaju konkretne vrste konekcija.
- Konekcija ne obavlja posao nad podacima; podatke čita i zapisuje driver.
- Veza se stvara jednom i dijeli među sesijama; sesije se otvaraju jedna po jedna kroz istu konekciju.
- Verziju vanjskog sustava konekcija ne određuje sama: ako je poznata, potječe iz odgovora samog vanjskog sustava, a inače je nepoznata.
- Konekcija se oslanja na pretplaćene promatrače samo za obavijesti; njihov kvar ne smije zaustaviti konekciju.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-CON-01** | Konekcija je jedina granica prema vanjskom sustavu: ostali dijelovi sustava poznaju njezino ponašanje, a ne protokol vanjskog sustava. |
| **BR-CON-02** | Svaka konekcija ima ime; bez imena se ne sastavlja. |
| **BR-CON-03** | Veza se stvara jednom i služi za više sesija; zatvaranje sesije ne zatvara vezu. |
| **BR-CON-04** | Veza se stvara pri sastavljanju konekcije, osim ako je uključeno odgođeno stvaranje; tada se stvara na prvi zahtjev. |
| **BR-CON-05** | Konekcija mijenja stanje samo propisanim prijelazima (§2.5). |
| **BR-CON-06** | Otvorena sesija dostupna je samo dok traje. |
| **BR-CON-07** | Zahtjev za povezivanje osigurava samo vezu; sesiju otvara onaj tko vezu koristi. |
| **BR-CON-08** | Neuspjeh stvaranja ili zatvaranja veze stavlja konekciju u stanje FAILED; iz njega se konekcija može samo zatvoriti ili vratiti u početno stanje, a ne izravno ponovno povezati. |
| **BR-CON-09** | Neuspjeh otvaranja sesije ne zatvara vezu: konekcija se vraća u stanje u kojem je sesija mogla početi. |
| **BR-CON-10** | Zatvorena ili neuspjela konekcija može se vratiti u početno stanje i ponovno stvoriti. |
| **BR-CON-11** | Kad konekcija prestane postojati, zatvara se sama; kvar pri tome ne smije prekinuti rad drugih. |
| **BR-CON-12** | Pretplatiti se mogu samo promatrači; promatrač s već pretplaćenim imenom ne pretplaćuje se ponovno. |
| **BR-CON-13** | Kvar jednog promatrača ne sprječava obavještavanje ostalih ni rad konekcije; bilježi se kao upozorenje. |
| **BR-CON-14** | Konekcija prima samo zahtjeve za povezivanje i odspajanje; svaku drugu radnju odbija. |
| **BR-CON-15** | Verziju vanjskog sustava konekcija navodi samo ako ju je sam vanjski sustav prijavio; inače je nepoznata. |
| **BR-CON-16** | Konekcija se sama ne pokušava oporaviti iz stanja FAILED: oporavak (zatvaranje ili povratak u početno stanje, pa ponovno stvaranje veze) pokreće netko izvana. |
| **BR-CON-17** | Kad se konekcija uklanja iz upravitelja ili se upravitelj gasi, upravitelj je zatvara, bez obzira na to je li sesija u tom trenutku otvorena. |

## 4. Nefunkcionalni zahtjevi

Povezivanje s NFRQ zapisima je otvoreno (§6). Iz narativa i koda proizlaze ove kakvoće:

| kakvoća | što znači za konekciju |
|---|---|
| pouzdanost | kvar pri stvaranju ili zatvaranju veze vidljiv je u stanju FAILED, a kvar sesije ne zatvara vezu (BR-CON-08, BR-CON-09) |
| učinkovitost | veza se stvara jednom i dijeli među sesijama (BR-CON-03) |
| urednost | pri uklanjanju i gašenju ne ostaju otvorene veze (BR-CON-17) |
| praćenje | konekcija bilježi sastavljanje, stanja, sesije i greške te obavještava promatrače |
| proširivost | programer može izraditi novu vrstu konekcije za svaki vanjski sustav (UC6) |
| odvojenost | zamjena vanjskog sustava ne dira ostatak toka jer on ne poznaje protokol (BR-CON-01) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-CON-*` i naziv sloja u naslovima dijagrama („External access") su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Zadano ponašanje: konekcija vezu stvara odmah pri sastavljanju (BR-CON-04), osim uz odgođeno stvaranje. Treba potvrditi da je to željeno zadano ponašanje.
4. **Vrste grešaka.** Konekcija javlja tri vrste grešaka: vlastitu, grešku upravitelja (kad stvaranje veze nije dopušteno u trenutnom stanju) i izvornu grešku vrste konekcije. Treba potvrditi je li to željeno.
5. Oporavak iz FAILED nema vlasnika: ponovni pokušaj se ne događa sam, a nije određeno tko ga pokreće, kada, ni što FAILED znači za posao toka. Isto vrijedi za povratak iz CLOSED u početno stanje (BR-CON-10, BR-CON-16). Treba odrediti akter i okidač, pa po potrebi dodati korisnički slučaj.
6. Zamjena konekcije u hodu: pokretač zamjene nije određen; isto pitanje otvoreno je u [`StRS-MANAGER`](StRS-MANAGER.md) i [`StRS-DRIVER`](StRS-DRIVER.md).
7. Sloj: nije odlučeno pripada li konekcija sloju „Data persistence" ili zasebnom sloju (infrastruktura). Naziv „External access" u naslovima dijagrama je privremen; isto pitanje otvoreno je u [`StRS-MANAGER`](StRS-MANAGER.md).
8. **Zatvaranje pri uklanjanju i gašenju.** BR-CON-17 traži da upravitelj zatvori svaku konekciju, a upravitelj trenutačno zatvara samo konekcije s otvorenom sesijom; ostale se zatvaraju tek kad same prestanu postojati. Kod treba uskladiti s pravilom (SyRS).
9. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driver koji radi kroz konekciju |
| [`StRS-MANAGER`](StRS-MANAGER.md) | upravitelj konekcija |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja konekciju |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | stroj stanja |
| `concrete/connection.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.4 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.3 | 2026-10-11 | Poveznice na StRS-FACTORY i StRS-STATE-MACHINE. |
| v0.0.2 | 2026-10-11 | Usklađeno s kodom i povezanim StRS-ovima: konekciju sastavlja Workflow Factory; driver povezivanje traži izravno od konekcije (UC2, UC3); obavijest o zamjeni šalje nova konekcija (UC5); neuspjelo stvaranje veze pri sastavljanju znači da se konekcija ne sastavlja (UC1). Novo pravilo BR-CON-17: upravitelj pri uklanjanju i gašenju zatvara sve konekcije. Jedinstveni naslovi dijagrama; poveznice na StRS-MANAGER umjesto FRQ-MGR; ispravljen red riječi („veza se stvara"); BR-CON-06 bez ponavljanja. *Otvoreno* očišćeno od zastarjelih stavki, a pitanje o vrstama grešaka postavljeno na razini StRS-a. |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz koda `concrete/connection.py`. |