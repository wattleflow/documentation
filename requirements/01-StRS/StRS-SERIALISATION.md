<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-SERIALISATION — Granica formata: parser, formater i konverter (generički sloj)

| | |
|---|---|
| **Verzija** | v0.0.9 |
| **Odluka** | Tri uloge u ontologiji (`CNV`, `PAR`, `FMT`); lagana razina; konverter nije `IDriver`; granica parsera i formatera |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Distribucija** | `wattleflow-workflow` — clean core tier |
| **Predmet** | `workflow/src/wattleflow/concrete/serialisation.py` |
| **Djeca (SyRS)** | [`SyRS-SER-PAR`](../02-SyRS/SyRS-SRL-PAR-parser.md) · [ `SyRS-SER-FMT`](../02-SyRS/SyRS-SRL-FMT-formatter.md) · [`SyRS-SER-CNV`](../02-SyRS/SyRS-SRL-CNV-converter.md) |
| **Sljedivost** | [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) · [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) · [`NFRQ-ORG-13`](../03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) · [`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md) · [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava zašto framework ima zajednički mehanizam za pretvorbu podataka iz jednog formata u drugi, tko ga koristi, kako se odvija pretvorba i što se događa kad nešto pođe po zlu.

### 1.2 Opseg

**U opsegu:** zajednički mehanizam pretvorbe — tri uloge (parser, formater, konverter), strategija pretvorbe i greške koje mogu javiti.

**Izvan opsega:** pretvorbe za pojedine formate (opisane u [`SyRS-CNV-01…07`](../02-SyRS/SyRS-000-INDEX.md)) i tehnički detalji izvedbe (opisani u zahtjevima sustava, SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| tok | workflow | niz koraka kroz koje podaci prolaze od izvora do odredišta |
| pozivatelj | Caller | korak toka koji traži pretvorbu podataka |
| format | — | način na koji su podaci zapisani (npr. Word, PDF, običan tekst) |
| parser | Parser | čita podatke u izvornom formatu i provjerava ih |
| formater | Formatter | oblikuje pročitane podatke u traženi format |
| konverter | Converter | izvodi pretvorbu zadanom strategijom i vraća rezultat |
| strategija | ConversionStrategy | par parser–formater koji zajedno daju traženu pretvorbu |
| izvor | source | podaci koje parser čita; zadaju se točno jednim od tri načina: otvoreni tok podataka, putanja do datoteke ili podaci u memoriji |
| greška | Error | obavijest pozivatelju da pretvorba nije uspjela, s uzrokom |
| uzrok | cause | izvorna greška koja kaže u kojoj je ulozi i zašto pretvorba pala |
| StRS | — | specifikacija zahtjeva dionika (*Stakeholder Requirements Specification*): što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava (*System Requirements Specification*): što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Podaci koji putuju tokom i njegovim pipelineima često zahtijevaju konverziju iz jednog formata u drugi (npr. iz Word ili PDF dokumenta u običan tekst i obratno). Čitanje i pisanje podataka te njihovu pretvorbu programeri često rješavaju svatko na svoj način, što dovodi do neujednačenih rezultata i redundantnog koda raspršenog po bibliotekama. Stoga su potrebna jedinstvena pravila i algoritmi serijalizacije i deserijalizacije te zamjenjive strategije konverzije koje se mogu pozvati po potrebi iz bilo kojeg dijela toka.

**Rješenje** `workflow` framework treba modularan i proširiv mehanizam serijalizacije i deserijalizacije. Mehanizam se temelji na generičkom konverteru koji je agnostičan prema strategiji konverzije. Pozivatelj konverteru zadaje željenu strategiju; strategija sadrži odgovarajući par `parser`–`formater` zadužen za pretvorbu danih podataka u traženi oblik. Svi parovi koriste ujednačeno sučelje i zajedničku taksonomiju formata.

**Smjer** Mehanizam se rastavlja na tri uloge, svaku s jednom zadaćom: parser čita i provjerava izvor, formater oblikuje izlaz, a konverter ih povezuje odabranom strategijom. Novi format dodaje se specijalizacijom parsera i formatera, bez izmjene pozivatelja, čime jezgra ostaje čista (*clean core*): generički sloj propisuje ugovor, a specijalizacije po formatima obrađuje [`SyRS-CNV-01…07`](../02-SyRS/SyRS-000-INDEX.md).

Rješenje se oslanja na postojeća sučelja frameworka **wattleflow**, pa nije potrebno graditi novu osnovu. Jednostavan dizajn ujednačuje pretvorbu podataka i jamči:

- **modularnost**: parser, formater i konverter samostalni su dijelovi koji se mogu zamijeniti bez utjecaja na ostale;
- **proširivost**: podrška za novi format dodaje se bez izmjene postojećeg koda;
- **zadanu brzinu**: pretvorba se izvršava unutar dogovorenih vremenskih okvira;
- **dosljedne i ponovljive rezultate**: isti ulaz uvijek daje isti izlaz, bez obzira na to tko i gdje pokreće pretvorbu.

### 2.2 Mjesto u dekompoziciji

Mehanizam se sastoji od tri uloge. Svaku detaljno opisuje zaseban zapis zahtjeva sustava (SyRS):

| dijete | uloga | zadaća |
|---|---|---|
| [ `SyRS-SER-PAR`](../02-SyRS/SyRS-SRL-PAR-parser.md) | parser | čita i provjerava izvorne podatke |
| [ `SyRS-SER-FMT`](../02-SyRS/SyRS-SRL-FMT-formatter.md) | formater | oblikuje podatke u traženi format |
| [ `SyRS-SER-CNV`](../02-SyRS/SyRS-SRL-CNV-converter.md) | konverter | izvodi strategiju i vraća rezultat pozivatelju |

**Kontekst.** Pozivatelj iz toka zadaje konverteru strategiju i predaje mu izvorne podatke. Konverter izvodi strategiju, a strategija svojim parom parser–formater daje rezultat. Konverter rezultat vraća pozivatelju, koji ga sam pohranjuje gdje treba.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Serialisation: Data conversion"
caption "Context Diagram"

actor "Caller\n(step in a workflow)" as Caller
database "Source data" as Source
database "Target data" as Target

rectangle "Serialisation" as Boundary <<system boundary>> {
  component Converter
  package ConversionStrategy {
    component Parser
    component Formatter
  }
}

Caller ..> Converter : sets strategy,\nrequests conversion
Converter ..> ConversionStrategy : executes
Source --> Converter : source
Converter --> Caller : result / error
Caller --> Target : stores
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Pozivatelj | korak toka kojem trebaju podaci u drugom formatu | zadaje strategiju, predaje podatke i preuzima rezultat ili grešku |
| Programer | razvija podršku za nove formate | dodaje parser, formater i strategiju za novi format |

### 2.4 Korisnički slučajevi

| | UC1 | UC3 |
|---|---|---|
| **Naziv** | Pretvori podatke | Dodaj novi format |
| **Akter** | Pozivatelj | Programer |
| **Okidač** | Korak toka treba podatke u drugom formatu | Tok treba format koji još nije podržan |
| **Preduvjet** | Strategija za traženu pretvorbu postoji | — |
| **Glavni tijek** | 1. Pozivatelj zadaje strategiju (UC2). 2. Pozivatelj predaje izvorne podatke. 3. Konverter izvodi strategiju: parser čita i provjerava podatke, formater ih oblikuje. 4. Konverter vraća rezultat pozivatelju. | 1. Programer izrađuje parser, formater ili oboje za novi format. 2. Programer sastavlja strategiju od para parser–formater. |
| **Alternativni tok** | A1. Strategija nije prave vrste: konverter je odbija. A2. Strategija nije zadana, izvor nije ispravan, ili čitanje ili oblikovanje ne uspije: pretvorba se prekida i pozivatelj prima grešku konvertera s uzrokom (§2.5). | — |
| **Ishod** | Pozivatelj ima podatke u traženom formatu | Nova strategija dostupna je pozivateljima bez izmjene konvertera |

UC2 (*Zadaj strategiju*) dio je UC1 i nema zaseban opis. Strategija se može zadati pri stvaranju konvertera ili kasnije.

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Serialisation: Data conversion"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor "Caller\n(step in a workflow)" as Caller
actor "Programmer" as Dev

rectangle "Serialisation" as Boundary <<system boundary>> {
    usecase "Pretvori podatke" as UC1
    usecase "Zadaj strategiju" as UC2
    usecase "Dodaj novi format" as UC3
}

Caller -- UC1
Dev -- UC3
UC1 ..> UC2 : <<include>>
@enduml
```

### 2.5 Tok pretvorbe i praćenje grešaka

Dijagram prikazuje normalni tok (od zadavanja strategije do vraćanja rezultata) i alternativne tokove u kojima se pretvorba prekida greškom. Pozivatelj uvijek prima grešku konvertera; njezin uzrok kaže u kojoj je ulozi kvar nastao.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Serialisation: Data conversion"
caption "Flow Chart"

start
:Caller sets strategy;
if (strategy of the right kind?) then (no)
  :<b><color:red>ConverterError</color></b>;
  :error returned to Caller;
  stop
endif
:Caller requests conversion;
if (strategy set?) then (no)
  :<b><color:red>ConverterError</color></b>;
  :error returned to Caller;
  stop
endif
:Parser checks source;
if (exactly one usable source?) then (no)
  :<b><color:red>ParserError</color></b>\nas cause of ConverterError;
  :error returned to Caller;
  stop
endif
:Parser reads source;
if (reading failed?) then (yes)
  :<b><color:red>ParserError</color></b>\nas cause of ConverterError;
  :error returned to Caller;
  stop
endif
:Formatter shapes result;
if (shaping failed?) then (yes)
  :<b><color:red>FormatterError</color></b>\nas cause of ConverterError;
  :error returned to Caller;
  stop
endif
:result returned to Caller;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `ConverterError` | strategija nije prave vrste ili nije zadana; ili je pretvorba unutar strategije ne uspjela | pretvorba se prekida; pozivatelj prima grešku i ne dobiva rezultat |
| `ParserError` (kao uzrok) | izvor nije zadan točno jednom, nije upotrebljiv ili se ne može pročitati | uzrok greške konvertera pokazuje da je kvar u parseru |
| `FormatterError` (kao uzrok) | nema sadržaja, sadržaj nije propisane vrste ili se ne može oblikovati | uzrok greške konvertera pokazuje da je kvar u formateru |
| `StrategyError` (kao uzrok) | strategija ne može razriješiti izvor | uzrok greške konvertera pokazuje da je kvar u strategiji |

Neočekivani kvar u parseru ili formateru najprije postaje greška te uloge, a zatim uzrok greške konvertera. Izvorni uzrok uvijek je sačuvan.

### 2.6 Pretpostavke i ograničenja

- Mehanizam se oslanja na postojeće temelje frameworka wattleflow.
- Mehanizam ne ovisi o vanjskim bibliotekama.
- Mehanizam ne vodi vlastiti zapis aktivnosti; podaci o kvaru putuju u samoj grešci.
- Mehanizam ne pohranjuje rezultat; to radi pozivatelj.
- Formati bi se trebali označavati zajedničkim nazivljem (taksonomijom), kako bi svi dijelovi toka isti format zvali istim imenom; generički sloj taksonomiju još ne sadrži (§6).

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-PRS-01** | Parser po svakom pozivu prima točno jedan izvor podataka. Ako izvora nema, ako ih je više ili ako izvor nije upotrebljiv, parser odbija posao prije čitanja. |
| **BR-PRS-02** | Pravila provjere izvora definirana su jednom i vrijede za sve parsere. |
| **BR-FRM-01** | Formater vraća rezultat i ništa ne pohranjuje; pohrana je posao pozivatelja. |
| **BR-FRM-02** | Formater bez ulaznih podataka odbija posao. Vrstu podataka provjerava samo ako je za format propisana. |
| **BR-FRM-03** | Izgled izlaza određuje isključivo formater. |
| **BR-CNV-01** | Konverter u svakom trenutku izvodi jednu strategiju. Bez strategije ili s neprikladnom strategijom odbija posao. |
| **BR-CNV-02** | Konverter samo izvodi strategiju; ne upravlja uređajima ni pohranom. |
| **BR-SRL-01** | Neočekivani kvar prijavljuje se kao greška uloge u kojoj je nastao, uz sačuvan uzrok. Pozivatelj konvertera uvijek prima grešku konvertera, a uzrok pokazuje ulogu u kojoj je kvar nastao. |
| **BR-SRL-02** | Kodiranje znakova u tekstu može zadati pozivatelj; inače vrijedi zadano kodiranje formata. |
| **BR-SRL-03** | Mehanizam ne vodi vlastiti zapis aktivnosti; podaci o kvaru putuju u samoj grešci. |
| **BR-SRL-04** | Mehanizam ne ovisi o vanjskim bibliotekama ([`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md)). |
| **BR-SRL-05** | Strategija se uvijek sastoji od jednog parsera i jednog formatera. |
| **BR-SRL-06** | Parser i formater međusobno su neovisni i mogu se zamijeniti zasebno. |

## 4. Nefunkcionalni zahtjevi

| zahtjev | što znači za ovaj mehanizam |
|---|---|
| [`NFRQ-SCR-03`](../03-NFRQ/NFRQ-SCR-03-supply-chain-locality.md) | mehanizam koristi samo vlastiti kod frameworka i standardnu biblioteku (`BR-SRL-04`) |
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | tri uloge pripadaju zatvorenom popisu uloga frameworka; konverter nije nova vrsta elementa |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | zajednička pravila (provjera izvora, prijava grešaka) definirana su na jednom mjestu |
| [`NFRQ-ORG-13`](../03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) | ono što je javno dostupno čini sučelje mehanizma |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | mehanizam radi bez detaljnog zapisa aktivnosti (`BR-SRL-03`) |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

| oznaka |  `SyRS-SER-PAR` |  `SyRS-SER-FMT` |  `SyRS-SER-CNV` |
|---|:-:|:-:|:-:|
| UC1 | ✔ | ✔ | ✔ |
| UC2 | | | ✔ |
| UC3 | ✔ | ✔ | |
| BR-PRS-01, BR-PRS-02 | ✔ | | |
| BR-FRM-01, BR-FRM-02, BR-FRM-03 | | ✔ | |
| BR-CNV-01, BR-CNV-02, BR-SRL-05 | | | ✔ |
| BR-SRL-01, BR-SRL-03, BR-SRL-04 | ✔ | ✔ | ✔ |
| BR-SRL-02, BR-SRL-06 | ✔ | ✔ | |

## 6. Otvoreno

1. Narativ jamči zadanu brzinu i ponovljive rezultate, a nijedan nefunkcionalni zahtjev (§4) to još ne pokriva.
2. **Taksonomija formata.** Narativ kaže da svi parovi koriste zajedničku taksonomiju formata, a generički sloj je ne sadrži. Treba odlučiti gdje se taksonomija vodi (generički sloj ili [`SyRS-CNV-01…07`](../02-SyRS/SyRS-000-INDEX.md)).
3. **Provjera vrste podataka u formateru ne radi.** Formater kojem je propisana vrsta podataka (BR-FRM-02) trenutačno uvijek javlja grešku, jer provjeru ne može pokrenuti. Kod treba ispraviti (SyRS).
4. **Pravilo BR-SRL-05** generički sloj ne nameće: strategija ne mora sadržavati par parser–formater, nego samo izvesti pretvorbu. Treba potvrditi je li pravilo obveza za sve strategije ili preporuka.
5. Naziv sloja u naslovima dijagrama („Data conversion") je prijedlog.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [ `SyRS-CNV-01…07`](../02-SyRS/SyRS-000-INDEX.md) | pretvorbe za pojedine formate |
| [ `SyRS-SER-PAR`](../02-SyRS/SyRS-SRL-PAR-parser.md) · [ `SyRS-SER-FMT`](../02-SyRS/SyRS-SRL-FMT-formatter.md) · [ `SyRS-SER-CNV`](../02-SyRS/SyRS-SRL-CNV-converter.md) | zahtjevi sustava (SyRS) |
| `workflow/src/wattleflow/concrete/serialisation.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.9 | 2026-10-11 | Oznaka bez broja (`StRS-SERIALISATION`). Usklađeno s kodom: pozivatelj uvijek prima grešku konvertera, a greške parsera, formatera i strategije stižu kao uzrok (UC1, BR-SRL-01, tok, tablica grešaka); dodana greška strategije; izvor mora biti upotrebljiv (BR-PRS-01). Dijagrami: naslov „Serialisation: Data conversion", alias granice sustava. *Otvoreno*: taksonomija formata, provjera vrste podataka u formateru, obveznost BR-SRL-05, novi nadređeni zapis. |
| v0.0.7 | 2026-10-10 | Zapis pročišćen za široku publiku: tehnički detalji (nazivi iz koda, komponentni dijagram, tablice formata, stanje testova i odstupanja koda od dizajna) premješteni u zahtjeve sustava; poslovna pravila preformulirana jednostavnim jezikom; dodani okidači i alternativni tokovi korisničkih slučajeva te tablica praćenja grešaka. |
| v0.0.6 | 2026-10-10 | Struktura usklađena s ISO/IEC/IEEE 29148; dodana pravila `BR-SRL-05`, `BR-SRL-06`, `BR-FRM-03`; ispravljeni dijagrami. |
| v0.0.5 | 2026-10-10 | Zapis prepisan: narativ kao Problem · Rješenje · Smjer, dekompozicija s dijagramima konteksta, slučajeva korištenja, komponenti i toka (`wattleflow-docs` §3h). |
| v0.0.5 | 2026-10-06 | Kuka parsera je zaštićena `_deserialise` (`NFRQ-ORG-13` k.2). |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — lagana razina |