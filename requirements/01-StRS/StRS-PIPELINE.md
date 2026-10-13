<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-PIPELINE — Generički pipeline: jedna transformacija nad jednim dokumentom

| | |
|---|---|
| **Verzija** | v0.0.4 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/pipeline.py`: `GenericPipeline`, `PipelineError` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — procesor pokreće pipeline nad svakim dokumentom · [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — pipeline čita i piše dokumente na Blackboardu · [`StRS-DOCUMENT`](StRS-DOCUMENT.md) — dokument koji pipeline prima kroz fasadu · [`StRS-FACTORY`](StRS-FACTORY.md) — sastavlja pipeline |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi generički pipeline, tko ga koristi, kako obavlja jednu transformaciju nad dokumentom i što se događa kad transformacija ne uspije.

### 1.2 Opseg

**U opsegu:** pipeline kao najmanja jedinica transformacije, provjera ulaza, pokretanje transformacije, prijavljivanje grešaka i bilježenje događaja.

**Izvan opsega:** koliko dokumenata ima, odakle dolaze i kojim ih redom procesor daje (opisano u [`StRS-PROCESSOR`](StRS-PROCESSOR.md)), kako Blackboard dokumente drži i predaje ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md)), što dokument sadrži ([`StRS-DOCUMENT`](StRS-DOCUMENT.md)) te što točno pojedina transformacija radi. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| pipeline | Pipeline | najmanja jedinica transformacije: jedan korak koji se izvodi nad jednim dokumentom |
| transformacija | transformation | posao koji pipeline obavlja nad dokumentom; određuje je konkretna vrsta pipelinea |
| dokument | document | strukturirana ili nestrukturirana jedinica podataka koja se obrađuje |
| fasada | Facade | omotač kroz koji pipeline vidi dokument ([`StRS-DOCUMENT`](StRS-DOCUMENT.md)) |
| procesor | Processor | motor koji dokumente jedan po jedan provodi kroz pipelineove |
| Blackboard | Blackboard | zajednički međuspremnik na kojem pipelineovi ostavljaju i čitaju dokumente |
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije sastavlja pipeline i registrira ga kod procesora |
| greška pipelinea | PipelineError | greška kojom pipeline javlja svaki neuspjeh |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Obrada podataka sastoji se od niza koraka, a svaki bi korak mogao provjeravati ulaz, prijavljivati greške i bilježiti događaje na svoj način. Takvi koraci ne bi bili zamjenjivi, a kvar u jednom ne bi bilo moguće prepoznati kao kvar obrade.

**Rješenje** Pipeline je najmanja jedinica transformacije: jedan dokument ulazi, jedna transformacija se izvodi. Zajednički dio pipelinea provjerava ulaz, bilježi početak i kraj te svaki neuspjeh javlja istom vrstom greške, s uzrokom. Samu transformaciju dodaje konkretna vrsta pipelinea.

**Smjer** Pipeline ne zna koliko dokumenata ima, odakle dolaze ni kada se rezultat trajno sprema: to je posao procesora i Blackboarda. Procesor ga pokreće nad svakim dokumentom, a pipeline radi samo svoj korak. Podešava se prema konfiguraciji toka. Bilježi se na razini detalja, jer je jedinica rada koja se bilježi cijela obrada dokumenta, a ne pojedini korak.

### 2.2 Kontekst

Workflow Factory iz konfiguracije sastavlja pipeline i registrira ga kod procesora. Procesor pri obradi svakog dokumenta pokreće svaki svoj pipeline, predajući mu sebe i fasadu dokumenta. Pipeline provjerava što je dobio i izvodi transformaciju, koja po potrebi čita i piše dokumente na Blackboardu ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), UC4). Pipeline procesoru javlja samo je li uspio ili javlja grešku.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Pipeline: Data processing"
caption "Context Diagram"

actor "Workflow Factory" as Factory
component "Processor" as Processor
component "Blackboard" as Blackboard

rectangle "Pipeline" as Boundary <<system boundary>> {
  component Pipeline
}

Factory ..> Pipeline : assembles
Factory ..> Processor : registers Pipeline
Processor ..> Pipeline : runs on each document
Pipeline ..> Blackboard : reads, writes
Pipeline --> Processor : success / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste pipelinea | izrađuje nove transformacije i održava postojeće |
| Workflow Factory | prema zadanoj konfiguraciji sastavlja pipeline | sastavlja pipeline i registrira ga kod procesora |
| Procesor | pokreće pipeline | predaje pipelineu sebe i fasadu dokumenta te prima obavijest o uspjehu ili grešku |
| Blackboard | mjesto dokumenata | na njemu transformacija čita i piše dokumente |

### 2.4 Korisnički slučajevi

| | UC1 — Sastavi pipeline | UC2 — Transformiraj dokument | UC3 — Razvij ili održavaj pipeline |
|---|---|---|---|
| **Akter** | Workflow Factory | Procesor (primarni), Blackboard (podupirući) | Programer |
| **Okidač** | Pokretanje toka čija konfiguracija navodi pipeline | Procesor je preuzeo sljedeći dokument | Tok treba novu transformaciju ili postojeća treba prilagodbu |
| **Preduvjet** | Konfiguracija navodi vrstu pipelinea i njegove postavke | Pipeline je sastavljen (UC1) i registriran kod procesora | — |
| **Glavni tijek** | 1. Workflow Factory iz konfiguracije čita pipeline. 2. Sastavlja ga s navedenim postavkama. 3. Registrira ga kod procesora. | 1. Procesor predaje pipelineu sebe i fasadu dokumenta. 2. Pipeline provjerava da je dobio procesor i fasadu. 3. Pipeline izvodi svoju transformaciju; ona po potrebi čita i piše dokumente na Blackboardu. 4. Pipeline javlja procesoru da je uspio. | 1. Programer izrađuje ili mijenja vrstu pipelinea i određuje njegovu transformaciju. 2. Nova ili izmijenjena vrsta postaje dostupna konfiguraciji. |
| **Alternativni tok** | — | A1. Procesor ili fasada nisu ispravne vrste: transformacija se ne izvodi, greška pipelinea. A2. Transformacija ne uspije: greška pipelinea s uzrokom. U oba slučaja procesor prekida cijelu obradu ([`BR-PRC-04`](StRS-PROCESSOR.md#3-poslovna-pravila)). | — |
| **Ishod** | Pipeline sastavljen i registriran kod procesora | Dokument transformiran; procesor nastavlja | Vrsta pipelinea koju Workflow Factory može zadati konfiguracijom |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Pipeline: Data processing"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Workflow Factory" as Factory
actor Processor
actor Blackboard
actor Programmer

rectangle "Pipeline" as Boundary <<system boundary>> {
    usecase "Sastavi pipeline" as UC1
    usecase "Transformiraj dokument" as UC2
    usecase "Razvij ili održavaj\npipeline" as UC3
}

Factory -- UC1
Processor -- UC2
UC2 -- Blackboard
Programmer -- UC3
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Pipeline nema stanja: svaki poziv je zaseban i ne ovisi o prethodnom. Dijagram prikazuje normalni tok (od poziva procesora do završetka transformacije) i alternativne tokove u kojima se rad prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Pipeline: Data processing"
caption "Flow Chart"

start
:Processor passes itself and\nthe Facade of the document;
if (Processor and Facade valid?) then (no)
  :<b><color:red>PipelineError</color></b>;
  :error returned to Processor;
  stop
endif
:Pipeline runs its transformation;
if (transformation failed?) then (yes)
  :<b><color:red>PipelineError</color></b>\n(with the cause);
  :error returned to Processor;
  stop
endif
:success reported to Processor;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `PipelineError` | pipeline nije dobio procesor i fasadu dokumenta; ili je transformacija javila bilo kakvu grešku | transformacija se prekida; procesor prima grešku koja nosi uzrok i prekida cijelu obradu |

Svaka greška nosi izvorni uzrok i ne omata se dvaput. Pipeline za grešku ostavlja samo trag za detaljnu analizu; grešku bilježi procesor, koji zaustavlja obradu.

### 2.6 Pretpostavke i ograničenja

- Pipeline obrađuje jedan dokument po pozivu; broj dokumenata i redoslijed određuje procesor.
- Pipeline procesoru javlja samo je li uspio; rezultat transformacije ostaje na dokumentu ili Blackboardu.
- Pipeline ne zna odakle dokument dolazi ni kamo se pohranjuje.
- Što transformacija radi, određuje konkretna vrsta pipelinea; ovaj zapis to ne propisuje.
- Pipeline nema stanja ni pauze: kvar jedne transformacije ne ostavlja ništa što bi sljedeći poziv morao popraviti.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-PPL-01** | Pipeline obavlja jednu transformaciju nad jednim dokumentom po pozivu. |
| **BR-PPL-02** | Pipeline prije transformacije provjerava da je dobio procesor i fasadu; inače se transformacija ne izvodi. |
| **BR-PPL-03** | Svaki neuspjeh pipelinea javlja se kao greška pipelinea; izvorna greška ostaje kao uzrok. |
| **BR-PPL-04** | Greška koju je transformacija već javila kao greška pipelinea prosljeđuje se nepromijenjena. |
| **BR-PPL-05** | Pipeline ne zna koliko dokumenata ima, odakle dolaze ni kamo se pohranjuju. |
| **BR-PPL-06** | Pipeline ne stvara zaseban zapis u dnevniku za svaki korak; dnevnik bilježi obradu cijelog dokumenta. |
| **BR-PPL-07** | Pipeline se podešava prema konfiguraciji toka. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; osim ondje gdje je poveznica navedena, još nema zapisa zahtjeva na kakvoću (§6).

| kakvoća | što znači za pipeline |
|---|---|
| razmjerno bilježenje | dnevnik ne raste s brojem koraka: po jedan zapis po dokumentu daje procesor ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md)) |
| jednoobrazno javljanje grešaka | svaki neuspjeh pipelinea ima isti oblik i uzrok (BR-PPL-03) |
| zamjenjivost | koraci su zamjenjivi jer dijele isti način pozivanja i javljanja grešaka |
| proširivost | programer može izraditi nove vrste pipelinea (UC3) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-PIP-*` i naziv sloja u naslovima dijagrama („Data processing", kao u [`StRS-PROCESSOR`](StRS-PROCESSOR.md)) su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. Registraciju pipelinea kod procesora provodi Workflow Factory (UC1, korak 3; potvrđeno u [`BR-FCT-10`](StRS-FACTORY.md#3-poslovna-pravila)). Treba odlučiti je li to zaseban korisnički slučaj; isto pitanje otvoreno je u [`StRS-PROCESSOR`](StRS-PROCESSOR.md).
4. Kakvoće iz §4 (osim bilježenja) treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor koji pokreće pipeline |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard na kojem transformacija čita i piše |
| [`StRS-DOCUMENT`](StRS-DOCUMENT.md) | dokument koji pipeline prima |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | opseg bilježenja |
| [`StRS-FACTORY`](StRS-FACTORY.md) | Workflow Factory, koji sastavlja pipeline |
| `concrete/pipeline.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.4 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.3 | 2026-10-11 | Registracija pipelinea kod procesora potvrđena kodom Workflow Factoryja (BR-FCT-10). Poveznica na StRS-FACTORY. |
| v0.0.2 | 2026-10-11 | Akter koji sastavlja i registrira pipeline je Workflow Factory. Usklađeno s povezanim StRS-ovima: rad s Blackboardom potvrđen StRS-BLACKBOARD UC4; posljedica greške povezana s BR-PRC-04; uklonjena netočna tvrdnja o Blackboardu i napomena o nazivu *zapis*. Odlomak o vrstama iznimki i razinama bilježenja zamijenjen opisom za široku publiku; BR-PPL-06 i BR-PPL-07 pojednostavljeni; BR-PPL-08 (oslobađanje postavki pri gašenju) prenesen u SyRS. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/pipeline.py`. |