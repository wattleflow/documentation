<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-REPOSITORY — Repozitorij: trajna pohrana dokumenata kroz strategije

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/repository.py`: `GenericRepository`, `RepositoryWithDriver` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — predaje dokumente repozitorijima · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — objavljuje pravila pohrane · [`StRS-DRIVER`](StRS-DRIVER.md) — driver kroz koji strategije dolaze do vanjskog resursa · [`StRS-DOCUMENT`](StRS-DOCUMENT.md) — dokument koji se pohranjuje · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja repozitorij i predaje mu driver |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi repozitorij, tko ga koristi, kako dokumenti kroz njega odlaze u trajnu pohranu i dolaze iz nje te što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** sastavljanje repozitorija, pisanje dokumenata kroz strategiju pisanja, čitanje kroz strategiju čitanja, brojanje potvrđenih pisanja, repozitorij koji radi preko drivera i greške koje repozitorij javlja.

**Izvan opsega:** kako Blackboard dokumente drži i kada ih predaje ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md)), što strategije točno rade s vanjskim resursom i kako driver s njim razgovara ([`StRS-DRIVER`](StRS-DRIVER.md)) te sadržaj dokumenta ([`StRS-DOCUMENT`](StRS-DOCUMENT.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| repozitorij | Repository | mjesto trajne pohrane dokumenata; sam ne zna kako se pohranjuje, to zna njegova strategija |
| dokument | document | strukturirana ili nestrukturirana jedinica podataka koja se pohranjuje (isti pojam u cijelom sloju postojanosti podataka) |
| fasada | Facade | omotač kroz koji repozitorij vidi dokument ([`StRS-DOCUMENT`](StRS-DOCUMENT.md)) |
| strategija pisanja | Write strategy | dio koji dokument zapisuje u vanjski resurs; obvezna je |
| strategija čitanja | Read strategy | dio koji dokument čita iz vanjskog resursa; nije obvezna |
| potvrda | confirmation | odgovor strategije pisanja da je dokument zapisan |
| brojač | counter | broj dokumenata koje je repozitorij zapisao uz potvrdu |
| pravila pohrane | write context | pravila koja procesor objavljuje za svoj skup dokumenata, a Blackboard ih predaje uz dokument |
| Blackboard | Blackboard | zajednički međuspremnik koji dokumente predaje repozitorijima |
| driver | Driver | dio sustava koji razgovara s vanjskim resursom ([`StRS-DRIVER`](StRS-DRIVER.md)) |
| vanjski resurs | External resource | baza, datoteka, servis ili drugo mjesto na koje se pohranjuje |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja repozitorij, predaje mu driver i registrira ga kod Blackboarda ([`StRS-FACTORY`](StRS-FACTORY.md)) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Dokumenti koji su obrađeni moraju završiti u trajnoj pohrani, a pohrana može biti bilo što: baza, datoteka ili servis. Kad bi svaki dio toka sam znao kamo i kako zapisuje, tok bi bio vezan uz pojedinu pohranu, a promjena odredišta tražila bi izmjene na mnogo mjesta.

**Rješenje** Repozitorij je jedno mjesto kroz koje dokumenti odlaze u trajnu pohranu i vraćaju se iz nje. On sam ne zna ništa o pohrani: dokument zapisuje svojom strategijom pisanja, a čita svojom strategijom čitanja. Vodi i broj dokumenata koje je uspješno zapisao.

**Smjer** Repozitorij se sastavlja prema konfiguraciji toka s obveznom strategijom pisanja i neobveznom strategijom čitanja. Posebna vrsta repozitorija radi preko drivera: driver dobiva svaka strategija. Tko dokumente predaje i kada, određuje Blackboard. Repozitorij ih samo prima i javlja je li ih strategija potvrdila.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja repozitorij i registrira ga kod Blackboarda. Pri predaji Blackboard šalje repozitoriju dokument po dokument, a repozitorij ga predaje svojoj strategiji pisanja. Strategija dokument zapisuje u vanjski resurs, najčešće preko drivera, i javlja je li ga zapisala. Repozitorij o tome javlja Blackboardu. Čitanje ide obrnutim smjerom: čitatelj traži dokument, repozitorij ga traži od svoje strategije čitanja i vraća ga čitatelju. Tko je čitatelj, još nije određeno (§6). Greške se vraćaju onome tko je radnju zatražio.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Repository: Data persistence"
caption "Context Diagram"

actor "Workflow Factory" as Factory
actor "Reader\n(not determined)" as Reader
component "Blackboard" as Blackboard
component "Write strategy" as Write
component "Read strategy" as Read
component "Driver\n(optional)" as Driver
database "External resource" as Resource

rectangle "Repository" as Boundary <<system boundary>> {
  component Repository
}

Factory ..> Repository : assembles
Factory ..> Blackboard : registers Repository
Blackboard ..> Repository : writes documents
Reader ..> Repository : reads document
Repository ..> Write : writes through
Repository ..> Read : reads through
Write ..> Driver : uses
Read ..> Driver : uses
Driver --> Resource : writes, reads
Repository --> Blackboard : confirmation / error
Repository --> Reader : document / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste repozitorija | izrađuje repozitorije za nova odredišta i određuje što svaka strategija dobiva od repozitorija |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja repozitorij | stvara repozitorij sa strategijama, od upravitelja drivera po imenu dohvaća driver i predaje ga repozitoriju te repozitorij registrira kod Blackboarda |
| Blackboard | predaje dokumente | šalje repozitoriju dokumente na pohranu i prima potvrdu |
| Strategija pisanja | zapisuje dokument | zapisuje dokument u vanjski resurs i javlja potvrdu |
| Strategija čitanja | čita dokument | vraća dokument iz vanjskog resursa |
| Driver | pristup vanjskom resursu | omogućuje strategijama da govore s resursom (samo kod repozitorija koji radi preko drivera) |
| Čitatelj | traži pohranjeni dokument | još nije određen (§6) |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi repozitorij

| | |
|---|---|
| **Akter** | Workflow Factory (primarni), Blackboard (podupirući) |
| **Okidač** | Konfiguracija toka navodi repozitorije za Blackboard koji Workflow Factory sastavlja |
| **Preduvjet** | Konfiguracija navodi vrstu repozitorija, strategiju pisanja i po potrebi strategiju čitanja i driver |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije stvara strategije. 2. Ako vrsta repozitorija traži driver, Workflow Factory ga po imenu iz konfiguracije dohvaća od upravitelja drivera. 3. Workflow Factory stvara repozitorij sa strategijama i driverom. 4. Workflow Factory registrira repozitorij kod Blackboarda. |
| **Alternativni tok** | A1. Strategija pisanja ili čitanja nije ispravne vrste: repozitorij se ne sastavlja, greška repozitorija. A2. Vrsta repozitorija traži driver, a driver nije predan ili nije ispravne vrste: greška repozitorija. |
| **Ishod** | Repozitorij sastavljen, s brojačem na nuli, i registriran kod Blackboarda |

#### UC2 — Zapiši dokument

| | |
|---|---|
| **Akter** | Blackboard (primarni), Strategija pisanja (podupirući) |
| **Okidač** | Blackboard predaje dokumente repozitorijima |
| **Preduvjet** | Repozitorij je sastavljen (UC1) |
| **Glavni tijek** | 1. Blackboard predaje repozitoriju sebe i fasadu dokumenta, uz pravila pohrane ako ih ima. 2. Repozitorij provjerava što je dobio. 3. Repozitorij predaje dokument svojoj strategiji pisanja. 4. Strategija dokument zapisuje u vanjski resurs i javlja potvrdu. 5. Repozitorij povećava brojač ako je dokument potvrđen. 6. Repozitorij javlja Blackboardu je li dokument potvrđen. |
| **Alternativni tok** | A1. Predan Blackboard nije Blackboard ili fasada nije fasada: pisanje se ne izvodi, greška repozitorija. A2. Strategija javi da dokument nije zapisan: brojač se ne mijenja, Blackboardu se javlja da dokument nije potvrđen (nije greška). A3. Zapisivanje ne uspije: greška repozitorija s uzrokom i podacima o strategiji, dokumentu i Blackboardu. |
| **Ishod** | Dokument zapisan u vanjski resurs uz potvrdu, ili Blackboard zna da potvrde nema |

Potvrda repozitorija po dokumentu ista je potvrda koju Blackboard prikuplja pri predaji: predaja je potvrđena samo ako su svi repozitoriji potvrdili sve dokumente ([`BR-BLC-06`](StRS-BLACKBOARD.md#3-poslovna-pravila)).

#### UC3 — Pročitaj dokument

| | |
|---|---|
| **Akter** | Čitatelj (nije određen, §6) |
| **Okidač** | Dio sustava treba dokument koji je već pohranjen |
| **Preduvjet** | Repozitorij je sastavljen (UC1) |
| **Glavni tijek** | 1. Čitatelj traži dokument po oznaci. 2. Repozitorij traži dokument od strategije čitanja. 3. Strategija dokument čita iz vanjskog resursa. 4. Repozitorij vraća fasadu dokumenta. |
| **Alternativni tok** | A1. Repozitorij nema strategiju čitanja: ne vraća ništa, bilježi se upozorenje. A2. Čitanje ne uspije: greška repozitorija s uzrokom. |
| **Ishod** | Čitatelj ima fasadu dokumenta, ili zna da čitanja nema |

#### UC4 — Razvij ili održavaj repozitorij

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Tok treba novo odredište pohrane ili postojeći repozitorij treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu repozitorija i određuje što strategije dobivaju od njega. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta repozitorija koju Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Repository: Data persistence"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Blackboard
actor "Write strategy" as Write
actor "Reader\n(not determined)" as Reader
actor Programmer

rectangle "Repository" as Boundary <<system boundary>> {
    usecase "Sastavi repozitorij" as UC1
    usecase "Zapiši dokument" as UC2
    usecase "Pročitaj dokument" as UC3
    usecase "Razvij ili održavaj\nrepozitorij" as UC4
}

Factory -- UC1
UC1 -- Blackboard
Blackboard -- UC2
UC2 -- Write
Reader -- UC3
Programmer -- UC4
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Repozitorij nema stanja, osim brojača zapisanih dokumenata. Dijagram prikazuje normalni tok (od sastavljanja do pisanja i čitanja) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Repository: Data persistence"
caption "Flow Chart"

start
:Workflow Factory assembles Repository\nfrom configuration;
if (strategies (and driver) valid?) then (no)
  :<b><color:red>RepositoryException</color></b>;
  :error returned to Workflow Factory;
  stop
endif
:Blackboard hands over a document;
if (Blackboard and Facade valid?) then (no)
  :<b><color:red>RepositoryException</color></b>;
  :error returned to Blackboard;
  stop
endif
:write strategy writes document;
if (write failed?) then (yes)
  :<b><color:red>RepositoryException</color></b>\n(with the cause);
  :error returned to Blackboard;
  stop
endif
if (strategy confirmed?) then (yes)
  :counter increased;
  :confirmation returned to Blackboard;
else (no)
  :"not confirmed" returned to Blackboard;
endif
:reader asks for a document;
if (read strategy assigned?) then (no)
  :warning logged;
  :nothing returned;
  stop
endif
:read strategy reads document;
if (read failed?) then (yes)
  :<b><color:red>RepositoryException</color></b>\n(with the cause);
  :error returned to reader;
  stop
endif
:Facade returned to reader;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `RepositoryException` | strategija nije ispravne vrste; kod repozitorija s driverom driver nije predan ili nije ispravne vrste; pri pisanju predan Blackboard ili fasada nisu ispravne vrste; zapisivanje ili čitanje ne uspije | radnja se prekida; onaj tko ju je zatražio prima grešku s uzrokom; pri pisanju greška imenuje strategiju, dokument i Blackboard |
| nepotvrđeno pisanje | strategija javi da dokument nije zapisan | brojač se ne mijenja; Blackboard dobiva ishod „nije potvrđeno" (nije greška) |
| upozorenje | čitanje se traži, a repozitorij nema strategiju čitanja | ne vraća se ništa; događaj se bilježi |

Repozitorij za grešku ostavlja samo trag za detaljnu analizu; grešku bilježi dio sustava koji zaustavlja obradu. Repozitorij bilježi početak i kraj čitanja i pisanja te vraćanje brojača na nulu.

### 2.6 Pretpostavke i ograničenja

- Repozitorij ne zna kamo se i kako pohranjuje: to znaju strategije.
- Repozitorij ne odlučuje kad se piše: dokumente mu predaje Blackboard.
- Brojač broji samo potvrđena pisanja; živi dok živi repozitorij i može se vratiti na nulu, ali se ne pohranjuje.
- Repozitorij može biti samo za pisanje: strategija čitanja nije obvezna.
- Repozitorij čita jedan dokument po oznaci; način na koji strategija nalazi dokument njezina je stvar.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-RPS-01** | Repozitorij dokument zapisuje samo kroz strategiju pisanja i čita samo kroz strategiju čitanja; sam ne zna kamo ni kako se pohranjuje. |
| **BR-RPS-02** | Strategija pisanja je obvezna, strategija čitanja nije. |
| **BR-RPS-03** | Repozitorij se ne sastavlja ako strategija nije ispravne vrste. |
| **BR-RPS-04** | Repozitorij koji radi preko drivera ne sastavlja se bez drivera ispravne vrste. Driver mu pri sastavljanju predaje Workflow Factory ([`BR-FCT-08`](StRS-FACTORY.md#3-poslovna-pravila)), a repozitorij ga predaje svakoj svojoj strategiji. |
| **BR-RPS-05** | Pisanje prima samo od Blackboarda i samo dokument predan kroz fasadu; inače je greška. |
| **BR-RPS-06** | Repozitorij javlja je li strategija dokument potvrdila; nepotvrđen dokument nije greška, ali se ne broji. |
| **BR-RPS-07** | Repozitorij broji potvrđena pisanja; brojač se može vratiti na nulu. |
| **BR-RPS-08** | Neuspjelo pisanje ili čitanje javlja se kao greška repozitorija s izvornim uzrokom; greška pisanja imenuje strategiju, dokument i Blackboard. |
| **BR-RPS-09** | Čitanje bez strategije čitanja ne diže grešku: ne vraća ništa i bilježi upozorenje. |
| **BR-RPS-10** | Pravila pohrane koja objavi procesor ([`BR-PRC-11`](StRS-PROCESSOR.md#3-poslovna-pravila)), a Blackboard preda uz dokument ([`BR-BLC-12`](StRS-BLACKBOARD.md#3-poslovna-pravila)), repozitorij prosljeđuje strategiji pisanja. |
| **BR-RPS-11** | Posebna vrsta repozitorija dodaje strategijama samo ono što im treba (npr. driver); čitanje i pisanje ostaju isti. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; osim ondje gdje je poveznica navedena, još nema zapisa zahtjeva na kakvoću (§6).

| kakvoća | što znači za repozitorij |
|---|---|
| zamjenjivost odredišta | promjena odredišta pohrane mijenja strategije, ne tok (BR-RPS-01) |
| proširivost | programer može izraditi repozitorije za nova odredišta (UC4) |
| razumljivost grešaka | greška pisanja nosi strategiju, dokument i Blackboard (BR-RPS-08) |
| razmjerno bilježenje | uzrok se bilježi jednom, na mjestu gdje se obrada zaustavi ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md)) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-REP-*` i naziv sloja u naslovima dijagrama („Data persistence", kao u [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md)) su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Čitanje: u pregledanom kodu nitko ne traži dokument od repozitorija. Nije određeno tko je čitatelj (UC3).
4. Brojač zapisanih dokumenata: nije određeno tko ga čita ni čemu služi njegovo vraćanje na nulu.
5. **Tko spaja driver.** Workflow Factory driver predaje repozitoriju nespojen, a driver se pri čitanju ili pisanju ne spaja sam ([`BR-DRV-04`](StRS-DRIVER.md#3-poslovna-pravila)). Nije određeno tko ga spaja prije prvog pisanja ([`StRS-FACTORY`](StRS-FACTORY.md)).
6. Kakvoće iz §4 (osim bilježenja) treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard koji predaje dokumente repozitorijima |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor koji objavljuje pravila pohrane |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driver kojim strategije dolaze do resursa |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | dokument koji se pohranjuje |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja repozitorij |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | opseg bilježenja |
| `concrete/repository.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.5 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.4 | 2026-10-11 | Prema kodu Workflow Factoryja: driver repozitoriju dohvaća Workflow Factory od upravitelja drivera i predaje ga pri sastavljanju (UC1, BR-RPS-04, dionici). *Otvoreno*: pitanje o putu do drivera zamijenjeno pitanjem tko spaja driver. Poveznica na StRS-FACTORY. |
| v0.0.3 | 2026-10-11 | Uklonjeno pitanje o odgođenom driveru: odgođeno stvaranje drivera služi paketu *blackwattle* i izvan je opsega. |
| v0.0.2 | 2026-10-11 | Repozitorij sastavlja i kod Blackboarda registrira Workflow Factory. Potvrda po dokumentu povezana s BR-BLC-06 (zatvoreno pitanje o pojmu potvrde); izvor pravila pohrane povezan s BR-PRC-11 i BR-BLC-12; čitatelj dodan na kontekstni dijagram; „brojač zapisa" postaje „brojač zapisanih dokumenata"; odlomak o bilježenju grešaka pojednostavljen. *Otvoreno*: dodana pitanja o tome kako strategije dolaze do drivera i o odgođenom driveru. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/repository.py`. |