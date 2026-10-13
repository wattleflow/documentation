<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-MEMENTO-STORE — Pohrana snimki: čuvanje stanja između pokretanja

| | |
|---|---|
| **Verzija** | v0.0.3 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/memento_store.py`: `MementoStore`, `MemoryMementoStore`, `FileMementoStore`, `MementoStoreException` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-MEMENTO`](StRS-MEMENTO.md) — snimka koju pohrana čuva · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — zapisuje, čita i briše kontrolnu točku · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja pohranu iz konfiguracije |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi pohrana snimki, tko je koristi, koje vrste pohrane postoje i što se događa kad zapis, čitanje ili brisanje ne uspije.

### 1.2 Opseg

**U opsegu:** čuvanje zadnje snimke po ključu, čitanje i brisanje snimke, memorijska i datotečna pohrana, zahtjevi na ključ i snimku te greške koje pohrana javlja.

**Izvan opsega:** što snimka sadrži i kako se vlasnik iz nje vraća ([`StRS-MEMENTO`](StRS-MEMENTO.md), [`StRS-PROCESSOR`](StRS-PROCESSOR.md)), kada procesor zapisuje kontrolnu točku te kako se pohrana zadaje u konfiguraciji ([`StRS-FACTORY`](StRS-FACTORY.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| pohrana snimki | Memento store | dio koji čuva zadnju snimku po ključu da preživi prekid rada |
| snimka | Memento | nepromjenjiv zapis stanja komponente ([`StRS-MEMENTO`](StRS-MEMENTO.md)) |
| ključ | key | običan naziv pod kojim se snimka čuva; za procesor je to njegovo ime |
| memorijska pohrana | Memory store | pohrana koja čuva snimke dok proces traje |
| datotečna pohrana | File store | pohrana koja čuva snimke u mapi, jednu datoteku po ključu |
| kontrolna točka | checkpoint | snimka koju procesor zapisuje da se prekinuta obrada može nastaviti ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| vlasnik stanja | Originator | komponenta čije se stanje čuva; za pohranu je to procesor |
| Workflow Factory | Workflow Factory | dio koji iz konfiguracije stvara pohranu i predaje je procesorima ([`StRS-FACTORY`](StRS-FACTORY.md)) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Obrada koja je prekinuta ne smije započeti ispočetka ni preskočiti posao. Da bi se mogla nastaviti, njezin napredak mora biti negdje zabilježen izvan same obrade, u obliku koji preživi prekid.

**Rješenje** Pohrana snimki čuva zadnju snimku po ključu. Procesor u nju zapisuje svoju kontrolnu točku, iz nje ga čita pri ponovnom pokretanju i briše je kad obrada uspješno završi. Pohrana ne tumači sadržaj snimke: samo je čuva i vraća.

**Smjer** Postoje dvije vrste pohrane: memorijska, koja čuva snimke dok proces traje, i datotečna, koja ih čuva u mapi pa preživi i ponovno pokretanje procesa. Vrsta se zadaje u konfiguraciji toka, a moguće je dodati i vlastitu. Pohrana ne odlučuje kada se zapisuje: to određuje procesor.

### 2.2 Kontekst

Workflow Factory iz konfiguracije stvara pohranu i predaje je procesorima, uz ključ i razmak između kontrolnih točaka. Procesor tijekom obrade zapisuje snimku, pri ponovnom pokretanju je čita, a po uspješnom završetku briše. Datotečna pohrana snimke zapisuje u mapu na disku. Greške se vraćaju onome tko je radnju zatražio.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Memento store: Checkpointing"
caption "Context Diagram"

actor "Workflow Factory" as Factory
actor "Programmer" as Programmer
component "Processor" as Processor
database "Storage folder\n(file store only)" as Folder

rectangle "Memento store" as Boundary <<system boundary>> {
  component "Memento store" as Store
}

Programmer ..> Store : adds kinds
Factory ..> Store : creates
Factory ..> Processor : hands over Store
Processor ..> Store : writes, reads, clears
Store --> Processor : Memento / error
Store --> Folder : one file per key
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste pohrane | dodaje nove vrste pohrane koje konfiguracija može imenovati |
| Workflow Factory | sastavlja pohranu | stvara pohranu iz konfiguracije i predaje je procesorima |
| Procesor | koristi pohranu | zapisuje kontrolnu točku, čita je pri ponovnom pokretanju i briše je po završetku |
| Mapa na disku | mjesto zapisa | prima datoteke datotečne pohrane |

### 2.4 Korisnički slučajevi

#### UC1 — Sastavi pohranu

| | |
|---|---|
| **Akter** | Workflow Factory |
| **Okidač** | Konfiguracija toka zadaje kontrolne točke |
| **Preduvjet** | Konfiguracija navodi vrstu pohrane (memorijsku, datotečnu ili registriranu) |
| **Glavni tijek** | 1. Workflow Factory stvara pohranu zadane vrste s njezinim postavkama. 2. Workflow Factory je predaje procesorima, svakom s njegovim ključem. |
| **Alternativni tok** | A1. Datotečna pohrana nema zadanu mapu: pohrana se ne stvara, greška pohrane. A2. Zadana vrsta nije pohrana snimki ili se ne može stvoriti, ili ime procesora nije dopušten ključ: sastavljanje se prekida, greška Workflow Factoryja ([`StRS-FACTORY`](StRS-FACTORY.md), UC3). |
| **Ishod** | Pohrana spremna za čuvanje snimki |

#### UC2 — Zapiši snimku

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Procesor je završio dovoljno ciklusa da zabilježi napredak |
| **Preduvjet** | Pohrana je sastavljena (UC1) |
| **Glavni tijek** | 1. Procesor predaje pohrani ključ i snimku. 2. Pohrana provjerava ključ i snimku. 3. Pohrana čuva snimku kao zadnju za taj ključ i zamjenjuje prethodnu. |
| **Alternativni tok** | A1. Ključ nije običan naziv ili predano nije snimka: zapis se odbija, greška pohrane. A2. Datotečna pohrana ne može zapisati vrijednost snimke ili zapis na disk ne uspije: zapis se odbija, prethodna snimka ostaje, greška pohrane. |
| **Ishod** | Zadnja snimka za ključ je nova; ili je prethodna ostala uz grešku |

Ako zapis ne uspije, procesor nastavlja obradu i bilježi upozorenje ([`StRS-PROCESSOR`](StRS-PROCESSOR.md), `BR-PRC-09`).

#### UC3 — Pročitaj snimku

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Procesor se ponovno pokreće i traži mogući nastavak |
| **Preduvjet** | Pohrana je sastavljena (UC1) |
| **Glavni tijek** | 1. Procesor traži snimku po ključu. 2. Pohrana vraća snimku sastavljenu od zapisanih vrijednosti. |
| **Alternativni tok** | A1. Za ključ nema snimke: pohrana ne vraća ništa, bez greške. A2. Zapis se ne može pročitati ili je oštećen: greška pohrane. |
| **Ishod** | Procesor ima snimku iz koje može nastaviti, ili zna da je nema |

Vrijednosti se vraćaju kao obični podaci: nabrojane vrijednosti (npr. stanje) kao imena, a nizovi kao popisi. Vlasnik ih sam pretvara natrag ([`StRS-MEMENTO`](StRS-MEMENTO.md)).

#### UC4 — Obriši snimku

| | |
|---|---|
| **Akter** | Procesor |
| **Okidač** | Obrada je uspješno završila |
| **Preduvjet** | Pohrana je sastavljena (UC1) |
| **Glavni tijek** | 1. Procesor traži brisanje snimke po ključu. 2. Pohrana je briše. |
| **Alternativni tok** | A1. Za ključ nema snimke: ništa se ne događa, bez greške. A2. Brisanje datoteke ne uspije: greška pohrane. |
| **Ishod** | Za ključ nema snimke, pa sljedeće pokretanje ne preskače posao |

Ako brisanje ne uspije, procesor bilježi upozorenje i obrada ostaje završena.

#### UC5 — Razvij ili održavaj vrstu pohrane

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Treba nova vrsta pohrane (npr. baza) ili postojeća treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje vrstu pohrane koja zna zapisati, pročitati i obrisati snimku po ključu. 2. Vrsta se upisuje u registar Workflow Factoryja. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta pohrane koju konfiguracija može imenovati |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Memento store: Checkpointing"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Processor
actor Programmer

rectangle "Memento store" as Boundary <<system boundary>> {
    usecase "Sastavi pohranu" as UC1
    usecase "Zapiši snimku" as UC2
    usecase "Pročitaj snimku" as UC3
    usecase "Obriši snimku" as UC4
    usecase "Razvij ili održavaj\nvrstu pohrane" as UC5
}

Factory -- UC1
Processor -- UC2
Processor -- UC3
Processor -- UC4
Programmer -- UC5
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Pohrana nema stanja: svaka radnja je zasebna. Dijagram prikazuje normalni tok (zapis, čitanje i brisanje) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Memento store: Checkpointing"
caption "Flow Chart"

start
:Processor writes Memento under a key;
if (key a plain name and\nvalue a Memento?) then (no)
  :<b><color:red>MementoStoreException</color></b>;
  :error returned to Processor;
  stop
endif
if (value can be stored?) then (no)
  :<b><color:red>MementoStoreException</color></b>\n(previous Memento kept);
  :error returned to Processor;
  stop
endif
:Memento kept as the latest for the key;
:Processor restarts and reads the key;
if (Memento exists?) then (no)
  :nothing returned;
else (yes)
  if (record readable and valid?) then (no)
    :<b><color:red>MementoStoreException</color></b>;
    :error returned to Processor;
    stop
  endif
  :Memento returned;
endif
:run completes, Processor clears the key;
if (clear failed?) then (yes)
  :<b><color:red>MementoStoreException</color></b>;
  :error returned to Processor;
  stop
endif
:no Memento for the key;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `MementoStoreException` | ključ nije običan naziv; predano nije snimka; datotečna pohrana nema mapu; vrijednost se ne može zapisati; zapis, čitanje ili brisanje na disku ne uspije; zapis je oštećen; zatražena je nepoznata radnja | radnja se prekida; prethodna snimka ostaje netaknuta; grešku prima onaj tko je radnju zatražio: pri sastavljanju Workflow Factory, inače procesor, koji je obrađuje prema svojim pravilima |
| prazan odgovor | čita se ključ koji nema snimke | pohrana ne vraća ništa (nije greška) |

Datotečna pohrana pri uspješnom zapisu i čitanju ostavlja samo trag za detaljnu analizu. Procesor, a ne pohrana, odlučuje je li neuspjeh zapisa ili brisanja upozorenje, a neuspjeh čitanja greška obrade.

### 2.6 Pretpostavke i ograničenja

- Pohrana čuva samo zadnju snimku po ključu; prošle snimke se ne pamte.
- Pohrana ne tumači sadržaj snimke i ne zna što on znači.
- Snimke u memorijskoj pohrani prežive neuspjelu obradu, ali ne i prekid samog procesa; snimke u datotečnoj pohrani prežive i to.
- Za nastavak nakon ponovnog pokretanja potrebna je datotečna pohrana s istom mapom.
- Ključ postaje naziv datoteke, pa je običan naziv, nikad putanja.
- Pohrana ne odlučuje kad se zapisuje, čita ili briše: to određuje procesor.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-MST-01** | Pohrana čuva zadnju snimku po ključu: nova snimka istog ključa zamjenjuje prethodnu. |
| **BR-MST-02** | Ključ mora biti običan naziv: slova engleske abecede (bez dijakritike), znamenke, točka, crtica i podvlaka; počinje slovom ili znamenkom i ne završava točkom. Inače je greška. |
| **BR-MST-03** | Pohrana prima samo snimke stanja; sve drugo je greška. |
| **BR-MST-04** | Pohrana ne tumači sadržaj snimke: vraćanje u izvorni oblik posao je vlasnika stanja. |
| **BR-MST-05** | Vrijednosti se čuvaju kao obični podaci, jednako u svakoj vrsti pohrane: nabrojana vrijednost (npr. stanje) čuva se imenom i čita kao ime, a niz se čita kao popis. |
| **BR-MST-06** | Čitanje ključa bez snimke vraća ništa, a brisanje takvog ključa nije greška. |
| **BR-MST-07** | Memorijska pohrana čuva snimke dok proces traje; datotečna ih čuva u mapi, jednu datoteku po ključu. |
| **BR-MST-08** | Datotečna pohrana traži mapu; ista mapa nakon ponovnog pokretanja omogućuje nastavak. |
| **BR-MST-09** | Datotečna pohrana zamjenjuje snimku u jednom nedjeljivom koraku: ako zapis ne uspije, prethodna snimka ostaje. |
| **BR-MST-10** | Datotečna pohrana odbija vrijednost koju ne može zapisati; prethodna snimka ostaje. |
| **BR-MST-11** | Oštećen zapis (nečitljiv ili ne sadrži popis vrijednosti) javlja se kao greška, a ne kao prazna snimka. |
| **BR-MST-12** | Svaki neuspjeh pohrane javlja se kao greška pohrane. |
| **BR-MST-13** | Pohrana se može zadati i kao vlastita vrsta upisana u registar Workflow Factoryja. |
| **BR-MST-14** | Pohrana se može pozvati i općim zahtjevom s radnjom (zapiši, pročitaj, obriši); nepoznata radnja je greška. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; za njih još nema poveznice na zapise zahtjeva na kakvoću (§6).

| kakvoća | što znači za pohranu |
|---|---|
| otpornost | prekinuta obrada može se nastaviti iz zadnje snimke; neuspjeh zapisa ne uništava prethodnu snimku (BR-MST-09) |
| cjelovitost | zapis datoteke je nedjeljiv; oštećen zapis se ne predstavlja kao ispravan (BR-MST-09, BR-MST-11) |
| sigurnost u paralelnom radu | memorijska pohrana štiti svoj sadržaj pri istodobnom pristupu |
| zamjenjivost | memorijska, datotečna i vlastita pohrana koriste se na isti način i vraćaju iste vrijednosti (BR-MST-05, BR-MST-13) |
| sigurnost putanje | ključ je običan naziv, pa ne može izaći iz mape pohrane (BR-MST-02) |
| proširivost | programer može dodati vlastitu vrstu pohrane (UC5) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-MST-*` i naziv sloja u naslovima dijagrama („Checkpointing", kao u [`StRS-MEMENTO`](StRS-MEMENTO.md)) su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. **Ime procesora kao ključ.** Ključ kontrolne točke je ime procesora, a ključ smije sadržavati samo slova bez dijakritike (BR-MST-02). Procesor s imenom koje sadrži npr. „č" ili razmak ne može imati kontrolne točke, a tok se zbog toga ne sastavlja. Treba odlučiti je li to ograničenje prihvatljivo ili ključ treba izvoditi iz imena.
4. **Memorijska pohrana.** Izvan procesa ne pomaže pri nastavku nakon prekida samog procesa. Treba potvrditi kada je ona željena za produkciju, a kada samo za ispitivanje.
5. **Vlasnik stanja.** Samo procesor je potvrđen kao korisnik pohrane; workflow bilježi stanje toka, ali ga u pohranu ne zapisuje ([`StRS-WORKFLOW`](StRS-WORKFLOW.md), stavka 5).
6. **Povrat vrijednosti.** Pohrana nabrojane vrijednosti vraća kao imena, a nizove kao popise. Treba potvrditi da pretvorba u izvorni oblik uvijek ostaje na vlasniku.
7. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-MEMENTO`](StRS-MEMENTO.md) | snimka koju pohrana čuva |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor, korisnik pohrane |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja pohranu |
| [`StRS-WORKFLOW`](StRS-WORKFLOW.md) | workflow, snimka stanja toka |
| `concrete/memento_store.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.3 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.2 | 2026-10-11 | Nazivlje: *Workflow Factory* umjesto *tvornica workflowa*. Prema kodu: ključ dopušta samo slova bez dijakritike (BR-MST-02); nizovi se vraćaju kao popisi u svakoj vrsti pohrane (BR-MST-05, UC3); greške pri sastavljanju pohrane prima Workflow Factory (UC1 A2, tablica grešaka). Odlomak o bilježenju napisan jednostavnim jezikom. *Otvoreno*: oznaka zapisa više nije prijedlog; stavka o ujednačenosti riješena usklađivanjem StRS-MEMENTO, StRS-PROCESSOR i StRS-FACTORY; dodano pitanje o imenu procesora kao ključu. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/memento_store.py`. |