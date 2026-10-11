<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-MEMENTO — Snimka stanja: nepromjenjiv zapis stanja komponente

| | |
|---|---|
| **Verzija** | v0.0.4 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/memento.py`: `GenericMemento` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — bilježi i vraća snimku kao kontrolnu točku · [`StRS-WORKFLOW`](StRS-WORKFLOW.md) — bilježi i vraća snimku stanja svih procesora · [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) — stanje koje snimka nosi · [`StRS-FACTORY`](StRS-FACTORY.md) — uključuje kontrolne točke · [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) — čuva snimke između pokretanja |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine (D-13) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi snimka stanja, tko je koristi, što nosi i što se događa kad se od nje traži nešto čega nema.

### 1.2 Opseg

**U opsegu:** snimka kao nepromjenjiv zapis imenovanih vrijednosti, čitanje vrijednosti i glavnog stanja, provjera postoji li vrijednost, kopiranje u običan popis te greška koju snimka javlja.

**Izvan opsega:** što točno vlasnik stavlja u snimku i kako se iz nje vraća ([`StRS-PROCESSOR`](StRS-PROCESSOR.md), [`StRS-WORKFLOW`](StRS-WORKFLOW.md)) te gdje se snimka trajno čuva između pokretanja ([`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)). Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| snimka | Memento | nepromjenjiv zapis stanja komponente u jednom trenutku; nosi imenovane vrijednosti |
| vlasnik stanja | Originator | komponenta čije se stanje snima i koja se iz snimke može vratiti: procesor ili workflow |
| glavno stanje | state | vrijednost snimke pod imenom „state"; stanje u kojem se komponenta nalazila |
| cijeli popis | values | kopija svih imenovanih vrijednosti snimke, kakve su primljene |
| pohrana snimki | Memento store | dio koji čuva zadnju snimku po ključu da preživi prekid ([`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)) |
| kontrolna točka | checkpoint | snimka koju procesor zapisuje u pohranu da se prekinuta obrada može nastaviti ([`StRS-PROCESSOR`](StRS-PROCESSOR.md)) |
| stanje toka | workflow state | snimka u kojoj su stanja svih procesora jednog workflowa ([`StRS-WORKFLOW`](StRS-WORKFLOW.md)) |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Komponenta koja radi dugo može biti prekinuta, a njezino stanje je tada izgubljeno. Da bi se rad mogao nastaviti ili vratiti, stanje se mora u nekom trenutku zabilježiti u obliku koji se poslije ne može nehotice promijeniti.

**Rješenje** Snimka je nepromjenjiv zapis stanja. Vlasnik pri stvaranju upiše imenovane vrijednosti (npr. koji je ciklus obrade bio i u kojem stanju), a poslije se one samo čitaju. Snimka ne zna kamo se zapisuje ni kako se iz nje vraća: to ostaje na vlasniku i na pohrani snimki.

**Smjer** Snimka je općenita: ne propisuje koje vrijednosti nosi, samo da ih čuva i izdaje onakve kakve su primljene. Procesor i workflow isti oblik snimke koriste za različite namjene. Prikaz snimke nikad ne otkriva vrijednosti, jer one mogu biti osjetljive.

### 2.2 Kontekst

Vlasnik stanja (procesor ili workflow) stvara snimku s vrijednostima koje želi sačuvati. Pohrana snimki je čuva između pokretanja, a vlasnik je poslije čita i iz nje se vraća. Workflow u jednu snimku slaže snimke svih svojih procesora. Snimka sama nikoga ne obavještava i ništa ne pokreće.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Memento: Checkpointing"
caption "Context Diagram"

component "Originator\n(Processor, Workflow)" as Originator
component "Memento store" as Store

rectangle "Memento" as Boundary <<system boundary>> {
  component Memento
}

Originator ..> Memento : creates, reads
Store ..> Memento : keeps, recreates
Originator --> Store : saves
Store --> Originator : latest Memento
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vlasnike stanja | određuje koje vrijednosti vlasnik snima i kako se iz njih vraća |
| Vlasnik stanja | koristi snimku | stvara snimku sa svojim stanjem i iz nje se vraća |
| Pohrana snimki | čuva snimku | čuva zadnju snimku po ključu i iz zapisanih vrijednosti ponovno stvara snimku |

### 2.4 Korisnički slučajevi

| | UC1 — Zabilježi stanje | UC2 — Pročitaj snimku | UC3 — Razvij ili održavaj vlasnika stanja |
|---|---|---|---|
| **Akter** | Vlasnik stanja (primarni), Pohrana snimki (podupirući) | Vlasnik stanja ili pohrana snimki | Programer |
| **Okidač** | Vlasnik želi sačuvati svoje stanje | Vlasnik se vraća iz snimke, ili je pohrana treba zapisati | Komponenta treba moći sačuvati i vratiti stanje, ili njezino snimanje treba izmjenu |
| **Preduvjet** | — | Snimka postoji (UC1) | — |
| **Glavni tijek** | 1. Vlasnik predaje imenovane vrijednosti. 2. Snimka prepisuje popis vrijednosti i dalje ga ne mijenja. 3. Vlasnik, po potrebi, snimku predaje pohrani. | 1. Čitatelj traži vrijednost po imenu, glavno stanje ili cijeli popis vrijednosti, ili pita postoji li vrijednost. 2. Snimka ih izdaje. | 1. Programer određuje koje vrijednosti vlasnik snima i kako ih provjerava pri povratku. 2. Vlasnik postaje sposoban sačuvati i vratiti stanje. |
| **Alternativni tok** | — | A1. Tražena vrijednost ne postoji: greška. A2. Glavnog stanja nema: izdaje se prazan odgovor, bez greške. A3. Ime vrijednosti poklapa se sa svojstvom same snimke (npr. ime ili postavke bilježenja): čitanjem po imenu dobiva se svojstvo snimke, pa se takva vrijednost čita iz cijelog popisa (BR-MEM-09). | — |
| **Ishod** | Snimka s vrijednostima koje se više ne mijenjaju | Čitatelj ima tražene vrijednosti; snimka je nepromijenjena | Vlasnik koji može sačuvati i vratiti stanje |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Memento: Checkpointing"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor Originator
actor "Memento store" as Store
actor Programmer

rectangle "Memento" as Boundary <<system boundary>> {
    usecase "Zabilježi stanje" as UC1
    usecase "Pročitaj snimku" as UC2
    usecase "Razvij ili održavaj\nvlasnika stanja" as UC3
}

Originator -- UC1
UC1 -- Store
Originator -- UC2
Store -- UC2
Programmer -- UC3
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Snimka nema stanja: nakon stvaranja ostaje onakva kakva je. Dijagram prikazuje normalni tok (od stvaranja do čitanja) i alternativni tok u kojem se traži vrijednost koje nema.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Memento: Checkpointing"
caption "Flow Chart"

start
:Originator creates Memento\nwith named values;
:Memento keeps a read-only copy of the list;
:reader asks for a value;
if (value present?) then (no)
  if (main state asked?) then (yes)
    :empty answer returned;
  else (no)
    :<b><color:red>AttributeError</color></b>;
    :error returned to reader;
  endif
  stop
endif
:value returned unchanged;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `AttributeError` | traži se vrijednost kojoj ime snimka nema | greška se vraća čitatelju; snimka je nepromijenjena |
| prazan odgovor | traži se glavno stanje, a snimka ga nema | čitatelj dobiva prazan odgovor (nije greška) |

Snimka ne bilježi događaje. Prikaz snimke navodi samo imena vrijednosti.

### 2.6 Pretpostavke i ograničenja

- Snimka čuva popis vrijednosti, ne njihov sadržaj: vrijednosti su pohranjene kakve su primljene. Ako je vrijednost promjenjiva, vlasnik odgovara da se ona ne mijenja.
- Snimka ne zna kamo se trajno zapisuje; to radi pohrana snimki.
- Snimka ne provjerava što vlasnik u nju stavlja ni je li povratak moguć; to provjerava vlasnik pri povratku.
- Snimka nema vlastitog identiteta ni vremena stvaranja; ako ih vlasnik želi, stavlja ih među vrijednosti.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-MEM-01** | Snimka je nepromjenjiva: nakon stvaranja njezin se popis vrijednosti ne mijenja. |
| **BR-MEM-02** | Snimka nosi proizvoljan skup imenovanih vrijednosti i čuva ih kakve su primljene; ako je vrijednost promjenjiva, vlasnik odgovara za njezinu cjelovitost. |
| **BR-MEM-03** | Vrijednost pod imenom „state" je glavno stanje snimke; kad je nema, glavno stanje je prazno, a ne greška. |
| **BR-MEM-04** | Tražena vrijednost kojoj ime snimka nema je greška; snimka je ne zamjenjuje praznom vrijednošću. |
| **BR-MEM-05** | Prikaz snimke navodi samo imena vrijednosti, nikad same vrijednosti, jer mogu biti osjetljive. |
| **BR-MEM-06** | Nijedno ime vrijednosti nije rezervirano: i ime koje se inače koristi za podešavanje bilježenja ostaje u cijelom popisu snimke. |
| **BR-MEM-07** | Snimka ne zna kako se trajno čuva; to radi pohrana snimki. |
| **BR-MEM-08** | Vlasnik sam određuje što stavlja u snimku i provjerava je pri povratku. |
| **BR-MEM-09** | Vrijednost čije se ime poklapa sa svojstvom same snimke (npr. ime ili postavke bilježenja) pouzdano se čita samo iz cijelog popisa; čitanjem po imenu dobiva se svojstvo snimke. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; osim ondje gdje je poveznica navedena, još nema zapisa zahtjeva na kakvoću (§6).

| kakvoća | što znači za snimku |
|---|---|
| nepromjenjivost | snimka se ne može nehotice promijeniti nakon stvaranja (BR-MEM-01) |
| povjerljivost | prikaz snimke ne otkriva vrijednosti (BR-MEM-05; [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md)) |
| općenitost | isti oblik snimke služi procesoru i workflowu (BR-MEM-02) |
| jednostavnost | snimka ne zna za pohranu ni povrat, pa se lako koristi (BR-MEM-07) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-MEM-*` i naziv sloja u naslovima dijagrama („Checkpointing") su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Odnos snimke toka ([`StRS-WORKFLOW`](StRS-WORKFLOW.md), UC3 i UC4) i kontrolne točke procesora ([`StRS-PROCESSOR`](StRS-PROCESSOR.md), UC4) nije određen: oba koriste istu snimku, ali nije jasno čemu služi stanje toka, kad za nastavak koristi kontrolna točka. Ista stavka otvorena je u `StRS-WORKFLOW`.
4. **Imena koja se poklapaju sa svojstvima snimke.** Vrijednost s imenom kao što su ime ili postavke bilježenja čuva se, ali se ne može čitati po imenu (BR-MEM-09), a postavke bilježenja usput podešavaju i samu snimku. Treba odlučiti jesu li takva imena dopuštena ili ih vlasnici trebaju izbjegavati.
5. Kakvoće iz §4 (osim povjerljivosti) treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor, bilježi kontrolnu točku |
| [`StRS-WORKFLOW`](StRS-WORKFLOW.md) | workflow, bilježi stanje toka |
| [`StRS-STATE-MACHINE`](StRS-STATE-MACHINE.md) | stroj stanja čije stanje snimka nosi |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji uključuje kontrolne točke |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | pohrana snimki |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | povjerljivost zapisa |
| `concrete/memento.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.4 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.3 | 2026-10-11 | Pohrana snimki opisana je u StRS-MEMENTO-STORE: poveznice u opsegu, pojmovima i referencama; stavka o pohrani uklonjena iz *Otvoreno*. |
| v0.0.2 | 2026-10-11 | Prema kodu: vrijednost čije se ime poklapa sa svojstvom snimke čita se samo iz cijelog popisa (novo pravilo BR-MEM-09, UC2 A3, BR-MEM-06 preciziran); dodana provjera postoji li vrijednost (UC2, opseg); pojam *cijeli popis*. Poveznica na StRS-FACTORY. *Otvoreno*: pitanje o imenima koja se poklapaju sa svojstvima snimke. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/memento.py`. |