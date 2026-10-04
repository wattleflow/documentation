<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-DOC — Dokument, adapter i fasada

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — narativ, `BR-WFL-01…02`, `BR-PTN-01…05`, `BR-PRC-01`, `BR-DRV-01`, zajednički ugovor generičke klase (§4), `BR-PTN-07` (`__slots__`) |
| **Predmet** | `Document(Wattleflow, IAdaptee, Generic[Content], ABC)`, `DocumentAdapter(Wattleflow, IAdapter, Generic[Adaptee])`, `DocumentFacade(Wattleflow, ITarget, Generic[Adaptee], ABC)`, `DummyReadDocument(Document[dict])` |
| **Sestrinski** | [`FRQ-PIP`](FRQ-PIP-pipeline.md) (prima `facade`) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) (drži ih na platnu) · [`FRQ-STR`](FRQ-STR-strategy.md) (stvara i piše) |
| **Izvedba** | `workflow/src/wattleflow/concrete/document.py` (286 linija) |
| **Dijagrami** | inline (02, 03, 06, 08, 09) — pogledi izvedeni iz koda, ne izvor istine (D-13) |

# Index
- [01. Interface](#01-interface)
- [02. Class Diagrams](#02-class-diagrams)
- [03. Context Diagram](#03-context-diagram)
- [04. User Diagram](#04-user-diagram)
- [05. Events](#05-events)
- [06. Use Case Diagrams](#06-use-case-diagrams)
- [07. Constraints and Preconditions](#07-constraints-and-preconditions)
- [08. Sequence Diagrams](#08-sequence-diagrams)
- [09. Flow Chart Diagrams](#09-flow-chart-diagrams)
- [10. State Machine](#10-state-machine)
- [11. Non-Functional Requirements](#11-non-functional-requirements)
- [12. Results](#12-results)
- [13. Acceptance Criteria](#13-acceptance-criteria)
- [14. Verification](#14-verification)
- [15. Open issues](#15-open-issues)
- [16. References](#16-references)
- [17. Change history](#17-change-history)

## 01. Interface

Dokument je **jedinica podatka koja putuje tokom**. Tri klase, tri različita razloga:

| klasa | pattern | zašto postoji |
|---|---|---|
| `Document` | Adaptee | nosi **sadržaj, identitet i povijest izmjena** |
| `DocumentAdapter` | Adapter | prevodi dokument u ono što tok očekuje |
| `DocumentFacade` | Facade | ono što pipeline i strategije **stvarno drže** u rukama |
| `DummyReadDocument` | rezervni dokument | što vrati čitanje koje nitko nije napisao (`StrategyRead`); sam se označava: `implemented=False` u metapodacima i napomena u sadržaju |

Pipeline nikad ne dobiva `Document` nego `DocumentFacade`. Razlog je delegacija: fasada
prosljeđuje svako nepoznato **javno** ime dokumentu iza sebe, pa strategija koja treba `filename`
ili `identifier` piše `facade.filename` bez obzira na to kojeg je tipa dokument. Nepoznato ime
koje dokument nema daje `AttributeError` s imenom **fasade**, ne adaptera — trag pokazuje na sloj
na kojem je poziv nastao.

**Povijest izmjena je zaštićena.** Ključevi `last_change_key` i `last_change_time` su rezervirani
(`_AUDIT_KEYS`): pozivatelj ih **ne smije** postaviti, jer bi time falsificirao povijest izmjene
dokumenta. Piše ih isključivo `update_metadata`/`update_content`, iznutra.

**Tip sadržaja se zaključava.** Prvi dodijeljeni sadržaj određuje `_expected_type`; svaka sljedeća
izmjena mora biti istog tipa. Dokument koji je počeo kao tekst ne postaje binaran usput.

**Sadržaj nosi podatke, metapodaci ih opisuju** (dokumentirana odluka 2026-09-11). Podaci — tekst, zapisi,
tablica — žive u sadržaju dokumenta. Metapodaci nose ono što podatke opisuje: izvor, ime datoteke,
list, shemu, broj redaka, trag obrade. Za strukturirane podatke koristi se dokument kojem je sadržaj
tablica (DataFrame), **gdje god je to moguće**; ime datoteke taj dokument već čita i piše kroz
metapodatke. Podaci u metapodacima su skriven kanal: vrsta dokumenta ne govori što nosi, a pristup
ide preko dogovorenog ključa.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Document, adapter and facade

top to bottom direction

  interface IAdaptee
  interface IAdapter
  interface ITarget
  class Wattleflow
  abstract class "Document<Content>" as Doc {
    +content : Content
    +identifier : str
    +metadata : Mapping
    {abstract} +size : int
    +update_content(content)
    +update_metadata(key, value)
    +clean()
    +specific_request() : Document
  }
  class "DocumentAdapter<Adaptee>" as DA {
    +adaptee : Adaptee
    +request()
  }
  abstract class "DocumentFacade<Adaptee>" as DF {
    +request() : Adaptee
  }
  class DummyReadDocument {
    {static} NOTICE
    +size : int
  }
Wattleflow <|-down- Doc
Wattleflow <|-down- DA
Wattleflow <|-down- DF
IAdaptee <|.down. Doc
IAdapter <|.down. DA
ITarget <|.down. DF
Doc <|-down- DummyReadDocument
DF *-down- "1" DA : _adapter
DA -right-> "1" IAdaptee : _adaptee
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Document, adapter and facade

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Document, DocumentFacade", "wattleflow.concrete.document: unit of data travelling through the flow")

System_Ext(pi, "GenericPipeline", "Transforms")
System_Ext(bb, "GenericBlackboard", "Holds facades")
System_Ext(sc, "StrategyCreate", "Creates document")
System_Ext(sw, "StrategyWrite", "Writes document")

Rel_D(pi, sys, "Changes content and metadata")
Rel_R(bb, sys, "Clears canvas (clean)")
Rel_L(sc, sys, "Builds document and facade")
Rel_U(sw, sys, "Reads through facade")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `StrategyCreate` — stvara dokument | sadržaj |
| **A2** | `GenericPipeline` — transformira | `facade` |
| **A3** | `StrategyWrite` — čita sadržaj i metapodatke pri pisanju | `facade` |
| **A4** | `GenericBlackboard` — drži fasade na platnu | ključ (identifikator ili digest) |
| **A5** | `Document` / `DocumentFacade` — predmet ovog zahtjeva | sadržaj i povijest |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 gradi dokument → `__init__` → `created_at` + prvi sadržaj |
| **EV02** | fasada se gradi nad dokumentom → `DocumentFacade.__init__` |
| **EV03** | A2 mijenja sadržaj → `update_content` |
| **EV04** | A2/A3 mijenja metapodatak → `update_metadata` |
| **EV05** | A3 čita kroz fasadu → `facade.<ime>` → delegacija |
| **EV06** | platno se prazni → `clean()` |
| **EV07** | kraj životnog ciklusa → `__del__` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Document, adapter and facade

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "StrategyCreate" as A1
  actor "GenericPipeline" as A2
  actor "StrategyWrite" as A3
  actor "GenericBlackboard" as A4
    usecase "Build document" as EV01
    usecase "Build facade" as EV02
    usecase "Change content" as EV03
    usecase "Change metadata" as EV04
    usecase "Read through facade" as EV05
    usecase "Clear canvas" as EV06
    usecase "End lifecycle" as EV07
  A1 --> EV01
  A1 --> EV02
  A2 --> EV03
  A2 --> EV04
  A3 --> EV04
  A3 --> EV05
  A4 --> EV06
  A4 --> EV07
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Specijalizacija dokumenta je implementirala `size`.
2. Objekt predan adapteru i fasadi je `IAdaptee` — inače `TypeError`.
3. Ključ metapodatka nije prazan i nije rezerviran.
4. Dokument ima sadržaj čim ga strategija stvaranja (`StrategyCreate`) inicijalizira: `None` se ne prima ni pri konstrukciji ni kasnije (`update_content`).
5. Dokument s datotekom (`FileDocument` i podrazredi, `blackwattle`) je referenca na datoteku (`filename` u metapodacima) uz tekstualni sadržaj `str`: prazan niz znači „teksta još nema", nikad `None` ni `Path`. Strategija stvaranja predaje samo `filename`; sadržaj pune cjevovodi. Sadržaj-staza nije dopušten, a novi tip za sadržaj datoteke nije potreban.

## 08. Sequence Diagrams

### Normalan tok

1. **EV01** — identitet je `uuid4()`, dodijeljen pri konstrukciji i **nepromjenjiv** izvana
   (`BR-PTN-01`). Metapodaci kreću praznim rječnikom, pa se upisuje `created_at` iz `Now.utc()`,
   pa prvi sadržaj.
2. **EV02** — fasada gradi adapter **s istom konfiguracijom** koju je sama dobila: adapter je
   njezin implementacijski detalj i ne smije padati na zadane postavke zapisa.
3. **EV03** — prvi sadržaj postavlja `_expected_type`; svaki sljedeći se provjerava prema njemu.
   Uspješna izmjena upisuje `last_change_key="content"` i vrijeme.
4. **EV04** — `update_metadata` odbija prazan ključ i rezervirani ključ, pa upiše vrijednost i
   osvježi `last_change_key` / `last_change_time`.
5. **EV05** — `facade.<ime>`: `_`-imena se odbijaju odmah; javno ime se traži na dokumentu iza
   adaptera i prosljeđuje ako postoji.
6. **EV06** — `clean()` oslobađa sadržaj i briše metapodatke.
7. Metapodaci se izvana vide **samo za čitanje** (`MappingProxyType`), kao i platno blackboarda.

### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Document, adapter and facade

  participant "StrategyCreate" as S
  participant "Document" as Dk
  participant "DocumentFacade" as F
  participant "DocumentAdapter" as A
  participant "GenericPipeline" as P
  participant "StrategyWrite" as W
  S -> Dk : Document(content)
  activate Dk #gold
  Dk -> Dk : created_at, first content
  deactivate Dk
  S -> F : DocumentFacade(adaptee)
  activate F #gold
  F -> A : adapter with the same configuration
  activate A #gold
  deactivate A
  deactivate F
  P -> F : update_content(content)
  activate F #gold
  F -> A : request()
  activate A #gold
  A -> Dk : specific_request()
  activate Dk #gold
  deactivate Dk
  deactivate A
  F -> Dk : update_content(content)
  activate Dk #gold
  deactivate Dk
  deactivate F
  W -> F : facade.<name>
  activate F #gold
  F -> A : request()
  activate A #gold
  A -> Dk : specific_request()
  activate Dk #gold
  deactivate Dk
  deactivate A
  F -> Dk : getattr(<name>) (public names only)
  activate Dk #gold
  deactivate Dk
  deactivate F
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| sadržaj drugog tipa nego prvi | `TypeError` s oba imena tipa | dokument ne mijenja prirodu usput |
| `update_content(None)` ili konstrukcija s `None` | `ValueError` | `None` nije sadržaj; sadržaj nestaje samo kroz `clean()`, a sadržaj i povijest ostaju nepromijenjeni |
| čitanje `content` nakon `clean()` | **`ValueError`** | upotreba nakon čišćenja pada odmah |
| prazan ključ metapodatka | `ValueError` | ključ bez imena nije metapodatak |
| rezervirani ključ (`last_change_*`) | `ValueError` s objašnjenjem | povijest se ne može falsificirati |
| objekt koji nije `IAdaptee` | `TypeError` iz adaptera odnosno fasade | tok ne prima tuđi tip |
| `request()` vrati `None` | `ValueError` s imenom klase | prazna fasada ne ide dalje |
| pristup `_`-imenu kroz fasadu | `AttributeError` s imenom **fasade** | delegacija ne otvara privatnu površinu |
| nepoznato javno ime | `AttributeError` s imenom **fasade** | trag pokazuje na sloj poziva |
| iznimka u `__del__` | `try/except Exception: pass` | destruktor ne diže ([`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §4 t.4) |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Document, adapter and facade

start
:update_content(content);
if (content is None?) then (yes)
:content is cleared; last_change_key, last_change_time written;
else (no)
if (first content?) then (yes)
  :_expected_type = content type;
else (no)
  if (content is instance of _expected_type?) then (no)
    :TypeError with both type names;
    stop
  else (yes)
  endif
endif
:content = content; last_change_key, last_change_time;
endif
:update_metadata(key, value);
if (key empty or reserved?) then (yes)
:ValueError;
stop
else (no)
:write value; refresh last_change_*;
endif
stop
@enduml
```

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica za ovaj zahtjev |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | `Document` je rezervirani primitiv; adapter i fasada su patterni oko njega, ne novi primitivi |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | fasada odbija delegirati `_`-imena; metapodaci su read-only pogled |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | clean core tier; nijedan third-party uvoz — formati (PDF, DOCX, Avro…) žive u processorsu |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | povijest izmjena je zaštićena od pozivatelja — `AU-3` integritet zapisa počinje ovdje |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | konstrukcija, izmjena sadržaja i čišćenje su `DEBUG`; izmjena metapodatka nema zapis; dokument nije jedinica posla |
| [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) | `Document` nema vlastiti sat: vrijeme dolazi iz `Now.utc()` (jedna ruta, DEF-DOC-09) |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi §Otvoreno |

## 12. Results

Stavka putuje tokom kao jedan objekt sa stabilnim identitetom, tipski zaključanim sadržajem i
poviješću izmjena koju pozivatelj ne može krivotvoriti. Pipeline i strategije rade s fasadom, pa
promjena tipa dokumenta ne dira nijedan od njih.

## 13. Acceptance Criteria

1. Identitet nastaje pri konstrukciji i ne postavlja se izvana (`BR-PTN-01`). ✅
2. Rezervirani ključevi povijesti nisu dostupni pozivatelju. ✅
3. Tip sadržaja se zaključava na prvoj dodjeli. ✅
4. Metapodaci se izvana vide samo za čitanje. ✅
5. Fasada delegira samo javna imena i griješi vlastitim imenom. ✅
6. Adapter se gradi s konfiguracijom fasade, ne sa zadanom. ✅
7. `Wattleflow` prethodi `Generic[Content]` u popisu baza (MRO). ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Dokument se može staviti u `set` ili koristiti kao ključ rječnika; hash slaže se s `__eq__` (isti tip i identifikator). ✅
10. `content` ne diže iznimku u stanju koje je klasa sama proglasila legalnim: dokument uvijek drži sadržaj do `clean()`, a `None` se ne prima. ✅
11. Podaci su u sadržaju, a metapodaci ih samo opisuju: generička klasa `Document` metapodatke samo opisuje (audit ključevi, `filename`). Odstupanje u `blackwattle` (zapisi u metapodacima) vodi se u `HLRQ-17` §6 t.8. ✅
12. Postoji jedna ruta do vremena, `Now.utc()`: `Document` nema vlastiti sat (`utc_time_stamp` je uklonjen), a audit ključevi `created_at` i `last_change_time` dolaze iz nje; nijedan kod ne traži vrijeme od dokumenta. ✅
13. Adapter i fasada odbijaju objekt koji nije `IAdaptee` prije svake bazne inicijalizacije (`TypeError`), pa odbijen objekt ne ostavlja ništa napola izgrađeno. ✅
14. Fasada delegira atribut adaptee-u jednim pristupom (svojstvo se izračuna jednom) i adaptee razrješava pri svakom promašaju, bez predmemorije. ✅
15. `FileDocument` je deklariran kao `Document[str]`, što izvršavanje već nameće (preduvjet 5). ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1 | pregled `__init__` | `self._identifier: str = str(uuid4())`; nema settera |
| 2 | pregled `update_metadata` | `if key in _AUDIT_KEYS: raise ValueError(...)` |
| 3 | pregled `update_content` | `_expected_type` postavljen jednom, potom `isinstance` provjera |
| 4 | pregled `metadata` propertyja | `MappingProxyType(self._metadata)` |
| 5 | pregled `DocumentFacade.__getattr__` | `_`-grana prva; obje poruke nose `self.__class__.__name__` |
| 6 | pregled `DocumentFacade.__init__` | `DocumentAdapter(adaptee, **kwargs)` |
| 7 | pregled popisa baza | `class Document(Wattleflow, IAdaptee, Generic[Content], ABC)` |
| 8 | pregled modula | `__all__` = 4 imena; uvozi `abc`, `datetime`, `typing`, `collections.abc`, `types`, `uuid` + `wattleflow.*` |
| 9 | `workflow/tests/test_document_hash.py` (7 testova) | prije ispravka 6 testova pada s `TypeError: unhashable type`; nakon njega 97/97 u `workflow/tests`, `ruff check` čist; mutacija `id(self)` ruši test jednakih dokumenata |
| 10 | `workflow/tests/test_document_content.py` (8 testova) | prije ispravka 3 testa padaju (konstrukcija s `None`, `update_content(None)`, nepromijenjeno stanje nakon odbijanja); nakon njega 105/105 u `workflow/tests`, `ruff check` čist |
| 12 | `workflow/tests/test_document_clock.py` (6 testova); `blackwattle/tests/documents/test_clock_route.py` (2 testa); izmjena 13 datoteka | prije izmjene pada 2 testa u workflowu (dokument još ima sat) i test `blackwattle` koji je nalazio pozive u 13 datoteka; zamijenjeno 19 poziva `x.utc_time_stamp()` s `Now.utc()` (2 u `document.py`, 4 u `rdf.py`, 13 u strategijama; uvoz dodan samo u `postgres.py`); nakon izmjene workflow 156/156, blackwattle 308 testova s istih 4 neovisnih padova (`tests/metrics`) i 34 grešaka učitavanja (okolina bez `PIL`, `docx`, `numpy`, `pyspark`). `rdf.py` i `graph.py` nije moguće uvesti bez `rdflib`, pa su provjereni samo lintom. Raniji kriterij o `@staticmethod` (DEF-DOC-03) time je nadomješten |
| 13 | `workflow/tests/test_document_facade.py` (7 testova) | prije ispravka pada test fasade (bazna inicijalizacija pozvana 1 put prije odbijanja, očekivano 0); nakon njega 118/118 u `workflow/tests`, `ruff check` čist; mutacija (stari redoslijed) ruši isti test |
| 14 | `workflow/tests/test_document_facade.py` (`DelegationTest`, 4 testa); mjerenje `bench_facade.py` | prije ispravka pada test (svojstvo izračunato 2 puta, očekivano 1); nakon njega 122/122 u `workflow/tests`, `ruff check` čist; mutacija (vraćen `hasattr`) ruši isti test. Cijena delegiranog pristupa ~200 ns naspram ~40 ns izravno, neizmijenjena |
| 15 | `blackwattle/tests/documents/test_file_document.py` (8 testova) | prije izmjene pada 2 testa deklaracije (`str \| Path` ≠ `str`); nakon nje prolazi svih 8; cijeli skup `blackwattle` uspoređen bez i s izmjenom: razlika je točno ta 2 testa (failures 6 → 4) |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda, `command grep`, i provjera pravila
`__eq__`/`__hash__` izvođenjem (`python3 -c`); kriterij — odjeljak 13; platforma — `workflow` radno
stablo, CPython 3.11 (Linux/WSL2). **Mjereno stablo:** `concrete/document.py`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-DOC-<nn>`.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/document.py`
- Testovi: `workflow/tests/test_document_clock.py`, `workflow/tests/test_document_content.py`, `workflow/tests/test_document_facade.py`, `workflow/tests/test_document_hash.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | DEF-DOC-07 premješten u `blackwattle` (`HLRQ-17` §6 t.8): problem je u `DocumentRecords`, ne u generičkoj klasi; kriterij 11 zadovoljen u `workflow`. Odjeljak 15 prazan: svi defekti `FRQ-DOC` zatvoreni ili premješteni. |
| v0.0.5 | 2026-10-04 | DEF-DOC-07: analiza koda (nositelj `DocumentRecords`, tri korisnika, uzrok u tipu `FileDocument`), poveznice na premještene dokumente, opcije; čeka odluku. |
| v0.0.5 | 2026-10-04 | DEF-DOC-09 zatvoren u `v0.0.1.23`: jedna ruta do vremena, `Now.utc()`; `Document.utc_time_stamp` uklonjen, 19 poziva zamijenjeno u 13 datoteka (u strategijama ih je bilo 13, ne 17 kako je ranije pisalo); kriterij 12 zamijenjen. DEF-DOC-10 zatvoren: `FileDocument` je `Document[str]`; kriterij 15. |
| v0.0.5 | 2026-10-04 | DEF-DOC-08 zatvoren: pravilo za `FileDocument` (referenca na datoteku + tekstualni `str`) potvrđeno nakon empirijske provjere 20 datoteka i zapisano kao preduvjet 5; ispravak deklaracije izdvojen u DEF-DOC-10. |
| v0.0.5 | 2026-10-04 | DEF-DOC-08: empirijska provjera uporaba `FileDocument`-a (20 datoteka, izvršavanje) i predloženo pravilo; čeka potvrdu. |
| v0.0.5 | 2026-10-03 | DEF-DOC-06 zatvoren u `v0.0.1.23`: `DocumentFacade.__getattr__` koristi jedan pristup umjesto `hasattr` + `getattr` (svojstvo se računalo dvaput); adaptee se i dalje razrješava pri svakom promašaju (ugovor `IAdapter` dopušta lijeno razrješavanje); kriterij 14 i 4 testa. |
| v0.0.5 | 2026-10-03 | DEF-DOC-05 zatvoren u `v0.0.1.23`: `DocumentFacade` provjerava `IAdaptee` prije `super().__init__`, kao adapter; kriterij 13 i 7 testova `test_document_facade.py`. |
| v0.0.5 | 2026-10-03 | DEF-DOC-03 zatvoren u `v0.0.1.23`: `utc_time_stamp` je `@staticmethod`; kriterij 12 i 6 testova `test_document_clock.py`. Dio o dvije rute izdvojen u DEF-DOC-09. |
| v0.0.5 | 2026-10-03 | Preduvjet 4: dokument ima sadržaj od inicijalizacije u `StrategyCreate`; dodan DEF-DOC-08 (prazan privremeni sadržaj u `FileDocument`). |
| v0.0.5 | 2026-10-03 | DEF-DOC-02 zatvoren u `v0.0.1.23`: `None` nije sadržaj, `update_content(None)` i konstrukcija s `None` dižu `ValueError`; kriterij 10 zadovoljen, 8 testova `test_document_content.py`. Time je zatvoren i DEF-DOC-04 (dokument ne može početi prazan, pa brava tipa uvijek vrijedi od konstrukcije). |
| v0.0.5 | 2026-10-03 | DEF-DOC-01 zatvoren u `v0.0.1.23`: `Document.__hash__` po (tip, identifikator), dosljedan s `__eq__`; kriterij 9 zadovoljen, 7 testova `test_document_hash.py`. |
| v0.0.5 | 2026-10-03 | Otvorene stavke u 15 preimenovane u defekte `DEF-DOC-<nn>`; reference na njih preusmjerene. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: `size` označen kao apstraktan, adapter drži `IAdaptee` (`_adaptee`), `DocumentFacade(adaptee)`, uvjeti u toku `update_content`. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu. Pravilo o sadržaju i metapodacima čeka dokumentiranu izmjenu (§11) |
