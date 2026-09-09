# FRQ-PRC-15.22 — Tok dokumenta: nastanak i pohrana

| | |
|---|---|
| **Status** | Prijedlog. Oba toka **provedena su u kodu** — provjereno 2026-09-05; audit mjesta osvježena 2026-09-09 ([`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md), prijedlog) |
| **Odluka** | Nema vlastiti DR. Kategorija `PRC` odabrana po vlasništvu prolaza; izbor je otvoren (§11 t.4) |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ i zajednički ugovor generičke klase |
| **Predmet** | **Suradnja** primitivâ u dva toka, ne građa jednog primitiva: nastanak dokumenta i njegova pohrana |
| **Sestrinski** | [`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md) · [`FRQ-PIP-15.2`](FRQ-PIP-15.2-pipeline.md) · [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) · [`FRQ-STR-15.4`](FRQ-STR-15.4-strategy.md) |
| **Dijagrami** | Interakcija (tok poziva među instancama): [nastanak](FRQ-PRC-15.22-create-sequence.puml) · [pohrana](FRQ-PRC-15.22-persistence-sequence.puml). Aktivnost (radnje i objektni tok): [put dokumenta](FRQ-PRC-15.22-document-activity.puml). Pogledi s deklariranim gledištem, ne izvor istine (D-13) |
| **Izvedba** | `concrete/processor.py`, `concrete/pipeline.py`, `concrete/repository.py` (workflow) · `blackboards/small.py`, `strategies/documents/*.py` (processors) |

## 1. Predmet

Zahtjev opisuje **dva toka** kojima dokument putuje kroz sustav, i pravilo da nijedan sudionik ne
preuzima tuđi korak. Tok nastanka odgovara na *što je ovaj predmet*, tok pohrane na *gdje ide ono
što je nad njim napravljeno*.

Odluke koje tokovi utjelovljuju:

| pitanje | nositelj | ne smije nositi |
|---|---|---|
| koje jedinice postoje | procesor | sadržaj, imenovanje, brisanje |
| što je predmet | create strategija | transformaciju, pretvorbu, ime izlaza |
| što s njim učiniti | pipeline | pohranu, odabir odredišta |
| kamo ide | write strategija (jedna po repozitoriju) | sam upis u spremište |
| kako se čita i piše | driver | odluku *što* i *kamo* |

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `DriverManager` — gradi driver **jednom**; njegov životni ciklus počinje ovdje | konfiguracija `managers.drivers` |
| **A2** | `WorkflowFactory` — razrješava imena u instance i povezuje ih | konfiguracija |
| **A3** | `GenericProcessor` — vlasnik prolaza | generator, pipelinei, granica flusha |
| **A4** | `GenericBlackboard` — platno između stope pipelinea i stope repozitorija | `strategy_create`, registrirani repozitoriji |
| **A5** | `StrategyCreate` — nastanak dokumenta | opisni ključevi |
| **A6** | `GenericPipeline` — jedna transformacija nad jednom stavkom | facade + ključevi procesora |
| **A7** | `RepositoryWithDriver` — **jedan po odredištu**; nosi svoj driver i svoju write strategiju | `driver`, `strategy_write` |
| **A8** | `StrategyWrite` — priprema plaćeni sadržaj i imenuje izlaz | facade, driver |
| **A9** | `GenericDriver` — čita, piše, pretražuje | payload, ime, sufiks |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger | tok |
|---|---|---|
| **EV01** | A1 gradi driver i podiže ga (`ensure_live`) | priprema |
| **EV02** | A2 predaje driver repozitoriju (`_driver_context`) i procesoru (`configuration.driver`) | priprema |
| **EV03** | generator daje jedinicu → `blackboard.create(caller=procesor, …)` | nastanak |
| **EV04** | blackboard poziva `strategy_create.create(caller=self, processor=…, blackboard=self, …)` | nastanak |
| **EV05** | strategija vraća `DocumentFacade` → procesor ga `yield`-a | nastanak |
| **EV06** | `pipeline.process(processor, facade)` → `transform(...)` | pohrana |
| **EV07** | pipeline zove `processor.blackboard.write(pipeline=self, facade=…)` | pohrana |
| **EV08** | ciklus dovršen → `blackboard.flush(caller=procesor, **processor.write_context)` | pohrana |
| **EV08a** | flush dovršen → procesor piše granicu dokumenta (`msg=Processed`) | pohrana |
| **EV09** | blackboard obilazi **svaki** registrirani repozitorij → `repository.write(...)` | pohrana |
| **EV10** | repozitorij zove `strategy_write.write(..., driver=self.driver, ...)` | pohrana |
| **EV11** | strategija zove `driver.write(payload, filename=…, suffix=…)` | pohrana |

## 4. Preduvjeti

1. Driver je instanciran i podignut **prije** procesora — vlasnik ciklusa je `DriverManager`, ne
   potrošač (`BR-15-02` za registraciju, `FRQ-DRV-15.6` za sam driver).
2. Blackboard nosi `strategy_create`; bez nje `create` daje `warning` i `None`.
3. Blackboard nosi **barem jedan** repozitorij; bez njega `write` diže `BlackboardException`.
4. Svaki repozitorij nosi vlastitu `strategy_write`; `RepositoryWithDriver` uz to i `driver`.

## 5. Normalan tok

### 5.1 Nastanak — [interakcija](FRQ-PRC-15.22-create-sequence.puml)

1. Procesor otkriva jedinicu i filtrira je (ime, datum, redoslijed). **Ne otvara je.**
2. Poziva `blackboard.create(self, <opisni ključevi>)`. Blackboard asertira `IProcessor`.
3. Blackboard prosljeđuje **sebe** kao `caller`, a procesor kao `processor=` — pa create
   strategija asertira `IBlackboard`, ne `IProcessor`.
4. Strategija provjeri obvezne ključeve, sagradi dokument svojeg predmeta, žigoše ishodišne
   metapodatke (`created_by`, `created_at`, `caller`, `source_format`) i vrati `DocumentFacade`.
5. Procesor `yield`-a facade. Identitet dokumenta od te točke stoji.

### 5.2 Pohrana — [interakcija](FRQ-PRC-15.22-persistence-sequence.puml)

1. Procesor provlači facade kroz **sve** pipelinee redom.
2. `GenericPipeline.process` asertira tipove, piše otvarajući zapis **na `DEBUG`** i zove
   `transform` — jedina metoda koju specijalizacija piše. Na `INFO` razini pipeline šuti:
   dokument je jedinica posla procesora, ne pipelinea (`DR-WFL-028`).
3. `transform` uzme dokument iz facade, izvede **svoju jednu** obradu i obogati dokument
   sadržajem i/ili metapodacima.
4. Pipeline zove `processor.blackboard.write(pipeline=self, facade=…)`. Blackboard postavi platno;
   uz `defer_flush=False` odmah emitira u sve repozitorije, inače čeka flush.
5. Nakon svih pipelinea procesor uveća ciklus i — ako je `flush_per_cycle` — zove
   `blackboard.flush(caller=self, **self.write_context)`. Generički `write_context` je prazan
   rječnik; nijedna podklasa ga danas ne nadjačava (§11 t.6), pa strategije ne dobivaju nijedan
   ključ ovim putem.
6. Blackboard obilazi **svaki registrirani repozitorij redom** (`for`, ne istodobno). *N*
   repozitorija znači *N* write strategija i *N* drivera nad **istim** dokumentom.
7. Repozitorij pridoda vlastiti driver kroz `_strategy_context()` i zove svoju write strategiju.
8. Strategija razriješi formatter (`FormatterFactory`), sagradi payload, složi ime izlaza i preda
   ga `driver.write(...)`. Ime se **komponira pri upisu**, nikad ne čuva unaprijed.
9. Strategija žigoše metapodatke pohrane i vrati `True`; repozitorij uveća brojač; platno se prazni.
10. **EV08a** — procesor zatvara jedinicu dokumenta jednim `INFO` zapisom (`Processed`, s
    `cycle`, `source`, `document`). To je **jedini** `INFO` koji ovaj tok proizvodi po dokumentu
    iz sloja vlasnika; koraci lanca prijavljuju se prema `DR-WFL-021` t.3, čije je zatečeno stanje
    nalaz (`NFRQ-OBS-03` §4).

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `strategy_create` nije dodijeljena | `warning`, `create` vraća `None` | procesor `yield`-a `None` — nije pokriveno provjerom (§11 t.1) |
| nema registriranih repozitorija | `BlackboardException` | pipeline ne može pisati |
| `defer_flush=False` uz *N* pipelinea | repozitoriji se pišu **nakon svakog** pipelinea | *N* upisa po dokumentu; namjerno za audit/debug |
| `flush_per_cycle=False` | platno se drži do kraja prolaza | jedan flush za cijeli prolaz |
| write strategija ne prepoznaje predmet | `warning`, `return False` | brojač se ne miče; ostali repozitoriji rade |
| kvar u bilo kojem repozitoriju | `RepositoryException` → `BlackboardException` | **cijeli flush pada**; preostali repozitoriji se ne obilaze (§11 t.2) |

## 7. Rezultat

Dokument je nastao jednom, obogaćen onoliko puta koliko ima pipelinea, i pohranjen u onoliko
odredišta koliko je registrirano repozitorija — svako preko vlastite write strategije i vlastitog
drivera. Nijedan sudionik nije obavio tuđi korak.

Stanja kroz koja pritom prolazi — `[opisan] → [obogaćen] → [pohranjen]` — nosi dijagram
aktivnosti; tko koga poziva i s čime nose dijagrami interakcije.

## 8. Kriteriji prihvaćanja

1. Procesor ne otvara, ne imenuje, ne premješta i ne briše izvor. ✅
2. Create strategija ne transformira sadržaj; gradi dokument i žigoše ishodište. ⚠ — `CreatePdfDocument` krši (§11 t.3)
3. Blackboard predaje **sebe** kao `caller`, procesor kao `processor=`. ✅
4. Transformacija se događa isključivo u `transform`. ✅
5. `flush` obilazi **svaki** registrirani repozitorij. ✅
6. Repozitorij predaje **svoj** driver strategiji kroz `_strategy_context()`. ✅
7. Driver je instanciran jednom, kod `DriverManager`-a. ✅
8. Write strategija imenuje izlaz pri upisu i žigoše metapodatke pohrane. ⚠ — ključ žiga nije jedinstven (§11 t.5)
9. Pipeline dohvaća driver kroz `processor` koji prima u `process`. ✅
10. Create strategija dohvaća driver kroz `processor` koji joj blackboard predaje. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-09-04) |
|---|---|---|
| 1 | čitanje `processors/file.py` nakon izmjene | nema `read_text`, `unlink`, `move`; predaje samo `filename` |
| 3 | čitanje `blackboards/small.py:127` | `create(caller=self, processor=caller, blackboard=self, **kwargs)` |
| 5 | čitanje `blackboards/small.py:161` | `for repository in self._repositories: repository.write(...)` |
| 6 | čitanje `concrete/repository.py:278` | `_strategy_context() -> {"driver": self.driver}` |
| 7 | čitanje `concrete/workflow.py:_build_drivers` | jedna instanca po imenu, upisana u `DriverManager` |
| 9 | `command grep -rn "processor\.driver" src/wattleflow/pipelines/` | `pipelines/nlp/entities.py:87` — `processor.driver.read(table="entitet")`; mehanizam je u upotrebi |
| 10 | `command grep -n "processor=caller" src/wattleflow/blackboards/*.py` | sva četiri blackboarda predaju `processor=caller` create strategiji |
| — | `plantuml -tpng` nad sva tri `.puml` (2026-09-04) | sva tri se renderiraju bez greške i pregledana su |
| — | isto, nakon izmjene dijagrama pohrane (2026-09-09) | **izvedeno.** PlantUML 1.2025.4 (lokalni jar, `-tpng -Djava.awt.headless=true`) renderira svih 9 `.puml` datoteka u `02-FRQ/` bez greške; dijagram pohrane je i **pregledan** — obje audit oznake stoje na očekivanom mjestu: `DEBUG Transform/Started` uz korak 3 (pipeline) i `INFO Processed` uz korak 34 (procesor), iza `flush_per_cycle` bloka |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda, `command grep`, PlantUML 1.2025.4
(`-tpng`, lokalni jar, OpenJDK 17); kriterij — §8 gore; platforma — radna stabla `workflow` i
`blackwattle` 2026-09-09, CPython 3.11.15 (Linux/WSL2). **Mjereno stablo:**
`concrete/{processor,pipeline,repository,workflow}.py`, `blackboards/small.py`,
`strategies/documents/{file,pdf,mail}.py`, `02-FRQ/*.puml`.
**Slijepa pjega (D-11):** nijedan kriterij nije pokriven testom niti lintom — sve je pregled koda
i pregled rendera. Generirane slike **ne žive uz izvor** u `02-FRQ/`, protivno §3.1 („izvor `.puml`
+ generirana slika"); vrijedi za svih devet dijagrama registra, ne samo za ovaj — vidi
`workflow/TODO.md`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | primitivi su rezervirani; tok je mjesto gdje se njihova podjela vidi ili krši |
| `NFRQ-ORG-08` | petlja, granice kvara i audit žive u generičkim klasama, ne po specijalizacijama |
| `NFRQ-OBS-01/03` | **procesor** prijavljuje i stavku i prolaz (`2 + N` zapisa); pipeline je na `DEBUG` (`DR-WFL-028`). Što od koraka lanca doista prijavljuje — vidi mjerenje u `NFRQ-OBS-03` §4 |
| `NFRQ-SEC-01` | procesor ne poznaje spremišta osim kroz platno; strategija ne poznaje putanju osim kroz driver |

## 11. Otvoreno

1. **`create` smije vratiti `None`, a procesor to ne provjerava.** Bez `strategy_create` blackboard
   vraća `None`, procesor ga `yield`-a, a pipeline padne na `assert isinstance(facade, ITarget)` —
   dakle konfiguracijska pogreška se prijavljuje kao tipska, jedan sloj prekasno.
2. **Kvar jednog repozitorija ruši cijeli flush.** `for` petlja u `flush` nema po-repozitorijsku
   granicu, pa neuspjeh drugog odredišta poništava obilazak trećeg. Je li to željeno (atomarnost)
   ili nije (otpornost) — nije odlučeno.
3. **`CreatePdfDocument` krši kriterij 2**: otvara PDF s `fitz` i izvlači tekst pri nastanku
   (`strategies/documents/pdf.py`). Ekstrakcija pripada pipelineu; usporedbe radi
   `CreateFileDocument` radi ispravno.
4. **Kategorija ovog zapisa.** Predmet je suradnja više primitiva, a os kategorija
   ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)) daje po jednu kategoriju po
   primitivu. `PRC` je odabran jer procesor posjeduje prolaz i oba toka počinju u njemu. Alternativa
   je nova kategorija za tokove — ali ona ulazi kroz DR (D-12), ne dopisivanjem. Do odluke oznaka je
   **provizorna**.
5. **Ključ žiga pohrane nije jedinstven.** Write strategije žigošu `output` (file, json, pdf, word),
   `storage_filename` (text, dataframe, avro, orc, protobuf, image, graph, mail), `output_filename`
   (jedna grana u mail) i `storage_uri` (opensearch, solr). Tko god želi strojno provjeriti *je li
   dokument pohranjen* mora poznavati sva četiri. Razrješenje traži DR (D-12): proširiti popis ili
   uvesti kanonski ključ i uskladiti strategije.
6. **`write_context` je put bez putnika.** EV08 ga opisuje kao mehanizam kojim procesor objavljuje
   konfiguraciju write strategijama, i `flush` ga doista prosljeđuje sve do
   `strategy.write(**kwargs)`. Provjereno 2026-09-09: **nijedna podklasa `GenericProcessor`-a ne
   nadjačava `write_context`**, pa je rječnik uvijek prazan i cijeli put nikad nije izvršen.
   Zahtjev ga opisuje kao postojeći, što je točno za *mehanizam* i netočno za *upotrebu* — razlika
   koju D-05 traži da se označi. Uočeno preko konfiguracije koja je `skip_inline`/`skip_types`/
   `skip_below` stavila na procesor očekujući taj put; ključevi su odbačeni uz `warning`
   (`NFRQ-ORG-07` radi kako je propisano) i uklonjeni iz `06_fetch_emails.yaml`.
