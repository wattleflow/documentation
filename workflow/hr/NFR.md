# Wattleflow nefunkcionalni zahtjevi (NFR registar)

## Uvod

**Verzija:** Draft v0.1  
**Izvorni jezik:** hrvatski  
**Pratioci:** [METHODOLOGY.md](METHODOLOGY.md), [PHILOSOPHY.md](PHILOSOPHY.md), [DOCTRINE.md](DOCTRINE.md)

Ovaj registar drži provedive, mjerljive nefunkcionalne zahtjeve Wattleflow
workflow ekosustava. Njegova uloga je prevesti arhitektonska i filozofska načela u
konkretne, provjerljive obveze.

---

## Svrha

Ovaj registar drži provedive, **mjerljive** nefunkcionalne zahtjeve Wattleflow Workflow ekosustava. 
Svaki NFR:

* nosi jedinstveni identifikator `NFR-<KATEGORIJA>-NN` (npr. `ORG` = organizacija /
  struktura);
* usidren je na karakteristiku kvalitete iz **ISO/IEC 25010** (a za podatke ISO/IEC
  25012);
* navodi strojno provjerljive **kriterije prihvaćanja** i metodu **verifikacije**;
* sljeduje se prema gore do načela koje ga opravdava (`METHODOLOGY.md` §9
  *Arhitektonska sljedivost*).

> **Status verifikacije.** Test-framework i CI pipeline još nisu odabrani (vidi
> `POLICY.md` otvorena pitanja). Do tada „strojno provjerljivo" znači **lint-skripta**
> nad import-grafom / gramatikom imena, pokretljiva na zahtjev; kad CI postoji, ista
> skripta postaje quality gate. Kriteriji su pisani za tu skriptu.
>
> **Izvedba:** `tools/wem_lint.py` (čita `tools/naming_registry.yaml` — registar
> kontroliranog vokabulara pod ADR upravljanjem). Pokretanje:
> `python tools/wem_lint.py` (izlazni kôd ≠ 0 na ERROR-prekršaj). Trenutno stanje
> nad zatečenim kodom: 13 ERROR + 16 WARN — to je worklist za migraciju imena (korak 3).

---

## Zajedničke definicije

Ove pojmove koristi svaki `NFR-ORG-*` zahtjev.

* **Domena** — *top-level funkcionalni paket*: `pipelines`, `drivers`, `processors`,
  `strategies`, `connections`, `documents`, `blackboards`.
* **Pomoćna (support/helper) klasa** — klasa koja nije dio javnog ugovora domene
  (nije `Pipeline*`, `Driver*`, `Processor*`, strategija itd.); postoji da posluži
  drugim klasama.
* **Domain-internal shared modul** — modul koji dijele dva ili više *pod-paketa
  jedne domene* (npr. OCR helper koji koriste `pipelines/pdf` i `pipelines/png`).
  Živi **unutar te domene** (npr. `pipelines/convertors`), a ne u globalnom
  `helpers/`.
* **Dijeljeni helper** — klasa koju koriste **dvije ili više različitih domena**;
  samo se takve promiču u globalni `helpers/`, organizirane po **sposobnosti**
  (`io`, `geometry`, `text`, `validation`), nikad po potrošačkom sloju.
* **Kanonski subjekt** — puni, neskraćeni naziv subjekta korišten i kao ime
  domenskog paketa i kao vodeći facet imena klase (`dataframe`, ne `dframe`;
  `text`, ne `txt`). Registar mapira svaku zatečenu kraticu na kanonski subjekt
  tijekom migracije.

> **Zero-trust napomena (CLAUDE.md §7.4).** Dijeljeni helper koji ovisi o
> third-party paketu (npr. OCR → `pytesseract`) **ne smije** u čisti core
> `helpers/`; pripada `wattleflow-processors` paketu i lazy se učitava. Promocija u
> „dijeljeni" poštuje zero-trust granicu.

> **Mjeriteljska oznaka `[M]` (mjerljivi kriterij).** Kriterij označen `[M]` iznosi
> **metriku**, ne boolean prag. Katalog ostaje kategoriziran **po domeni**; `[M]` samo
> razlikuje mjerne kriterije radi razvoja metričkih metoda. Za svaki `[M]` vrijedi povelja:
> (1) deklarira **tip skale** (nominalna/ordinalna/intervalna/omjerna) i **jedinicu**;
> dopuštene su samo statistike te skale i ne agregira se preko skala. (2) Objavljuje se kao
> **dijagnostika** (vrijednost + slijepe pjege), **nikad kao pass/fail gate**, dok nije
> validirana **out-of-sample** na vanjskom kriteriju (trošak promjene [h], gustoća defekata).
> (3) Računa se nad grafom koji uključuje **statičke + relativne + dinamičke (ClassLoader/YAML)**
> rubove; deklarira slijepe pjege (kao ORG-01 fan-in donja granica). (4) Doseg (propagation
> cost, blast radius) računa se nad **tranzitivnim zatvorenjem**, ne direktnim bridovima.
> (5) Promocija `[M]` metrike u gate traži **ADR** s validacijskim dosjeom.
> Osnova: reprezentacijska teorija mjerenja (Stevens; Krantz i dr.), Goodhart/Campbell,
> ISO/IEC 15939 (mjeriteljski proces).

---

## NFR-ORG-01 — Lokalnost ovisnosti pomoćnih klasa

**Izjava**

Pomoćna (helper) klasa mora boraviti u najužem modulskom opsegu koji obuhvaća sve
svoje potrošače.

* Helper koji koristi jedna domena ko-lociran je *unutar* te domene.
* Helper koji koriste dva ili više **pod-paketa jedne domene** promiče se u
  **domain-internal shared modul** te domene (npr. `pipelines/convertors`).
* Helper koji koriste dvije ili više **različitih domena** promiče se u dijeljeni
  `helpers/`.
* Dijeljeni helperi organiziraju se po **sposobnosti** (`io`, `geometry`, `text`,
  `validation`), nikad po potrošačkom sloju (`helpers/pipelines`,
  `helpers/drivers`), jer helper dijeljen kroz slojeve nema jednog vlasnika-potrošača.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija za smještaj |
|---|---|
| Common Closure Principle | Klase koje se mijenjaju zajedno pakiraju se zajedno; helper samo jedne domene mijenja se s tom domenom. |
| Common Reuse Principle | Klase korištene zajedno pakiraju se zajedno; grupiranje po potrošačkom sloju krši to kad helper služi više slojeva. |
| Stable Dependencies Principle | Dijeljeni core ne smije ovisiti o volatilnim, domenski-specifičnim konceptima. |
| Acyclic Dependencies Principle | Dijeljeni `helpers/` core nikad ne smije uvoziti iz domenskog paketa. |
| Information Hiding (Parnas) | Domenski-interni helper ostaje skriven iza granice domene, ne izložen kroz cijeli framework. |
| YAGNI / Occamova britva | `helpers/<sposobnost>` pod-hijerarhije uvode se tek pod dokazanim klasteriranjem, nikad špekulativno. |

Referenca standarda: ISO/IEC 25010 — Održivost (Modularnost, Ponovna iskoristivost, Analitičnost).

**Kriteriji prihvaćanja** *(strojno provjerljivi)*

1. Svaka klasa u dijeljenom `helpers/` ima import fan-in od **≥ 2 različite domene**.
   Dijeljeni helper koji uvozi točno jedna domena je prekršaj (pripada unutar te
   domene ili u njezin domain-internal shared modul).
2. Nijedna domenski-lokalna pomoćna klasa ne uvozi se izvan svojega domenskog paketa,
   osim kroz **javno sučelje** te domene (`__all__`).
3. Ne postoji **import-brid** iz `helpers/` prema bilo kojem domenskom paketu
   (acikličnost).
4. Imena dijeljenih helper modula označavaju **sposobnost**, ne potrošački sloj.

**Implementacijske napomene**

* Metrika u kriteriju 1 definirana je nad *različitim domenama*, ne pod-paketima:
  OCR helper koji dijele `pipelines/pdf` i `pipelines/png` ima domenski fan-in **1**
  (`pipelines`) i stoga pripada u `pipelines/convertors`, **ne** u globalni
  `helpers/`.
* Postojeći jedno-potrošački moduli već u `helpers/` su **tolerirani** i migriraju
  inkrementalno; lint ih prijavljuje kao upozorenja dok se ne presele.

**Verifikacija**

Automatizirana analiza import-grafa kao lint (kasnije CI/CD quality gate); build pada
na bilo kojem kršenju kriterija 1–4.

---

## NFR-ORG-02 — Nomenklatura klasa

**Izjava**

Ime pipeline klase mora biti složeno od kontroliranih, ortogonalnih faceta u fiksnoj
gramatici, tako da ime kodira ugovor ulaz→izlaz, a ne unutarnji mehanizam.

* **Gramatika:** `Pipeline + <Subject> + <Operation | ToTarget> + [Qualifier]`
* **Subject** je vodeći facet (format, izvor ili domena), odabran da maksimizira
  kolokaciju: svi artefakti istog subjekta sortiraju se zajedno u imenskom prostoru
  i u autocompleteu. **Subject facet jednak je domenskom paketu koji sadrži klasu**
  (njegov kanonski subjekt), uspoređeno case-insensitive — pa
  `pipelines/pdf/PipelinePDFExtractText` usklađuje ime s putanjom. Znanstvena osnova
  za *Subject-first* redoslijed dana je u **Dodatku A**.
* **Operation** je iz zatvorenog vokabulara glagola (`Extract, Redact, Clean, Repair,
  Translate, Write`); konvertori izražavaju odnos izvor→cilj kroz veznik `To`
  (npr. `CSVToSheets`).
* **Qualifier** nosi engine, jezik ili varijantu (npr. `Spacy, Stanza, Flair, En, Hr`).
* Univerzalni glagol `transform()` **nije** kodiran u imenu: token predvidljiv za
  svaku klasu nosi nula diskriminirajuće informacije.
* Marker `Pipeline` rezerviran je isključivo za klase koje implementiraju javni
  pipeline ugovor; pomoćne klase ne smiju ga nositi.
* Generičke uloga-imenice (`Manager, Helper, Utility, Piece, Sheet, Parser`)
  zabranjene su kao **samostalna** imena; ista imenica dopuštena je kao
  *kvalificirani* facet (`DriverS3UriParser`, `CSVToSheets`). Ime mora označavati
  sloj, domenu i odgovornost.

**Dva režima imenovanja.** Gornja gramatika uređuje **pipeline** klase (one koje
nasljeđuju pipeline bazu). **Pomoćne** klase uređuje samo: bez `Pipeline` prefiksa;
bez samostalne generičke uloga-imenice; domenski kvalificirano
(`SheetNestingPlacement`, ne `Placement`; `CuttingSheet`, ne `Sheet`).

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija za imenovanje |
|---|---|
| Principle of Least Astonishment | Fiksna gramatika čini ime predvidljivim iz faceta, a facete obnovljivima iz imena. |
| Faceted Classification (Ranganathan) | Ortogonalne osi složene na mjestu uporabe izbjegavaju kombinatornu eksploziju jedne enumerativne hijerarhije. |
| Teorija informacije (Shannon) | Imena maksimiziraju uzajamnu informaciju s diskriminirajućim facetima; konstantni tokeni (`Transform`) uklanjaju se kao šum. |
| Information Hiding (Parnas) | Ime izražava vanjski ugovor, ne omotanu implementacijsku klasu. |
| Single fundamentum divisionis | Jedna osnova klasifikacije po osi; miješanje formata, operacije i domene na istoj razini je zabranjeno. |
| DRY | Svaki facet ima točno jedan kontrolirani termin; sinonimi (`Cleanup`/`Correction`/`Fix`, `Entities`/`EntityRecognition`) se sažimaju. Ponavljanje subjekta i u putanji i u imenu je prihvaćen, svjestan trade-off DRY-a za samoopisivost i kolokaciju (Dodatak A §4). |

Referenca standarda: ISO/IEC 25010 — Održivost (Analizabilnost); Python PEP 8 —
konvencije imenovanja (uključujući akronime velikim slovima).

**Kriteriji prihvaćanja** *(strojno provjerljivi)*

1. Svaka klasa koja nasljeđuje pipeline bazu odgovara gramatici
   `Pipeline<Subject>(<Operation>|To<Target>)(<Qualifier>)?`, gdje se svaki facet-token
   razrješava prema registru kontroliranog vokabulara; svaki drugi oblik je prekršaj.
2. Prefiks `Pipeline` pojavljuje se **samo** na klasama koje implementiraju javni
   pipeline ugovor; njegova prisutnost na pomoćnoj klasi je prekršaj.
3. Svaki facet-token pripada svojem registriranom kontroliranom vokabularu;
   neregistrirani tokeni ruše build dok se vokabular ne proširi kroz ADR.
4. Casing akronima slijedi **PEP 8** — sva slova kratice velikim slovom (`PDF`, `PNG`,
   `CSV`, `HTML`, `RDF`); miješani casing (`Pdf`/`PDF`) je prekršaj. Jezični kodovi
   (`En`, `Hr`) slijede registar.
5. Subject facet jednak je kanonskom subjektu domenskog paketa koji sadrži klasu
   (case-insensitive); nepodudaranje je prekršaj.
6. Nijedno ime klase ne sastoji se isključivo od zabranjene generičke uloga-imenice.

**Implementacijske napomene**

* Kriteriji 1/3/5 zahtijevaju **registar kontroliranog vokabulara** (strojno čitljiva
  datoteka koja navodi `domains`, `subjects` = paketi, `operations`, `acronyms` +
  casing, `qualifiers`), pod ADR upravljanjem. Izgradnja tog registra je preduvjet
  provedbe.
* Sažimanje sinonima za trenutni katalog: `Redaction→Redact`,
  `Cleanup/Correction→Clean`, `Fix→Repair`.
* Migracija je **hard rename** (bez dugotrajnih aliasa); primjeri i importi ažuriraju
  se u istom prolazu. Reprezentativna preimenovanja:
  `PipelinePDFExtractText → PipelinePDFExtractText`,
  `PipelinePDFRedactText → PipelinePDFRedact`,
  `PipelinePNGRedaction → PipelinePNGRedact`,
  `PipelineFixCorruptedText → PipelineTextRepair`,
  `PipelineDataframeCleanup → PipelineDataFrameClean`,
  `PipelineCsvToSheets → PipelineCSVToSheets`.

**Verifikacija**

Automatizirano lintanje gramatike imena i vokabulara kao lint (kasnije CI/CD quality
gate); build pada na bilo kojem kršenju kriterija 1–6.

---

## NFR-ORG-03 — Nomenklatura tipskih varijabli (generičkih parametara)

> **Status:** Prijedlog (Draft). Nadovezuje se na NFR-ORG-02 (Nomenklatura klasa)
> i dijeli isti registar kontroliranog vokabulara (`tools/naming_registry.yaml`)
> i istu lint izvedbu (`tools/wem_lint.py`).

**Izjava**

Ime tipske varijable (`TypeVar`) mora imenovati **semantičku ulogu** koju tip igra
u generičkom ugovoru, odabranu iz kontroliranog vokabulara uloga, a ne mehanizam
(činjenicu da je riječ o tipu). Ime kodira *što tip predstavlja u ugovoru*, ne
*da je tip*.

* **Gramatika:** `<RoleNoun>` — jedna imenica-uloga iz registra (`Key`, `Value`,
  `Message`, `Destination`, `Context`, `Result`, `State`, `Action`, `Vertex`,
  `Edge`, `Input`, `Output`, `Element`, …). Višerječne uloge u PascalCase
  (`KeyType` je prekršaj; `Key` je ispravan).
* **Bez mehanizam-sufiksa.** Sufiksi `T`, `Type`, `_t` zabranjeni su jer nose nula
  diskriminirajuće informacije — svaka je tipska varijabla tip (`YieldT` →
  `Output`, `SendT` → `Input`).
* **Goli `T` rezerviran** je isključivo za **jedan neograničen** „bilo koji
  element" parametar generika bez semantičke specijalizacije (npr. spremnik koji
  je doista parametričan po jednom proizvoljnom tipu). Čim parametar nosi ulogu,
  imenuje se ulogom.
* **Višeparametarski generik** (≥ 2 parametra) imenuje svaki parametar **distinktnom
  imenicom-ulogom**; sekvence jednoslova `T, U, V` su prekršaj
  (`Generic[Key, Value]`, ne `Generic[T, U]`).
* **Akronimi** velikim slovima prema PEP 8 (`URI`, `RDF`), kao i u NFR-ORG-02.
* **Varijanca se ne kodira u imenu.** Kovarijantnost/kontravarijantnost deklarira
  se kroz `TypeVar(..., covariant=True | contravariant=True)`, nikad sufiksom
  `_co`/`_contra` u imenu (Principle of Least Astonishment: ime nosi ulogu, ne
  teorijsko svojstvo varijance).

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija za imenovanje tipskih varijabli |
|---|---|
| Teorija informacije (Shannon) | Sufiks `T`/`Type` je konstantni token na svakoj tipskoj varijabli — uzajamna informacija s identitetom parametra je nula; uklanja se kao šum (isti argument kao univerzalni `transform()` u NFR-ORG-02). |
| Principle of Least Astonishment | Ime imenuje ulogu (`Input`, `Output`, `Result`), pa je ugovor čitljiv bez otvaranja definicije `TypeVar`-a. |
| Single fundamentum divisionis | Ime klasificira po jednoj osnovi — semantičkoj ulozi tipa — a ne miješa ulogu s mehanizmom (da je riječ o tipu). |
| DRY | Jedan kanonski termin po ulozi; sinonimi (`Output`/`Yielded`, `Input`/`Sent`, `Result`/`Return`) sažimaju se na jedan registrirani termin. |
| Information Hiding (Parnas) | Ime izražava ulogu u javnom ugovoru generika, ne internu implementaciju ili podrijetlo (npr. „yield" je mehanizam korutine, „output" je uloga). |
| Faceted Classification (Ranganathan) | Uloge tipova su ortogonalne osi (ključ ↔ vrijednost, ulaz ↔ izlaz); svaka os nosi vlastiti kontrolirani termin. |

Referenca standarda: ISO/IEC 25010 — Održivost (Analizabilnost); Python PEP 8
(konvencije imenovanja, akronimi); PEP 484 (`TypeVar`, varijanca).

**Kriteriji prihvaćanja** *(strojno provjerljivi)*

1. Svako ime `TypeVar`-a razrješava se prema registriranom vokabularu uloga
   (`type_vars` u `naming_registry.yaml`); neregistrirano ime ruši build dok se
   vokabular ne proširi kroz DR.
2. Nijedno ime `TypeVar`-a ne nosi mehanizam-sufiks (`T`, `Type`, `_t`); prisutnost
   sufiksa je prekršaj. Iznimka: goli identifikator `T`.
3. Goli jednoslov (`T`, `U`, `V`, …) dopušten je samo za **jedan** neograničen
   parametar unutar jednog generika; pojava dvaju ili više jednoslovnih parametara
   u istom `Generic[…]` je prekršaj.
4. Casing akronima u imenu slijedi PEP 8 (sva slova velikim); miješani casing je
   prekršaj.
5. Ime ne kodira varijancu (`_co`/`_contra`); varijanca se izražava isključivo
   argumentima `TypeVar`-a.
6. Jedan kanonski termin po ulozi; registrirani sinonim umjesto kanonskog termina
   je prekršaj (analogno sažimanju sinonima u NFR-ORG-02).

**Implementacijske napomene**

* Registar dobiva ključ `type_vars:` (popis kanonskih imenica-uloga + dopušteni
  sinonimi za sažimanje), dijeleći `acronyms` blok s NFR-ORG-02. Izgradnja tog
  popisa je preduvjet provedbe.
* Sažimanje sinonima za trenutni katalog: `Yielded→Output`, `Sent→Input`,
  `Return/Returned→Result`, `Elem→Element`.
* Reprezentativna preimenovanja (worklist, hard rename):
  * `ICoroutine[YieldT, SendT, ReturnT]` → `ICoroutine[Output, Input, Result]`.
    Napomena: `ReturnT`/`Result` trenutno nije upotrijebljen u potpisima
    `ICoroutine` — ili se izostavlja (`Generic[Output, Input]`), ili zadržava uz
    eksplicitnu uporabu u povratnom tipu završetka.
  * `WattleType` (iterator element) → odluka registra: ili **tolerirati** kao
    brendirani „element" termin, ili rename u kanonski `Element`. Lint ga do
    odluke prijavljuje kao WARN (analogno politici tolerancije u NFR-ORG-01).
* Postojeća golema imena koja su već usklađena (`Context`, `Result`, `State`,
  `Key`, `Value`, `Message`, `Destination`, `Vertex`, `Edge`, `Action`) ulaze u
  registar kao kanonska bez izmjene.

**Verifikacija**

Automatizirano lintanje imena tipskih varijabli kao lint (kasnije CI/CD quality
gate). Provjera je AST-bazirana: pronalazi `TypeVar(...)` dodjele
(`ast.Assign` čija je vrijednost `ast.Call` na ime `TypeVar`), izvlači ime cilja
i deklarirane `covariant`/`contravariant` argumente, te ih provjerava prema
kriterijima 1–6. Build pada na bilo kojem kršenju.

---

## NFR-ORG-04 — Cross-cutting sposobnost kao helper (ne domenski primitiv)

> **Status:** Prijedlog (Draft). Uveden kroz `docs/adr/helpers/DR-WFL-001`. Dijeli
> registar (`tools/naming_registry.yaml`) i lint (`tools/wem_lint.py`) s NFR-ORG-01/02.

**Izjava**

Cross-cutting odgovornost koja služi više potrošača i ne pripada nijednom domenskom
primitivu — npr. **rutiranje** artefakta na odredište (pod-direktorij / topic / indeks
/ tablica) — modelira se kao **pozivljiva helper-sposobnost**, a ne kao specijalizacija
domenskog primitiva (`Strategy`, `Pipeline`, `Driver`, `Processor`). Smješta se i
imenuje **po sposobnosti** (NFR-ORG-01 t.4), a klasna imena slijede NFR-ORG-02.

* Sposobnost je **pozivljiva** iz bilo kojeg potrošača (strategija, procesor, driver);
  `Strategy` to nije — poziva ga isključivo njegov kontekst (Repository/Blackboard) pa
  ga susjedni `Strategy` ne može koristiti.
* Per-transport varijante su polimorfne implementacije iza **stabilne apstrakcije**
  (DIP), ne GoF Strategy sudionici.
* Neutralni podatak sposobnosti (npr. `route` labela) transport-agnostičan je i ne
  posuđuje transport-specifičan pojam za generičku ulogu (npr. `partition` — Kafka/Spark
  fizička particija).

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Ontologija (`PHILOSOPHY.md`) | `Strategy`/`Pipeline`/… su rezervirani primitivi; sposobnost se ne maskira u njih |
| GoF Strategy | poziva ga kontekst, nije komponibilan u susjedne strategije → sposobnost mora biti pozivljiva |
| GRASP — Pure Fabrication, High Cohesion, Low Coupling | cross-cutting odgovornost bez matične domene → fabricirani kohezivni helper |
| SRP / Separation of Concerns (Parnas, Dijkstra) | „gdje/kako se adresira" odvojeno od „kako se perzistira/transformira" |
| DIP / Stable Abstractions | apstraktni ugovor + per-transport konkreti |
| Open–Closed | novi transport = nova implementacija bez diranja potrošača |
| Information Hiding (Parnas) | neutralni podatak sposobnosti skriva transport-specifičnosti |

Referenca standarda: ISO/IEC 25010 — Održivost (Modularnost, Ponovna iskoristivost,
Analizabilnost).

**Kriteriji prihvaćanja** *(dijelom strojno provjerljivi)*

1. Klasa/funkcija koja implementira cross-cutting sposobnost **ne nasljeđuje** domenski
   primitiv (`Strategy`, `Pipeline`, `Driver`, `Processor`, `Blackboard`, `Repository`);
   izloženo sučelje je pozivljivo (metoda/funkcija), ne delegat konteksta.
2. Sposobnost-modul imenovan je po **sposobnosti** (`routing`), ne po potrošačkom sloju
   (naslijeđeno iz NFR-ORG-01 kriterij 4).
3. Klasna imena poštuju NFR-ORG-02 — kvalificirano, uloga-eksplicitno; generičke
   uloga-imenice (`Router`, `Resolver`, `Generator`, `Scanner`) dopuštene su **samo
   kvalificirane**.
4. Transport-neutralni podatak sposobnosti ne koristi transport-specifičan termin za
   generičku ulogu (npr. logička ruta nije `partition`).

**Implementacijske napomene**

* Kriteriji 1/4 su trenutno **pregledom-vođeni** (kao ORG-01 #2/#4); kriterij 3 dijeli
  `prohibited_standalone` provjeru s NFR-ORG-02 (`wem_lint`). Puna automatizacija
  1 (baza nasljeđivanja) i 4 (vokabular termina) planira se.
* Registar dobiva `capabilities:` popis (priznate sposobnosti; trenutno `routing`) i
  prošireni `prohibited_standalone`.
* Prvi nositelji: `helpers/routing.py` (`RoutingRule`, `PatternSpec`, `DestinationRouter`,
  `LocalStorageDestinationRouter`, `route_label`/`route_target`) i `helpers/files.py`
  (`FileSourceScanner`). Vidi `docs/adr/helpers/DR-WFL-001`.

**Verifikacija**

Kombinacija lint provjere (kriterij 3 preko `prohibited_standalone`; ORG-01 acikličnost
za smještaj) i arhitektonskog pregleda (kriteriji 1/4) dok se ne automatiziraju. Build
pada na kršenju automatiziranih dijelova.

---

## NFR-ORG-05 — Enkapsulacija samoreferencirajućih pomoćnih metoda (`cls` nad tvrdo kodiranim imenom klase)

> **Status:** Prijedlog (Draft, 2026-07-09). Statička (AST) provjera; dijeli lint
> (`tools/wem_lint.py`) i `prohibited_standalone` s NFR-ORG-02. Ne uvodi runtime.

**Izjava**

Srodne **bezstanjne** pomoćne funkcije koje dijeli **više klasa/strategija** omataju se u
**kvalificiranu klasu** (ne slobodne module-funkcije, ne generički `Helper` — NFR-ORG-02)
koja drži pripadne **konstante kao class-atribute** i **metode**. Unutar takve klase:

* metoda koja u tijelu referira **vlastitu klasu** (class-konstanta ili srodna metoda)
  ide kroz `cls` i deklarira se `@classmethod`;
* metoda koja **ne** dira nijedan član svoje klase (čista funkcija argumenata) ostaje
  `@staticmethod`.

Tvrdo kodirano ime vlastite klase (`ClassName.CONST`, `ClassName.sibling()`) unutar
`@staticmethod` je prekršaj.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Information Hiding (Parnas) | konstante i srodne metode skrivene iza kvalificiranog imenskog prostora klase, ne raspršene po modulu |
| GRASP — High Cohesion, Pure Fabrication | srodni bezstanjni helperi + njihove konstante čine jednu kohezivnu jedinicu |
| Open–Closed / Liskov | `cls` razrješava člana kroz podklasu; tvrdo kodirano ime lomi override |
| Least Astonishment | izbor dekoratora **izražava namjeru**: `static` = čisto, `classmethod` = koristi stanje klase |
| DRY / Once and Only Once | ime klase se ne ponavlja u tijelu → preimenovanje je jednomjesno |

Referenca standarda: ISO/IEC 25010 — Održivost (Modifikabilnost, Modularnost, Analizabilnost).

**Kriteriji prihvaćanja** *(strojno provjerljivi)*

1. Nijedan `@staticmethod` u tijelu ne referira svoju obuhvatnu klasu tvrdo kodiranim
   imenom (`<EnclosingClass>.<član>` gdje je `<član>` class-atribut ili metoda te klase);
   takva metoda mora biti `@classmethod` i koristiti `cls`.
2. `@staticmethod` je dopušten **samo** kad tijelo ne referira nijedan član obuhvatne
   klase (čista funkcija argumenata).
3. Konstante-članovi ne nose redundantan prefiks imena klase (`Layout.SUBDIR`, ne
   `Layout.LAYOUT_SUBDIR`).

**Iznimka (kriterij 1).** Metoda čiji potpis već veže parametar imenom `cls` (domenski
tip, npr. `Attribute.mandatory(caller, name, cls: type, …)`, korišten i keywordom `cls=`)
**ne** konvertira se u `@classmethod` (duplo ime) dok se parametar ne preimenuje
(`cls`→`kind`/`expected`) — zasebna API odluka. Do tada ostaje `@staticmethod` i lint to
priznaje kao odgođeno (analogno toleranciji u ORG-01), ne kao prekršaj.

**Implementacijske napomene**

* Referentni obrazac: `MailAttachmentLayout` u `strategies/documents/mail.py` — `resolve`/
  `write` (`@classmethod`, `cls.FLAT`/`cls.label`) uz čistu `label` (`@staticmethod`).
* Provedba je legacy-migracija „dio po dio" (kao ORG-01/02 worklist). AST-popis
  zatečenih kandidata (2026-07-09): **28** mjesta — `constants/filetype` (5),
  `helpers/attribute` (5), `helpers/converters` (3), `helpers/files` (2),
  `helpers/config` (1), `drivers/{postgres,solr,spark}` (6), `pipelines/eml` (2),
  `cad/` (3, `gcode`+`gcode_slicer`). Kriterij 1/2 je kandidat za `wem_lint` proširenje.

**Verifikacija**

Statička AST-provjera (kriterij 1/2) + `prohibited_standalone` (kriterij 3 dijeli s
ORG-02) dok se ne automatiziraju u CI-u. Do tada lint na zahtjev; build pada na
ERROR-prekršaj kad se kriterij automatizira.

---

## Sigurnosni zahtjevi — razrada zero-trust arhitekture (NFR-SEC-*)

> Zero-trust je temeljni koncept, razrađen ovdje na **ograničene i/ili mjerljive** dijelove.
> SEC iznosi sigurnosne **zahtjeve i mjere**; strukturni ORG-* i pakiranje su njihov mehanizam.
> **Povijest:** nekadašnji NFR-ORG-06 (Lokalnost distribucije i validacija opskrbnog lanca)
> dekomponiran je 2026-07-22 u SEC obitelj — njegov supply-chain/lokalnost jezgro je **SEC-03**
> (DR-WFL-002/07 reference vrijede preko SEC-03); blast i napadna površina izdvojeni su u
> SEC-01/02. Mjerni kriteriji nose oznaku `[M]` (vidi Zajedničke definicije).

---

## NFR-SEC-01 — Zadržavanje blast-radiusa (kompartmentalizacija)

> **Status:** Prijedlog (Draft, 2026-07-22). Razrada zero-trust u mjerljivi dio; oslanja se na
> strukturni graf (ORG-01) i mjeriteljsku povelju `[M]`.

**Izjava**

Očekivani gubitak od kompromitacije je `E[L] = Σ P(kompromitacija_i) · Blast(i) · V`, gdje je
`Blast(i)` dosežljivost iz `i` u grafu ovisnosti (i povlastica). Arhitektura mora **ograničiti
blast radius** po komponenti: dekompozicija i least-privilege smanjuju `Blast`, ali svako novo
sučelje je ulazna točka — optimum je **unutarnji**, ne rubni (zrcalo MDL-a, poveznica SEC-02).

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Least privilege / kompartmentalizacija (Saltzer–Schroeder) | manji `Blast` po kompromitaciji |
| Design rules / Net Option Value (Baldwin–Clark) | modularnost je **množitelj u računu rizika**, ne estetika |
| Korelirani proboji (log4j) | `Blast` nad **tranzitivnim** ovisnostima, uklj. dijeljene third-party |
| Unutarnji optimum (zrcalo MDL / Manadhata–Wing) | prekomjerna dekompozicija ↑ broj sučelja ↑ napadna površina |

Referenca standarda: ISO/IEC 25010 — Sigurnost (Povjerljivost, Integritet).

**Kriteriji prihvaćanja**

1. **[M]** `Blast(i)` = udio sustava dosežljiv iz `i` nad **tranzitivnim zatvorenjem** grafa
   ovisnosti (uklj. dijeljene ovisnosti). Omjerna skala; dijagnostika.
2. **[M]** Hub s `Blast` iznad **kalibriranog** praga (na vanjskom kriteriju) je kandidat za
   kompartmentalizaciju; prag nije gate dok nije validiran (povelja `[M]`).
3. Nijedan clean-core hub nema **third-party u tranzitivnom `Blast`-u** (poveznica SEC-03).
4. `Blast` uključuje **dinamičke rubove** (ClassLoader/YAML), ne samo statičke importe.

**Verifikacija**

`wem_lint` metrika (dijagnostika); WARN kad hub prijeđe kalibrirani prag. Gate tek DR-om.

---

## NFR-SEC-02 — Minimalnost napadne površine

> **Status:** Prijedlog (Draft, 2026-07-22). Razrada zero-trust; oslanja se na povelju `[M]`.

**Izjava**

Napadna površina komponente mjeri se preko **metoda, kanala i podatkovnih stavki** izloženih
preko granice povjerenja (Manadhata–Wing). Javno sučelje mora biti **minimalno** — svaka
izložena metoda/kanal/stavka je trošak — uz isti unutarnji optimum kao SEC-01: premala
dekompozicija ↑ blast, prevelika ↑ broj sučelja.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Attack surface metric (Manadhata–Wing 2011) | izloženost = f(metode, kanali, podaci) |
| Economy of mechanism (Saltzer–Schroeder) | manje sučelje = manje za napasti i verificirati |
| Zrcalo MDL (Rissanen; Cilibrasi–Vitányi) | `L(moduli)+L(sučelja)`: optimum minimizira ukupnu duljinu opisa |
| Information hiding (Parnas) | interne specifičnosti ne izlaze kroz granicu |

Referenca standarda: ISO/IEC 25010 — Sigurnost; Održivost (Modularnost).

**Kriteriji prihvaćanja**

1. **[M]** Napadna površina = ponderirani zbroj izloženih metoda/kanala/podatkovnih stavki
   preko granice (povjerenja/distribucije). Skala: intervalna (ponderi); dijagnostika.
2. Javno sučelje modula deklarira se eksplicitno (`__all__`); neizloženo ostaje privatno.
3. **[M]** Rast napadne površine **bez** pada `E[L]` je regresija (Pareto-dominirano).

**Verifikacija**

`wem_lint` metrika nad AST-om (broj javnih metoda/ulaza) + manifest granice; dijagnostika.

---

## NFR-SEC-03 — Supply-chain trust i lokalnost distribucije

> **Status:** Prijedlog (Draft, 2026-07-09; premješteno iz NFR-ORG-06 i prošireno 2026-07-22).
> Razrada zero-trust: **distribucijska granica JE sigurnosni kompartment**. Nasljeđuje
> `adr/DR-WFL-002-distribution-locality` i `DR-WFL-003`. Proširuje NFR-ORG-01 (Dependency
> Locality) preko granice distribucije. Dijeli lint (`tools/wem_lint.py`, `core_libraries`
> allowlist) s ORG-01.

**Izjava**

Matična distribucija modula određena je njegovim **import-closureom, ne ulogom**. Modul
pripada clean core distribuciji (`wattleflow`, `wattleflow-workflow`) **samo ako** mu cijeli
tranzitivni closure staje u dopušteni tier te distribucije (stdlib + eksplicitni core
allowlist). Modul koji — eager ili **lazy** (§7.4) — referira third-party paket pripada
ne-core distribuciji (`wattleflow-processors`, `wattleflow-cad`). Svaka distribucija
deklarira **manifest** podstabala koja posjeduje; nijedno `wattleflow.*` podstablo nema
dva vlasnika.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Zero-trust (Supply-chain, §7.1) | clean core nosi nula third-party koda → kompromitirana biblioteka ne dopire do core korisnika |
| Dependency Inversion / Stable Abstractions | čisto sučelje u core, third-party adapter u ne-core distribuciji |
| Information Hiding (Parnas) | third-party specifičnosti skrivene iza core apstrakcije |
| Occam / DRY | integritet opskrbnog lanca oslonjen na standarde (SBOM/lock/atestacije), ne na vlastiti mehanizam |
| PEP 420 / PEP 660 | namespace paketi + editable installovi zamjenjuju symlinkove bez gubitka live-dev |

Referenca standarda: ISO/IEC 25010 — Održivost (Modularnost), Sigurnost (Integritet);
NIST (OSCAL) za compliance evidenciju SBOM-a.

**Kriteriji prihvaćanja** *(dijelom strojno provjerljivi)*

1. **Locality (a).** Svaki modul u distribuciji ima closure ⊆ tier te distribucije;
   modul s third-party referencom nije u clean core distribuciji. (Strojno: `wem_lint`
   + per-distribucija manifest.)
   * **Iznimka — čuvana opcionalna ovisnost (`DR-WFL-003`, 2026-07-15).** Referenca
     zaštićena `try/except ImportError` granom čiji je fallback **funkcionalno potpun** i
     unutar tiera distribucije ne izmješta modul: efektivni closure je tier. Kriterij je
     **potpunost fallbacka**, ne postojanje `try/except`-a — grana koja diže iznimku,
     vraća `None` ili degradira funkciju **nije** čuvana ovisnost. Verifikacija je
     **test maskiranja** (maskiraj paket → modul se mora uvesti i raditi), ne pregled
     koda. Nositelji: `helpers/config.py`, `helpers/config_adapter.py`,
     `helpers/config_validator.py` (→ `yaml`/`jsonschema`, fallback `helpers/yaml.py`).
2. **Jedinstveno vlasništvo.** Svako `wattleflow.*` podstablo pakira točno jedna
   distribucija (nema import-shadowinga u namespace mergeu). (Strojno: usporedba manifesta.)
3. **Bez symlinkova u pakiranju.** Distribucija ne pakira tuđi `__init__.py`; dev koristi
   editable installove, ne symlinkove. (Pregled + `packages.find include` allowlist.)
4. **Supply-chain (b).** Svaka ne-core distribucija isporučuje hash-pinned lock + SBOM;
   third-party koji krši core politiku (paket/licenca/known-bad) diže upozorenje.
   (Strojno: SBOM validator vođen core politikom.)
5. **Self-integritet vlastitih modula.** Integritet isporučenih wattleflow modula
   oslanja se na **wheel `RECORD`** (per-file `sha256`, standardno prisutan u svakoj
   izgrađenoj distribuciji) — verificira se, ne re-implementira. „Distribucijski digest"
   (Merkle-korijen) = hash `RECORD`-a / SBOM korijen; smislen i stabilan **samo za
   izgrađene artefakte**. (Strojno: verifikacija `RECORD`-a; za dev/editable stabla vidi
   napomenu.)
6. **[M] Korelirani proboj.** `Blast` (SEC-01) računa se nad **tranzitivnim** zatvorenjem
   ovisnosti; moduli koji dijele istu third-party ovisnost imaju **korelirane** proboje —
   nominalna izolacija bez izolacije lanca opskrbe je iluzija (log4j). (Strojno: udio modula
   s dijeljenom ovisnošću iz SBOM-a; omjerna skala, dijagnostika.)

**Implementacijske napomene**

* Kriterij 1/2 nadograđuju postojeći `wem_lint` (`core_libraries` allowlist već računa tier;
  nalazi 2026-06-29: 2 procurivanja u core — `concrete/logger.py`→pandas **[RIJEŠENO
  2026-07-22: duck-typing `hasattr(shape, columns)`, workflow clean-core sad 0 pandas rubova]**;
  `mappers/schema_yaml_json.py`→pandas/yaml/jsonschema (preseljen u `wattleflow-processors`)).
* **Alat mora `foreign_imports` tretirati kao SEC-03 nalaz (flag), ne kao exclusion.** Trenutno
  `wem_lint` third-party rub koristi da modul *isključi* iz ORG-01/02/03 opsega — isti podatak
  ORG-06/SEC-03 traži *prijaviti*. Razrješava se redizajnom (jedan graf, `Rule` čita rub za
  supply-chain, `Metric` ga broji za napadnu površinu).
* Kriterij 3 je već dokazan u `wattleflow-cad` (`namespaces=true` + `include`); generalizira
  se na sve distribucije.
* Kriterij 4 ne re-implementira integritet — `pip --require-hashes`/`uv.lock` + CycloneDX/SPDX
  SBOM + (opcionalno) Sigstore/PEP 740. `helpers/digest.py` (`FileDigest`) ostaje za integritet
  **vlastitih** modula, ne za third-party vetting.
* Kriterij 5: `RECORD` **editable** installa sadrži samo `.pth`, ne izvore — u dev stablu je
  prazan. Ondje `wem_lint` digest-scan (`FileDigest`) preuzima self-integritet i **istodobno
  služi kao detekcija namespace-sjenčanja/kolizije** (kriterij 2). U dev stablu distribucijski
  digest nije identitetski token (izvor se stalno mijenja) — služi samo za drift/shadowing lint.

**Verifikacija**

`wem_lint` distribution-manifest gate (1/2) + pregled pakiranja (3) + SBOM validator (4)
+ `RECORD` verifikacija za izgrađene artefakte / `wem_lint` digest-scan za dev stabla (5).
Uvodi se WARN→ERROR (tolerancija kao ORG-01) dok migracija (DR-WFL-002 §5) traje.

---

## NFR-SEC-04 — Radna točka detekcije i psihološka prihvatljivost

> **Status:** Prijedlog (Draft, 2026-07-22, skica). Pretežno pregledom-vođen; formalizira se
> kasnije (ljudski faktori, telemetrija kontrola).

**Izjava**

Svaka sigurnosna kontrola je **klasifikator**: lažno pozitivno = trošak udobnosti (blokiran
legitiman korisnik), lažno negativno = trošak proboja. Kontrola bira **radnu točku** (ROC), ne
„strože"; a mehanizam koji se zaobilazi ne štiti — **psihološka prihvatljivost** (Saltzer–Schroeder,
načelo P8) izvorno je sigurnosno načelo, ne naknadni kompromis.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Teorija detekcije signala | prag = `P(signal)/P(šum) · C_FP/C_FN`; pomak praga je kretanje po ROC-u, ne poboljšanje |
| Paradoks lažno pozitivnog (base rate) | uz nisku prevalenciju napada i dobar klasifikator daje pretežno lažne uzbune |
| Compliance budget (Beautement–Sasse–Wonham 2008) | prekoračenje truda → **zaobilaženje**, ne otpor |
| Psihološka prihvatljivost (Saltzer–Schroeder; Adams–Sasse 1999) | korisnik nije protivnik; zaobilaženje je signal loše radne točke |

**Kriteriji prihvaćanja** *(pretežno pregledom)*

1. Prag se postavlja iz omjera troškova i **base-rate** prevalencije, ne ad hoc „strože".
2. **[M]** Trenje se mjeri: **stopa zaobilaženja** (primarni KPI — ishod, ne stav), koraci/vrijeme
   po zadatku. Omjerna skala; dijagnostika.
3. Poboljšanje = pomak **ROC krivulje** (bolji senzor/kontekst), ne pomak praga po njoj.

**Verifikacija**

Telemetrija kontrola (FP/FN, stopa zaobilaženja) + pregled radne točke. Nije CI gate.

---

## NFR-SEC-05 — Model protivnika i disciplina ulaganja

> **Status:** Prijedlog (Draft, 2026-07-22, skica). Teorijski okvir; usmjerava odluke, ne provodi
> se u CI.

**Izjava**

Sigurnosni rizik **nije egzogena varijanca** (kako pretpostavljaju opcijski modeli): protivnik
promatra arhitekturu i bira točku napada **nakon** odluke. Ispravan formalizam je **Stackelberg**
(branitelj vodi, protivnik odgovara najbolje) → kriterij je **minimaks nad najboljim odgovorom**,
ne očekivana vrijednost. Napadna površina ne trpi napade — privlači ih.

**Znanstveno i inženjersko opravdanje**

| Načelo | Implikacija |
|---|---|
| Stackelberg (vođa–sljedbenik) | branitelj bira prvi; optimizira minimaks, ne `E[·]` uz danu varijancu |
| Endogena volatilnost prijetnje | opcijski modeli (Baldwin–Clark) **podcjenjuju** sigurnosni rizik — varijanca je funkcija odluke |
| Gordon–Loeb (2002) | ulaganje ≤ ~`1/e ≈ 37%` očekivanog gubitka; najranjivija imovina nije nužno prioritet |
| Ekonomija sigurnosti (Anderson 2001) | mnogi neuspjesi su problem **poticaja**, ne mjerenja |

**Kriteriji prihvaćanja** *(teorijski / pregledom)*

1. Sigurnosne odluke procjenjuju se **minimaksom** nad protivnikovim najboljim odgovorom.
2. **[M]** Ulaganje ne prelazi **Gordon–Loeb** strop (~`1/e·E[L]`). Napomena: `1/e` vrijedi za
   **neovisne, neadaptivne** prijetnje — uz strateškog protivnika nije invarijantno.
3. Svaka procjena `P(kompromitacija)` ima **rok trajanja** (nestacionarnost) kraći od
   arhitektonske odluke koju opravdava; obnavlja se.
4. Trošak sigurnosnog neuspjeha mora snositi **odlučitelj** (poticaji); inače nijedan indeks ne
   popravlja pogrešnu raspodjelu.

**Verifikacija**

Pregled arhitektonskih odluka (threat model) + Gordon–Loeb provjera proračuna. Nije CI gate.

---

## Dodatak A — Redoslijed faceta u nomenklaturi klasa

## Znanstvena osnova za primarnu os imenovanja

Kad je ime klase složeno od ortogonalnih faceta
(`Pipeline + <Subject> + <Operation | ToTarget> + [Qualifier]`), jedna odluka uređuje
cijelu gramatiku: **koji facet vodi**. To nije stvar ukusa. Četiri neovisna korpusa
znanja informiraju izbor i ne slažu se uvijek. Gdje su u sukobu, ovaj dokument navodi
koje načelo prevladava i zašto.

### Metoda

Izvori su prikupljeni **ciljanim pregledom kanonskih izvora**, ne sistematskim
pregledom u PRISMA smislu. Nema iscrpne pretrage baza, formalnog protokola
uključivanja/isključivanja ni tvrdnje o pokrivenosti. Kriterij odabira bio je
namjeran: za svako vodeće pitanje identificirati temeljni rad koji utemeljuje ili
kanonski iznosi relevantno načelo. Razina rigoroznosti je stoga *sljedivost svake
tvrdnje do utemeljenog izvora*, ne *potpunost područja*. Ovo je navedeno eksplicitno
kako bi se pregled mogao revidirati i, po potrebi, kasnije podići na sistematski
protokol.

Okvirno pitanje cijelog istraživanja bilo je: **postoji li objektivan, mjerljiv
kriterij za redoslijed faceta u imenu klase, ili je nužno stvar konvencije?** To je
odredilo da se konzultiraju i teorija informacije i teorija klasifikacije, ne samo
stilski vodiči. Razlaže se na četiri sloja, svaki sa svojim vodećim pitanjima i
izvorom koji su ta pitanja odabrala.

| Sloj | Vodeća pitanja | Odabrani izvor |
|---|---|---|
| Teorija informacije | Kako se mjeri diskriminirajuća moć token-a imena? Zašto redoslijed token-a utječe na efikasnost čitanja? Jesu li dva faceta uistinu neovisna? | Shannon (1948) — entropija; prefiksni kodovi; uzajamna informacija |
| Knjižnična klasifikacija | Postoji li disciplina koja već propisuje redoslijed faceta? Što taj redoslijed optimizira i uz koju pretpostavku? | Ranganathan (1937) — PMEST; kolokacija |
| Kognitivna psihologija | Postoji li prirodna razina na kojoj ljudi kategoriziraju? Je li izbor logičan ili empirijski za dani tim? Koje načelo presuđuje kad se intuicija i teorija razilaze? | Rosch (1978) — basic-level kategorije; Principle of Least Astonishment |
| Strukturna usklađenost | Treba li struktura imenovanja pratiti strukturu koda? U kojem smjeru smiju teći ovisnosti? | Conway (1968); Stable Dependencies Principle |

Sintezno pitanje — *kad su stupovi u sukobu, koji prevladava i zašto?* — odgovoreno je
u zaključnom odjeljku.

### 1. Teorija informacije — poredak po padajućoj entropiji

Hijerarhijsko ime ponaša se kao prefiksni kôd. Čitatelj razrješava ime token po
token, a prefiks je najefikasniji kad facet koji *najviše grana* (najveća marginalna
entropija, najveća diskriminirajuća moć) dolazi prvi, jer u prosjeku najbrže sužava
prostor kandidata.

To je mjerljivo, ne estetsko. Za trenutni katalog od 27 pipeline klasa:

| Facet | Različitih vrijednosti | Shannonova entropija H | Efikasnost H / log₂(k) |
|---|---|---|---|
| Subject | 12 | **2.791 bita** | 0.779 |
| Operation | 7 | 2.056 bita | 0.732 |

Granica identifikacije je log₂(27) = 4.755 bita. **Subject** facet nosi višu
marginalnu entropiju i višu efikasnost, pa vođenje sa Subjectom najranije razrješava
najviše neizvjesnosti. Teorija informacije stoga favorizira **Subject-first**.

Drugo mjerenje kvalificira rezultat. Uzajamna informacija dvaju faceta je
I(Subject; Operation) = 1.862 bita — visoka. Faceti nisu potpuno ortogonalni u praksi:
jedan klaster (named-entity recognition) čini 44 % svih klasa i uniformno je `Extract`,
pa je unutar tog klastera diskriminirajuća informacija migrirala u Qualifier (engine,
jezik). Pretpostavka dviju ortogonalnih osi vrijedi svuda osim u tom klasteru, koji se
označava kao strukturna iznimka, a ne protuprimjer.

> *Referenca:* Shannon, C. E. (1948), *A Mathematical Theory of Communication*,
> Bell System Technical Journal — entropija kao mjera diskriminirajuće informacije;
> efikasnost prefiksnog koda.

### 2. Knjižnična klasifikacija — Ranganathanov PMEST i kolokacija

Facetna klasifikacija propisuje fiksni redoslijed navođenja faceta:
**Personality – Matter – Energy – Space – Time**. *Personality* (sama stvar — Subject)
prethodi *Energy* (procesu koji djeluje na nju — Operation). Vladajući razlog je
**kolokacija**: korisnik koji traži sve o danom subjektu očekuje to okupljeno na jednom
mjestu, a ne razasuto po operacijama. Vođenje sa Subjectom kolocira sve artefakte iste
vrste zajedno u imenskom prostoru i autocompleteu.

Argument je uvjetovan time da je dominantni način dohvata *po subjektu*, a ne *po
operaciji*; pretpostavlja da je prvo pitanje čitatelja „na što ovo djeluje?". Prijenos
knjižničnog redoslijeda *police/notacije* na imenovanje klasa je analogijski, ne
doslovan.

> *Referenca:* Ranganathan, S. R. (1937), *Prolegomena to Library Classification*
> — facetna analiza i PMEST redoslijed.

### 3. Kognitivna psihologija — basic-level kategorije

Ljudi spontano kategoriziraju na „basic level" koja je najinformativnija za zadatak.
Relevantno pitanje je stoga empirijsko, o mentalnom modelu tima: kad razvojnom
inženjeru treba klasa, je li prva misao *„trebam nešto za PDF"* (Subject-led) ili
*„trebam nešto što ekstrahira"* (Operation-led)? To se ne može riješiti logikom; mora
se promatrati. Principle of Least Astonishment tada zahtijeva da gramatika slijedi ono
što tim već misli, neovisno o teorijskoj preferenciji.

> *Reference:* Rosch, E. (1978), *Principles of Categorization*; Saltzer & Kaashoek
> (2009) i šire Principle of Least Astonishment u dizajnu sučelja.

### 4. Strukturna usklađenost — Conway's Law i Stable Dependencies Principle

Ako su moduli već organizirani po Subjectu (`pdf/`, `png/`, `cad/`), ime koje počinje
Subjectom zrcali fizičku strukturu, pa se ime i putanja podudaraju
(`pipelines/pdf/…PipelinePDF…`). To zatvara kognitivni jaz između navigacije
datotekama i čitanja imena — **homomorfizam imenskog i datotečnog prostora**, tako da
jedna taksonomija služi za oboje umjesto dvije rivalske.

> *Reference:* Conway, M. (1968), *How Do Committees Invent?* (labavo — opomena da se
> ime i struktura koda ne smiju razilaziti); Martin, R. C., Stable Dependencies
> Principle.

### Razrješenje sukoba

Teorija informacije (§1) i kognitivni model (§3) mogu se razići: matematički
najinformativniji poredak nije nužno onaj koji tim intuitivno čita. Kad su u sukobu,
**Principle of Least Astonishment (§3) i strukturna usklađenost (§4) imaju prednost
nad sirovom entropijom**, jer je svrha imena predvidljivost za ljudski i strojni um
koji njime navigira, ne maksimalna kompresija.

Metodologija je stoga: izračunati entropiju faceta iz postojećeg kataloga kao
objektivan ulaz (§1), potkrijepljeno kolokacijom (§2) i strukturnom usklađenošću (§4);
odlučiti vodeću os prema promatranom mentalnom modelu tima (§3) gdje je u napetosti s
mjerenjem. Za trenutni katalog mjerenje pokazuje Subject-first, a §2 i §4 se slažu; §3
je presudni glas kad se ne slažu.

### Ograničenja i reproducibilnost

* Brojke entropije i uzajamne informacije (§1) ovise o frekvencijskoj raspodjeli
  kataloga, koju treba **regenerirati lintom** iz žive liste klasa, a ne navoditi
  statički; sirovu raspodjelu treba priložiti kao podatkovnu datoteku da brojke budu
  neovisno provjerljive.
* Katalog je malen (n = 27) i dominira ga (44 %) `nlp` klaster, trenutno odgođen
  modul. Empirijska entropija je pristrana na malim uzorcima; brojke će se mijenjati
  kako katalog raste i moraju se ponovno računati.
* §3 „presudni glas" (mentalni model tima) **još nije izmjeren**; sadašnja odluka
  počiva na slaganju §1, §2 i §4, uz §3 kao buduću empirijsku provjeru (card-sort ili
  telemetrija autocomplete/pretrage).

---

# Reference

Ovaj registar je **policy** izveden iz istraživanja; **ne nosi vlastitu bibliografiju**
znanstvene literature. Znanstveni i epistemološki temelj mjeriteljskih (`[M]`) i sigurnosnih
(SEC) zahtjeva — pregled literature, praznine u znanju, izvod pitanja i zaključci — nalazi se
u istraživačkom radu **`tools/ANALIZA.md`** (kanonska bibliografija: 34 reference s DOI, §7).

NFR kriteriji referiraju **zaključke** te analize, a ne re-navode izvore:

| NFR | Zaključak analize |
|---|---|
| SEC-01 (blast radius) | `E[L]=Σ P·Blast·V`; struktura kao množitelj rizika — ANALIZA §5.5 |
| SEC-02 (napadna površina) | Manadhata–Wing; unutarnji optimum (zrcalo MDL) — ANALIZA §4.5, §5.5 |
| SEC-03 (supply-chain) | korelirani proboji nad tranzitivnim ovisnostima — ANALIZA §5.5 |
| SEC-04 (radna točka detekcije) | kontrola = klasifikator; ROC, base-rate, compliance budget — ANALIZA §5.4 |
| SEC-05 (protivnik/ulaganje) | Stackelberg/minimaks; Gordon–Loeb strop `1/e` — ANALIZA §5.3, §5.6 |
| `[M]` povelja | preduvjeti mjerenja; tip skale; postotak samo za 3 veličine — ANALIZA §4.1, §5.1, §5.7 |

Dodatak A nosi **vlastite inline reference** (Shannon 1948; Ranganathan 1937; Rosch 1978) jer
obrađuje zaseban predmet (redoslijed faceta u imenovanju), ne mjeriteljsko-sigurnosnu nit.

## Normativne reference (policy-mete izravno pozvane u kriterijima)

- ISO/IEC 25010:2011 — *Systems and software engineering — SQuaRE — System and software quality models.*
- ISO/IEC 15939:2017 — *Systems and software engineering — Measurement process.*
- NIST — *Open Security Controls Assessment Language (OSCAL).*
- Python Enhancement Proposals: PEP 8, PEP 420, PEP 484, PEP 660, PEP 740.
- CycloneDX; SPDX — Software Bill of Materials. Sigstore — artifact signing.
