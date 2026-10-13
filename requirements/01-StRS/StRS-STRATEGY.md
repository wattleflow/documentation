<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-STRATEGY — Strategija: zamjenjiv način da se obavi jedan posao

| | |
|---|---|
| **Verzija** | v0.0.1 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/strategy.py`: `Strategy`, `StrategyGenerate`, `StrategyCreate`, `StrategyRead`, `StrategyWrite` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — izvodi strategiju stvaranja · [`StRS-REPOSITORY`](StRS-REPOSITORY.md) — izvodi strategije čitanja i pisanja · [`StRS-DOCUMENT`](StRS-DOCUMENT.md) — dokument koji strategije stvaraju, čitaju i zapisuju · [`StRS-DRIVER`](StRS-DRIVER.md) — driver koji strategije koriste · [`SyRS-STR`](../02-SyRS/SyRS-STR-strategy.md) — prethodni zapis iste komponente · [`StRS-FACTORY`](StRS-FACTORY.md) — Workflow Factory koja sastavlja strategije |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi strategija, tko je koristi, koje vrste strategija postoje i što se događa kad strategija ne uspije.

### 1.2 Opseg

**U opsegu:** zajednički okvir svih strategija: četiri namjene (generiranje, stvaranje, čitanje, pisanje), način pozivanja, povratne vrijednosti i javljanje grešaka.

**Izvan opsega:** što pojedina strategija konkretno radi (stvara li tekst ili tablicu, zapisuje li u bazu ili datoteku), kako Blackboard i repozitorij strategije drže i pokreću ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), [`StRS-REPOSITORY`](StRS-REPOSITORY.md)) te kako driver razgovara s resursom ([`StRS-DRIVER`](StRS-DRIVER.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| strategija | Strategy | zamjenjiv način da se obavi jedan posao; pozivatelj ne zna kako radi, zna samo što od nje može tražiti |
| strategija stvaranja | Create strategy | stvara novi dokument i vraća njegovu fasadu |
| strategija čitanja | Read strategy | iz vanjskog resursa čita dokument zadane oznake |
| strategija pisanja | Write strategy | zapisuje dokument u vanjski resurs i javlja je li ga zapisala |
| strategija generiranja | Generate strategy | strategija za nastajanje podataka; u pregledanom kodu nije povezana ni s jednim dijelom toka (§6) |
| dokument | document | strukturirana ili nestrukturirana jedinica podataka (isti pojam u cijelom sloju postojanosti podataka) |
| fasada | Facade | omotač kroz koji se dokument predaje i prima ([`StRS-DOCUMENT`](StRS-DOCUMENT.md)) |
| pozivatelj | caller | dio sustava koji izvodi strategiju i kojem se vraćaju rezultat ili greška |
| greška strategije | StrategyException | greška kojom strategija javlja svaki neuspjeh |
| Blackboard | Blackboard | zajednički međuspremnik koji izvodi strategiju stvaranja |
| repozitorij | Repository | mjesto trajne pohrane koje izvodi strategije čitanja i pisanja |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja strategije |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Kako se dokument stvara, čita i zapisuje, razlikuje se od slučaja do slučaja: tekst iz datoteke, tablica iz baze, poruka iz servisa. Kad bi svaki dio toka to znao sam, promjena izvora ili odredišta tražila bi izmjene na mnogo mjesta, a svaki bi dio javljao svoje kvarove na svoj način.

**Rješenje** Strategija je zamjenjiv način da se obavi jedan posao. Pozivatelj traži samo što mu treba (stvori dokument, pročitaj ga, zapiši ga), a kako se to radi, ostaje u strategiji. Zajednički okvir svim strategijama daje isti način pozivanja i isti način javljanja grešaka.

**Smjer** Postoje četiri namjene, od kojih svaka ima vlastiti naziv radnje: generiranje, stvaranje, čitanje i pisanje. Konkretnu strategiju izrađuje programer, a Workflow Factory je sastavlja prema konfiguraciji toka. Strategija ne zna tko je poziva ni zašto; zna samo svoj posao. Svaki neuspjeh javlja istom vrstom greške, s uzrokom.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja strategije i predaje ih onome tko ih izvodi: strategiju stvaranja Blackboardu, strategije čitanja i pisanja repozitoriju. Blackboard strategiju stvaranja izvodi kad procesor zatraži novi dokument. Repozitorij strategiju pisanja izvodi pri predaji dokumenata, a strategiju čitanja kad treba pročitati dokument. Strategije za rad s vanjskim resursom obično koriste driver. Rezultat ili greška vraćaju se pozivatelju.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Strategy: Data persistence"
caption "Context Diagram"

actor "Workflow Factory" as Factory
component "Blackboard" as Blackboard
component "Repository" as Repository
component "Driver" as Driver
database "External resource" as Resource

rectangle "Strategy" as Boundary <<system boundary>> {
  component Strategy
}

Factory ..> Strategy : assembles
Blackboard ..> Strategy : runs create
Repository ..> Strategy : runs read, write
Strategy ..> Driver : uses
Driver --> Resource : writes, reads
Strategy --> Blackboard : Facade / error
Strategy --> Repository : Facade, confirmation / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava strategije | izrađuje konkretne strategije za nove izvore i odredišta te održava postojeće |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja strategije | stvara strategije i predaje ih Blackboardu odnosno repozitoriju |
| Blackboard | izvodi strategiju stvaranja | pokreće je kad procesor zatraži novi dokument i prima fasadu ili grešku |
| Repozitorij | izvodi strategije čitanja i pisanja | pokreće ih pri čitanju i pri predaji dokumenata i prima rezultat ili grešku |
| Driver | pristup vanjskom resursu | strategije ga koriste za razgovor s resursom |

### 2.4 Korisnički slučajevi

| | UC1 — Sastavi strategiju | UC2 — Izvedi strategiju | UC3 — Razvij ili održavaj strategiju |
|---|---|---|---|
| **Akter** | Workflow Factory | Blackboard (za stvaranje) ili repozitorij (za čitanje i pisanje) | Programer |
| **Okidač** | Pokretanje toka čija konfiguracija navodi strategiju | Pozivatelju treba dokument stvoren, pročitan ili zapisan | Tok treba novi izvor ili odredište ili postojeća strategija treba prilagodbu |
| **Preduvjet** | Konfiguracija navodi vrstu strategije i njezine postavke | Strategija je sastavljena (UC1) i predana pozivatelju | — |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije čita strategiju. 2. Sastavlja je s postavkama toka. 3. Predaje je onome tko će je izvoditi. | 1. Pozivatelj traži radnju strategije i predaje joj sebe i ulaz (za čitanje oznaku dokumenta, za pisanje fasadu dokumenta). 2. Strategija obavlja svoj posao. 3. Strategija vraća rezultat: fasadu dokumenta, ništa ili, za pisanje, potvrdu da/ne. | 1. Programer izrađuje ili mijenja vrstu strategije i određuje njezin posao. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — | A1. Strategija pri radu ne uspije: pozivatelj prima grešku strategije s uzrokom i imenom radnje. A2. Strategija pisanja dokument ne zapiše: pozivatelj dobiva odgovor „ne" (nije greška). A3. Strategija stvaranja, čitanja ili generiranja ne vrati ništa: pozivatelj dobiva prazan rezultat. | — |
| **Ishod** | Strategija sastavljena i dostupna pozivatelju | Pozivatelj ima rezultat ili grešku | Vrsta strategije koju Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Strategy: Data persistence"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Blackboard
actor Repository
actor Programmer

rectangle "Strategy" as Boundary <<system boundary>> {
    usecase "Sastavi strategiju" as UC1
    usecase "Izvedi strategiju" as UC2
    usecase "Razvij ili održavaj\nstrategiju" as UC3
}

Factory -- UC1
Blackboard -- UC2
Repository -- UC2
Programmer -- UC3
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Strategija nema stanja: svaki poziv je zaseban. Dijagram prikazuje normalni tok (od poziva do rezultata) i alternativni tok u kojem se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Strategy: Data persistence"
caption "Flow Chart"

start
:caller runs the strategy\n(create, read, write or generate);
:strategy does its work;
if (work failed?) then (yes)
  if (already StrategyException?) then (yes)
    :error passed on unchanged;
  else (no)
    :<b><color:red>StrategyException</color></b>\n(operation, class and cause);
  endif
  :error returned to caller;
  stop
endif
if (write strategy?) then (yes)
  :confirmation yes / no returned;
else (no)
  :Facade or nothing returned;
endif
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `StrategyException` | strategija pri radu javi bilo kakvu grešku | rad se prekida; pozivatelj prima grešku koja nosi naziv radnje, vrstu strategije i izvorni uzrok |
| nepotvrđeno pisanje | strategija pisanja ne zapiše dokument | pozivatelj dobiva odgovor „ne" (nije greška) |
| prazan rezultat | strategija stvaranja, čitanja ili generiranja ne vrati dokument | pozivatelj dobiva prazan rezultat (nije greška) |

Greška koju je strategija već javila kao `StrategyException` prosljeđuje se nepromijenjena. Strategija ostavlja trag neuspjeha na razini detalja; grešku na razini greške bilježi onaj koji je zaustavi. Repozitorij grešku strategije prima i prosljeđuje u svojoj vrsti greške, s uzrokom.

### 2.6 Pretpostavke i ograničenja

- Strategija ne zna tko je izvodi ni zašto; pozivatelj se predaje kao ulaz, ali strategija ne ovisi o njemu za svoj posao.
- Strategija ne drži stanje između poziva.
- Što strategija konkretno radi, određuje programer; ovaj zapis propisuje samo okvir.
- Pisanje uvijek vraća odgovor „da" ili „ne": svaki odgovor koji nije izričito potvrdan znači „ne".
- Postoje četiri namjene strategije; nova namjena traži izmjenu okvira.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-STR-01** | Pozivatelj od strategije traži što mu treba, a ne kako se to radi. |
| **BR-STR-02** | Postoje četiri namjene strategije (generiranje, stvaranje, čitanje, pisanje), svaka s vlastitim nazivom radnje. |
| **BR-STR-03** | Konkretna vrsta strategije određuje samo sam posao; način pozivanja i javljanja grešaka isti je za sve. |
| **BR-STR-04** | Svaki neuspjeh strategije javlja se kao greška strategije, s nazivom radnje, vrstom strategije i izvornim uzrokom. |
| **BR-STR-05** | Greška koju je strategija već javila kao grešku strategije prosljeđuje se nepromijenjena. |
| **BR-STR-06** | Strategija pisanja uvijek vraća „da" ili „ne"; „ne" nije greška. |
| **BR-STR-07** | Strategije stvaranja, čitanja i generiranja vraćaju fasadu dokumenta ili ništa. |
| **BR-STR-08** | Strategiju stvaranja izvodi Blackboard na zahtjev procesora; strategije čitanja i pisanja izvodi repozitorij. |
| **BR-STR-09** | Strategija bilježi neuspjeh na razini detalja; grešku na razini greške bilježi onaj koji je zaustavi. |
| **BR-STR-10** | Strategija se podešava prema konfiguraciji toka. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; osim ondje gdje je poveznica navedena, još nema zapisa zahtjeva na kakvoću (§6).

| kakvoća | što znači za strategiju |
|---|---|
| zamjenjivost | promjena izvora ili odredišta mijenja strategiju, ne tok (BR-STR-01) |
| proširivost | programer može izraditi nove strategije za nove izvore i odredišta (UC3) |
| jednoobrazno javljanje grešaka | svaki neuspjeh ima isti oblik i uzrok (BR-STR-04) |
| razmjerno bilježenje | uzrok se bilježi jednom, na mjestu gdje se greška zaustavi ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md)) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-STR-*` i naziv sloja u naslovima dijagrama („Data persistence", kao u [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) i [`StRS-REPOSITORY`](StRS-REPOSITORY.md)) su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Strategija generiranja: u pregledanom kodu nitko je ne izvodi. Procesor zapise preuzima vlastitim generatorom, a ne ovom strategijom. Treba odrediti čemu služi i tko je izvodi, ili je ukloniti iz okvira.
4. Usklađenost s [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md): ondje se kaže da strategiju stvaranja izvodi Blackboard na zahtjev procesora; ovdje je Blackboard naveden kao njezin pozivatelj. Treba potvrditi da se opisi poklapaju.
5. Kakvoće iz §4 (osim bilježenja) treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard koji izvodi strategiju stvaranja |
| [`StRS-REPOSITORY`](StRS-REPOSITORY.md) | repozitorij koji izvodi strategije čitanja i pisanja |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | dokument koji strategije stvaraju, čitaju i zapisuju |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driver koji strategije koriste |
| [`SyRS-STR-strategy`](../02-SyRS/SyRS-STR-strategy.md) | zahtjevi sustava iste komponente |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | opseg bilježenja |
| `concrete/strategy.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/strategy.py`. |
