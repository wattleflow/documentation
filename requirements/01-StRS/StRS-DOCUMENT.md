<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-DOCUMENT — Dokument: jedinica podataka koja putuje tokom

| | |
|---|---|
| **Verzija** | v0.0.3 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/document.py`: `Document`, `DocumentAdapter`, `DocumentFacade` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) — drži dokumente na platnu · [`StRS-PROCESSOR`](StRS-PROCESSOR.md) — provodi dokumente kroz pipelineove · [`StRS-PIPELINE`](StRS-PIPELINE.md) — prima dokument kroz fasadu |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi dokument, tko ga koristi, što o sebi nosi, kako se mijenja i što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** dokument kao nositelj sadržaja, identiteta i zapisa o zadnjoj izmjeni; fasada kroz koju tok drži dokument; promjena sadržaja i metapodataka; čišćenje dokumenta i greške koje dokument javlja.

**Izvan opsega:** kako strategija stvaranja gradi dokument i koje vrste dokumenata postoje, kako Blackboard dokumente drži i predaje ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md)), što pipelineovi rade s dokumentima ([`StRS-PIPELINE`](StRS-PIPELINE.md)) te kako ih repozitoriji trajno pohranjuju. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| dokument | Document | strukturirana ili nestrukturirana jedinica podataka (npr. tablica, tekst ili niz bajtova) koja putuje tokom; pojam znači isto u cijelom sloju postojanosti podataka (*data persistence*): na Blackboardu, u pipelineovima i u repozitorijima |
| sadržaj | content | sami podaci koje dokument nosi; i prazan tekst ili prazna tablica jesu sadržaj |
| metapodaci | metadata | podaci koji opisuju sadržaj: izvor, ime datoteke, shema, trag obrade i sl. |
| identitet | identifier | jedinstvena oznaka dokumenta, određena pri stvaranju i nepromjenjiva |
| zapis o zadnjoj izmjeni | last change | što je zadnje promijenjeno u dokumentu i kada; vodi ga sam dokument |
| čišćenje | clean | brisanje sadržaja i metapodataka dokumenta |
| fasada | Facade | omotač kroz koji tok drži dokument; propušta pozivatelju svojstva dokumenta, a sam dokument ostaje iza |
| adapter | Adapter | posrednik ugrađen u fasadu, koji joj daje dokument iza nje |
| strategija stvaranja | Create strategy | strategija koja stvara novi dokument i fasadu za njega |
| strategija pisanja | Write strategy | strategija koja čita dokument kroz fasadu i zapisuje ga u repozitorij |
| Blackboard | Blackboard | zajednički međuspremnik u kojem dokumenti žive dok traje obrada |
| pipeline | Pipeline | najmanja jedinica transformacije; čita i mijenja dokument |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Dijelovi toka međusobno predaju podatke, a svaki bi ih mogao nositi na svoj način: bez jedinstvene oznake, bez zapisa tko ih je i kada mijenjao, s vrstom podataka koja se usput mijenja. Takav tok je nepregledan, a pogreškama u podacima teško se ulazi u trag.

**Rješenje** Dokument je jedinica podataka koja putuje tokom. Nosi sadržaj, jedinstveni identitet i povijest izmjena, a promjene prima samo kroz propisane radnje. Tok ne drži sam dokument nego fasadu, kroz koju vidi svojstva dokumenta bez obzira na to koje je vrste.

**Smjer** Dokument je općenit: ne zna koja je vrsta podataka u njemu niti kamo će otići. Vrsta sadržaja zadaje se prvim sadržajem i dalje se ne mijenja. Podaci žive u sadržaju, a metapodaci ih samo opisuju. Povijest izmjena zaštićena je od krivotvorenja: zapis o zadnjoj izmjeni piše samo dokument. Konkretne vrste dokumenata (tekst, tablica, binarni niz) izrađuju programeri.

### 2.2 Kontekst

Strategija stvaranja stvara dokument i fasadu za njega. Dokument zatim živi na Blackboardu, a pipelineovi ga kroz fasadu čitaju i mijenjaju. Strategija pisanja kroz fasadu čita dokument kad ga treba zapisati u repozitorij. Kad se platno Blackboarda čisti, čiste se i dokumenti. Grešku dobiva onaj tko je radnju zatražio.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Document: Data persistence"
caption "Context Diagram"

component "Create strategy" as Create
component "Pipeline" as Pipeline
component "Write strategy" as Write
component "Blackboard" as Blackboard

rectangle "Document" as Boundary <<system boundary>> {
  component "Document\n(with Facade)" as Document
}

Create ..> Document : creates
Pipeline ..> Document : reads, changes
Write ..> Document : reads
Blackboard ..> Document : holds, cleans
Document --> Pipeline : content, metadata
Document --> Write : content, metadata
Document --> Create : outcome / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste dokumenata | izrađuje nove vrste dokumenata i određuje kako se mjeri njihova veličina |
| Strategija stvaranja | stvara dokumente | stvara dokument s početnim sadržajem i fasadu za njega |
| Pipeline | transformira dokumente | čita i mijenja sadržaj i metapodatke kroz fasadu |
| Strategija pisanja | zapisuje dokumente | kroz fasadu čita sadržaj i metapodatke radi zapisa u repozitorij |
| Blackboard | drži dokumente | drži dokumente na platnu; pri čišćenju platna čiste se i dokumenti |

### 2.4 Korisnički slučajevi

#### UC1 — Stvori dokument

| | |
|---|---|
| **Akter** | Strategija stvaranja |
| **Okidač** | Obrada treba novi dokument |
| **Preduvjet** | Poznat je početni sadržaj |
| **Glavni tijek** | 1. Strategija stvaranja stvara dokument s početnim sadržajem. 2. Dokument dobiva jedinstveni identitet. 3. Bilježi se vrijeme stvaranja. 4. Vrsta sadržaja se zaključava prema početnom sadržaju, a zapis o zadnjoj izmjeni pokazuje sadržaj. 5. Strategija stvara fasadu za dokument. |
| **Alternativni tok** | A1. Početni sadržaj nije zadan: dokument se ne stvara, greška. A2. Fasadi nije predan dokument: fasada se ne stvara, greška. |
| **Ishod** | Dokument s identitetom, sadržajem i metapodacima te fasada kroz koju ga tok drži |

#### UC2 — Dohvati dokument kroz fasadu

| | |
|---|---|
| **Akter** | Pipeline (primarni), Strategija pisanja (primarni) |
| **Okidač** | Pipeline ili strategija pisanja treba podatke dokumenta |
| **Preduvjet** | Postoji fasada dokumenta (UC1) |
| **Glavni tijek** | 1. Pozivatelj traži od fasade sadržaj, identitet ili metapodatak. 2. Fasada ga uzima iz dokumenta iza sebe i vraća. |
| **Alternativni tok** | A1. Dokument iza fasade nije dostupan: greška. A2. Traženo svojstvo dokument nema: greška koja imenuje fasadu. A3. Tražen je sadržaj očišćenog dokumenta: greška dokumenta stiže pozivatelju nepromijenjena. |
| **Ishod** | Pozivatelj ima tražene podatke, a dokument je netaknut |

Metapodaci se izvana mogu samo čitati.

#### UC3 — Promijeni sadržaj

| | |
|---|---|
| **Akter** | Pipeline |
| **Okidač** | Pipeline je obradio podatke i treba zamijeniti sadržaj dokumenta |
| **Preduvjet** | Dokument postoji |
| **Glavni tijek** | 1. Pipeline predaje novi sadržaj. 2. Dokument provjerava da je sadržaj zadan i da je iste vrste kao dotadašnji. 3. Dokument preuzima novi sadržaj. 4. Dokument bilježi da je zadnje promijenjen sadržaj i kada. |
| **Alternativni tok** | A1. Novi sadržaj nije zadan: izmjena se odbija, greška. A2. Vrsta novog sadržaja razlikuje se od dotadašnje: izmjena se odbija, greška. |
| **Ishod** | Dokument ima novi sadržaj i ažuriran zapis o zadnjoj izmjeni |

#### UC4 — Zapiši metapodatak

| | |
|---|---|
| **Akter** | Pipeline |
| **Okidač** | Pipeline treba dokumentu dodati ili promijeniti opisni podatak |
| **Preduvjet** | Dokument postoji |
| **Glavni tijek** | 1. Pipeline predaje ključ i vrijednost. 2. Dokument provjerava ključ. 3. Dokument zapisuje vrijednost. 4. Dokument bilježi koji je ključ zadnje promijenjen i kada. |
| **Alternativni tok** | A1. Ključ je prazan: izmjena se odbija, greška. A2. Ključ je rezerviran za zapis o zadnjoj izmjeni: izmjena se odbija, greška. |
| **Ishod** | Metapodatak zapisan i zapis o zadnjoj izmjeni ažuriran |

#### UC5 — Očisti dokument

| | |
|---|---|
| **Akter** | Blackboard |
| **Okidač** | Platno Blackboarda se čisti |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Traži se čišćenje dokumenta. 2. Dokument briše sadržaj i metapodatke. |
| **Alternativni tok** | — |
| **Ishod** | Dokument bez sadržaja i metapodataka; identitet ostaje. Sadržaj se ne može čitati dok se ne zada novi, iste vrste. |

#### UC6 — Razvij ili održavaj vrstu dokumenta

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Tok treba novu vrstu podataka ili postojeća vrsta treba prilagodbu |
| **Preduvjet** | — |
| **Glavni tijek** | 1. Programer izrađuje ili mijenja vrstu dokumenta i određuje kako se mjeri njegova veličina. 2. Nova ili izmijenjena vrsta postaje dostupna strategijama stvaranja. |
| **Alternativni tok** | — |
| **Ishod** | Vrsta dokumenta koju strategija stvaranja može stvarati |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Document: Data persistence"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Create strategy" as Create
actor Pipeline
actor "Write strategy" as Write
actor Blackboard
actor Programmer

rectangle "Document" as Boundary <<system boundary>> {
    usecase "Stvori dokument" as UC1
    usecase "Dohvati dokument\nkroz fasadu" as UC2
    usecase "Promijeni sadržaj" as UC3
    usecase "Zapiši metapodatak" as UC4
    usecase "Očisti dokument" as UC5
    usecase "Razvij ili održavaj\nvrstu dokumenta" as UC6
}

Create -- UC1
Pipeline -- UC2
Write -- UC2
Pipeline -- UC3
Pipeline -- UC4
Blackboard -- UC5
Programmer -- UC6
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Dokument nema stroj stanja pa odjeljak o stanjima izostaje. Dijagram prikazuje normalni tok (od stvaranja do čišćenja) i alternativne tokove u kojima se radnja odbija greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Document: Data persistence"
caption "Flow Chart"

start
:Create strategy creates document\nwith initial content;
if (content given?) then (no)
  :<b><color:red>ValueError</color></b>;
  :error returned to Create strategy;
  stop
endif
:identity, creation time\nand last change recorded;
:Create strategy creates Facade;
if (Facade given a document?) then (no)
  :<b><color:red>TypeError</color></b>;
  :error returned to Create strategy;
  stop
endif
while (Pipeline works on document?) is (yes)
  if (change content or metadata?) then (content)
    if (content given?) then (no)
      :<b><color:red>ValueError</color></b>;
      :error returned to Pipeline;
      stop
    endif
    if (same type as before?) then (no)
      :<b><color:red>TypeError</color></b>;
      :error returned to Pipeline;
      stop
    endif
    :content replaced,\nlast change recorded;
  else (metadata)
    if (key valid and not reserved?) then (no)
      :<b><color:red>ValueError</color></b>;
      :error returned to Pipeline;
      stop
    endif
    :metadata written,\nlast change recorded;
  endif
endwhile (no)
if (request through Facade valid?) then (no)
  :<b><color:red>ValueError / AttributeError</color></b>;
  :error returned to caller;
  stop
endif
:document cleaned\n(content and metadata removed);
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ValueError` | sadržaj nije zadan (pri stvaranju ili izmjeni); ključ metapodatka je prazan ili rezerviran; traži se sadržaj očišćenog dokumenta; fasada od adaptera ne dobije dokument | radnja se odbija i dokument ostaje kakav je bio; onaj tko ju je zatražio prima grešku |
| `TypeError` | vrsta novog sadržaja razlikuje se od dotadašnje; pri stvaranju fasade ili adaptera nije predan dokument | radnja se odbija; onaj tko ju je zatražio prima grešku |
| `AttributeError` | preko fasade traži se svojstvo koje dokument nema ili koje je skriveno | greška imenuje fasadu, ne dokument; prima je pozivatelj |

Kad dokument traženo svojstvo nema, greška imenuje fasadu, sloj na kojem je poziv nastao. Svaka druga greška iz dokumenta (npr. čitanje sadržaja očišćenog dokumenta) stiže pozivatelju nepromijenjena. Dokument ne bilježi svaku izmjenu ključa kao zaseban događaj; promjena je zabilježena u zapisu o zadnjoj izmjeni.

### 2.6 Pretpostavke i ograničenja

- Dokument ne zna kako je nastao ni kamo ide: to znaju strategije stvaranja i pisanja.
- Veličinu dokumenta određuje konkretna vrsta, jer je mjera različita za tekst, tablicu i niz bajtova.
- Zapis o zadnjoj izmjeni čuva samo posljednju izmjenu, ne cijeli popis izmjena.
- Identitet se dodjeljuje pri stvaranju i vrijedi samo za taj dokument; nije otisak sadržaja.
- Dokument nema stanja pauze ni oporavka. Nakon čišćenja sadržaj se ne može čitati dok se ne zada novi.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-DCM-01** | Svaki dokument ima jedinstveni identitet, određen pri stvaranju i nepromjenjiv. |
| **BR-DCM-02** | Dokument ne može biti bez sadržaja: sadržaj koji nije zadan odbija se pri stvaranju i pri izmjeni. Prazan tekst ili prazna tablica jesu sadržaj. |
| **BR-DCM-03** | Vrsta sadržaja određena je prvim sadržajem; svaka kasnija izmjena sadržaja mora biti iste vrste. |
| **BR-DCM-04** | Podaci žive u sadržaju, a metapodaci ih samo opisuju (izvor, ime datoteke, shema, trag obrade). |
| **BR-DCM-05** | Dokument pri stvaranju bilježi vrijeme stvaranja, a pri svakoj izmjeni sadržaja ili metapodatka bilježi što je promijenjeno i kada; vrijeme dolazi iz jednog zajedničkog izvora. |
| **BR-DCM-06** | Zapis o zadnjoj izmjeni piše samo dokument; izvana se ne može postaviti, kako se povijest izmjena ne bi krivotvorila. |
| **BR-DCM-07** | Ključ metapodatka ne smije biti prazan. |
| **BR-DCM-08** | Metapodaci se izvana mogu samo čitati; mijenjaju se isključivo propisanom radnjom. |
| **BR-DCM-09** | Čišćenje briše sadržaj i metapodatke, a identitet ostaje. Nakon čišćenja sadržaj se ne može čitati dok se ne zada novi, iste vrste. |
| **BR-DCM-10** | Metapodaci se brišu samo izričitim čišćenjem: tko ih još čita kad dokument prestane postojati, i dalje ih vidi. |
| **BR-DCM-11** | Dva dokumenta smatraju se istim samo ako imaju isti identitet i istu vrstu. |
| **BR-DCM-12** | Tok drži fasadu, ne dokument; fasada propušta pozivatelju javna svojstva dokumenta. |
| **BR-DCM-13** | Fasada se ne stvara bez dokumenta i nikad ne vraća prazan rezultat umjesto dokumenta. |
| **BR-DCM-14** | Veličinu dokumenta određuje njegova vrsta. |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; gdje postoji zapis zahtjeva na kakvoću (NFRQ), poveznica je navedena.

| kakvoća | što znači za dokument |
|---|---|
| sljedivost | svaki dokument ima identitet i zapis o zadnjoj izmjeni (BR-DCM-01, BR-DCM-05); vrijeme se zapisuje jednim zajedničkim izvorom ([`NFRQ-DFN-04`](../03-NFRQ/NFRQ-DFN-04-moment-aware-helper.md)) |
| integritet | vrsta sadržaja ne mijenja se usput, a zapis o zadnjoj izmjeni ne može se krivotvoriti (BR-DCM-03, BR-DCM-06) |
| razmjerno bilježenje | izmjena pojedinog ključa nije zasebna jedinica rada i ne bilježi se zasebno; izmjena je vidljiva u zapisu o zadnjoj izmjeni ([`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md)) |
| razumljivost grešaka | kad dokument traženo svojstvo nema, greška imenuje fasadu, sloj na kojem je poziv nastao |
| proširivost | programer može izraditi nove vrste dokumenata (UC6) |
| preglednost | tok za sve vrste dokumenata koristi isti način pristupa (BR-DCM-12) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-DOC-*` su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. **Vrijeme stvaranja nije zaštićeno.** Zapis o zadnjoj izmjeni izvana se ne može postaviti (BR-DCM-06), ali vrijeme stvaranja može se prepisati kao običan metapodatak. Treba odlučiti treba li i ono biti zaštićeno.
4. **Ponovno punjenje nakon čišćenja.** Očišćenom dokumentu može se ponovno zadati sadržaj iste vrste, a vrijeme stvaranja pritom se ne obnavlja. Treba potvrditi je li to željeno ili je čišćenje konačno.
5. **Tko čisti dokumente.** Zapis pretpostavlja da se dokumenti čiste kad se čisti platno Blackboarda; generički Blackboard ([`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), UC6) to ne propisuje. Treba potvrditi tko traži čišćenje dokumenta.
6. **Vrste grešaka.** Dokument javlja uobičajene greške jezika (`ValueError`, `TypeError`, `AttributeError`), a ne vlastitu grešku komponente kao procesor, pipeline ili upravitelji. Treba potvrditi je li to željeno.
7. Kakvoće „sljedivost", „integritet", „razumljivost grešaka", „proširivost" i „preglednost" (§4) nemaju vlastiti NFRQ zapis; treba ih povezati s postojećim ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard koji drži dokumente |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor koji provodi dokumente kroz pipelineove |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | pipeline koji prima dokument kroz fasadu |
| [`NFRQ-DFN-04`](../03-NFRQ/NFRQ-DFN-04-moment-aware-helper.md) | jedinstveni izvor vremena |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | opseg bilježenja izmjena |
| `concrete/document.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.3 | 2026-10-11 | Uklonjene poveznice na prethodni zapis iste komponente i pripadna stavka iz *Otvoreno*: zapis ga zamjenjuje, pa upućuje samo na važeće dokumente. |
| v0.0.2 | 2026-10-11 | Usklađeno s kodom: odbija se sadržaj koji nije zadan, a prazan tekst ili tablica jesu sadržaj (BR-DCM-02, UC1, UC3, tok); nakon čišćenja identitet ostaje i može se zadati novi sadržaj iste vrste (BR-DCM-09, UC5, §2.6); kroz fasadu se preimenuje samo greška nepostojećeg svojstva, ostale stižu nepromijenjene (UC2, §2.5); razdvojeni uzroci grešaka `ValueError` i `TypeError` za fasadu. Dodana poveznica na StRS-PIPELINE. *Otvoreno*: uklonjena zastarjela stavka o oznaci i stavka o `DummyReadDocument` (razina SyRS-a); dodana pitanja o zaštiti vremena stvaranja, ponovnom punjenju nakon čišćenja i o tome tko čisti dokumente. |
| v0.0.1 | 2026-10-10 | Prvi zapis, izveden iz koda `concrete/document.py`. |