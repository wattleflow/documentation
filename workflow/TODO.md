# TODO — wattleflow-workflow

Worklist; organizirano po cijeni izvedbe i riziku. Odvojen od `CLAUDE.md` (policy sloj):
policy nosi normu na snazi, ovaj popis nosi stanje rada. Zatvorena stavka se **briše**, ne
arhivira — povijest nose git i `DR-WFL` serija.

## Stanje izmješteno iz policyja (2026-08-16)

Brojke i oznake stanja izašle su iz `CLAUDE.md` po §9/D-13; ovdje im je mjesto.

- [ ] **`@staticmethod`/`@classmethod` migracija (`NFR-ORG-05`, §2.9).** AST popis zatečenih
  kandidata ponovno izvesti pretragom; posljednji poznati ostatak je `cad`. `processors` i
  `workflow` prošli, uz odgođene slučajeve zbog `cls`-param iznimke.
  - (2026-08-21) Zaseban ostatak istog pravila u `workflow`: **privatne modul-funkcije** uz
    pripadne modul-konstante (nisu bile dio prethodnog popisa, jer popis je gledao dekoratore
    postojećih metoda, ne slobodne funkcije). Omotane u kvalificirane klase; pretraga
    `^def _` nad `src/` sada je prazna. Konstante u `constants/filetype.py` namjerno ostaju
    modul-razine — unutar `Enum` tijela postale bi članovi enuma.
- [ ] **Aspiracije bez mehanizma (§6.2 SIEM, §6.4 observability).** Ostaju označene kao
  aspiracija (D-05) dok se ne odluči transport i format; tada prelaze u zahtjev ili se povlače.

## Usklađivanje dokumentacije (nalaz 2026-07-30)

Nalazi iz analize `PHILOSOPHY` / `METHODOLOGY` / `DOCTRINE` / `FR` / `NFR`. Sve su izmjene
registara → idu kroz DR (D-03).

- [ ] **Ukinuti reference na `tools/naming_registry.yaml`** u `NFR.md` (4×), `FR.md`,
  `METHODOLOGIA.md` (§10) i `DOCTRINE.md` (D-12) → `tools/dictionary.json`.
- [ ] **Ispraviti putanje:** `docs/adr/helpers/DR-WFL-001` → `documentation/04-DR/…`;
  `docs/conformance/` → `documentation/workflow/conformance/`; `documentation/dr/`
  (FILOZOFIJA, tablica svezaka) → `documentation/04-DR/`; `tools/ANALIZA.md` →
  `tools/Analiza.md`; `documentation/dictionary.json` (POSTULATE, RIJECNIK) →
  `documentation/dictionary.yaml`; `documentation/literatura.md` (dictionary.yaml) →
  `LITERATURE.md`.
- [ ] **Ukloniti zastarjele brojke lint nalaza** („13 ERROR + 16 WARN" u `NFR.md`/`FR.md`) —
  zamijeniti uputom na C-snimku; nalaz bez trojke krši D-10.
- [ ] **Dvostruki registri (D-12/D-13):** `LITERATURE.md` (1–59) i `workflow/hr/LITERATURA.md`
  (1–57) dodjeljuju **ključevima 56/57 različite radove** → kršenje append-only pravila; isto
  drift `POSTULATE.md` (P-21) vs `workflow/hr/POSTULATI.md`. Odabrati jedan izvor po registru;
  uskladiti i layout (dio korijenskih dokumenata su symlinkovi na `hr/`, dio nisu).
- [ ] **Oznake odluka u `DOCTRINE.md`:** `DR-013/014/016/018` bez prefiksa; `DR-016`/`DR-018`
  nemaju zapis ni u jednom indeksu. Prefiksirati i otvoriti zapise ili označiti kao kandidate.
- [ ] **`DOCTRINE` D-02:** „suspendirano do `DR-015`" → `DR-WFL-004` (preimenovano u indeksu).
- [ ] **`METHODOLOGIA` Povijest izmjena / Bilješka t.2:** nulti rename „DR-WFL-004 →
  DR-WFL-004" → „`DR-015` → `DR-WFL-004`".
- [ ] **Predložak DR-a vs `DOCTRINE`:** polje *Temelji* (D-01/D-05/D-15) nije u predlošku →
  dodati u predložak ili uskladiti naziv sa *Svjedočanstvo*/*Registar*.
- [ ] **`NFR-ORG-06` više ne postoji** (dekomponiran u SEC-01/02/03, 2026-07-22): ažurirati
  `METHODOLOGIA` §7 i §10, `DOCTRINE` D-07 i zaglavlje `DR-WFL-002` → `NFR-SEC-03`.
  Usput: `NFR-SEC-03` citira „`DR-WFL-002/07`" — `07` je predrename oznaka (`DR-WFL-003`).
- [ ] **Akronimi — sukob registra i metode:** `NFR-ORG-02` kriterij 4 i `NFR-ORG-03` kriterij 4
  kažu „prekršaj / build pada", dok je pravilo suspendirano na WARNING do `DR-WFL-004`
  (`METHODOLOGIA` §8, `acronym_identifier_casing.status: undecided`). Kriteriji moraju nositi
  suspenziju (severity iz statusa) — inače metoda propisuje neodlučeno (D-02).
- [ ] **`ADR` ostaci u `NFR.md`** („pod ADR upravljanjem", „kroz ADR") — `ADR` je zabranjen za
  nove zapise; `NFR-ORG-03` već koristi „kroz DR".
- [ ] **`DR-WFL-002` §5, zadnja natuknica:** „Umbrella distribucije (**budući `DR-WFL-003`**)"
  — zaostatak predrename numeracije. `DR-WFL-003` je čuvana opcionalna ovisnost; registar
  umbrella distribucija nema zapisa ni kandidata. Preimenovati u „budući DR" bez broja i
  otvoriti kandidata, ili povući stavku.
- [ ] **Test maskiranja nije ničiji kriterij.** Postupak nose dva DR-a (`DR-WFL-003` §2.3 na
  razini modula, `DR-WFL-007` §4 na razini paketa) i §2.7, ali nijedan NFR ga ne navodi kao
  kriterij prihvaćanja, pa ga `wem_lint` ne mjeri. Kandidat: `NFR-SEC-03` novi kriterij
  („uvozivost paketa pod maskiranim opcionalnim tierom") ili `NFR-SEC-02` (import-time
  closure je učitani kod = napadna površina). Traži DR (D-03).
- [ ] **Agregatni rub nestaje iz AST-a (`DR-WFL-007`).** `dictionary.yaml` već bilježi da
  ORG-01 fan-in ne razrješava agregatne uvoze pa je donja granica; s `__getattr__` agregatom
  ruba nema ni u AST-u — mapa `_EXPORTS` je jedini statički trag. Pravilo mora naučiti čitati
  tu mapu (obična literalna `dict`), inače `drivers`/`connections`/`processors` postaju
  nevidljivi grafu ovisnosti.
- [ ] **`PHILOSOPHY.,md`** (zarez u nazivu datoteke) — 17 pojava u 7 dokumenata
  (`POSTULATE.md`, `workflow/hr/{POSTULATI,LITERATURA,METHODOLOGIA}.md`, `DR-WFL-002`,
  `DR-WFL-003`, `workflow/changes/2026-07-28-…`). Nijedna ne razrješava u postojeću putanju.
- [ ] **`NFR-ORG-02`, implementacijske napomene:** `PipelinePDFExtractText →
  PipelinePDFExtractText` je identitet (izvorno ime izgubljeno); popis traži
  `PipelinePDFRedact → PipelinePDFRedact`, a tablica u „Deduplikacija koda" vodi
  `PipelinePDFRedact` — uskladiti worklist imena.
- [ ] **Duplicirane „Zajedničke definicije"** (domena, helper, kanonski subjekt) u `FR.md` i
  `NFR.md` — držati u jednom registru, drugi referira.
- [ ] **Kriva referenca sljedivosti:** `FR.md` i `NFR.md` upućuju na „`METHODOLOGY.md` §9
  *Arhitektonska sljedivost*". Provjereno 2026-08-23: §9 je *AI-potpomognuto inženjerstvo*, §8
  je *Samoopisivost i imenovanje*, a **sljedivost je §6**. Usput: reference na `METHODOLOGIA`
  §10/§11 u ovom worklistu pokazuju na odjeljke kojih nema — dokument ima §1–§9 (+5.1, 7.1);
  cijeli skup unakrsnih referenci treba ponovno prebrojati, ne prepravljati napamet.
- [ ] **`POSTULATE.md` zaglavlje** navodi nadređene v0.4 / v0.3 → v0.4.1 / v0.3.1.
- [ ] **Kolizija DR oznaka u nacrtima:** `core/DR.md` i `workflow/concrete/DR.md` nose
  neprefiksirani `DR-001` (indeks ih vodi kao nacrte, jedan „sadržajno proturječan") → staviti
  banner „superseded" u same datoteke.
- [ ] **`DR-WFL-006` vs kod:** odluka pina `wattleflow>=0.0.0.46`, `pyproject.toml` ima
  `>=0.0.0.47` → zabilježiti kao verziju zapisa (D-03).
- [ ] **`FR.md` je prazan** (`## FR-ORG-01 — ...`) iako ga FILOZOFIJA/METHODOLOGIA/DOCTRINE
  tretiraju kao aktivan registar (Svezak V) → popuniti ili deklarirati kao aspiraciju (D-05).
- [ ] **Nedovršene doktrinarne tvrdnje:** (a) `DOCTRINE` tvrdi da proza citira norme
  D-oznakama — nijedna D-oznaka nije u `PHILOSOPHY`/`METHODOLOGY`; (b) **H4-DQI** se koristi
  kao doktrinarna hipoteza, ali `PHILOSOPHY` §Hipoteze ima samo H1–H3— upisati u registar
  hipoteza ili je voditi kao kandidat.

## Metode

- [ ] **Registar metoda ne postoji**, pa dokument metode nema oznaku i referira se putanjom.
  Uz registar ide i izdvajanje drugog referentnog primjera (`METHODOLOGIA` §10, wem_lint) —
  taj pripada ovoj distribuciji, za razliku od DQI-ja.
- [ ] **DQI nalazi pripadaju `wattleflow-processors`** — metoda i implementacija su tamo
  (`processors/method/dqi.md`, `pipelines/quality/dqi.py`). Prenijeti u worklist te
  distribucije kad dobije svoj: modul se ne uvozi (referira `wattleflow.helpers.datetime`,
  preimenovan u workflow stablu — cross-distribucijska posljedica), implementirana je jedna
  dimenzija od šest, `__all__` nije deklariran.
- [ ] **`POSTULATE.md` zaglavlje** navodi nadređene v0.4 / v0.3 → v0.4.1 / v0.3.1.
- [ ] **Tri mrtve poveznice u `METHODOLOGIA.md`:** `PHILOSOPHY.md`, `DOCTRINE.md` i
  `POSTULATE.md` navedeni su relativno, a dokument živi u `workflow/hr/` — razrješavaju se
  tek s `../../`. (`NFR.md` radi jer u tom direktoriju postoji istoimena datoteka.)
- [ ] **Kolizija DR oznaka u nacrtima:** `core/DR.md` i `workflow/concrete/DR.md` nose
  neprefiksirani `DR-001` (indeks ih vodi kao nacrte, jedan „sadržajno proturječan") →
  staviti banner „superseded" u same datoteke.
- [ ] **`DR-WFL-006` vs kod:** odluka pina `wattleflow>=0.0.0.46`, `pyproject.toml` ima
  `>=0.0.0.47` → zabilježiti kao verziju zapisa (D-03).

## Veće (značajna cijena ili arhitektonske odluke)

- [ ] **Sijanje entiteta pripada driveru, a driver ga danas ne zna primiti** (nalaz
  2026-09-05). `EntityFileDocumentProcessor._sync_entities` je pisao `INSERT INTO
  entitet/patterns` **iz procesora**, preko `self._conn` i `self.validate` — a nijedno ne
  postoji nigdje u MRO-u, pa bi svaka konfiguracija s `entities:` pukla na `AttributeError`.
  Kod je uklonjen (procesor ne piše, `NFRQ-ORG-04`), pa je sposobnost sada **odsutna**, ne
  samo premještena. Vratiti je znači proširiti `DriverEntity`, koji je pandas-tabelarni
  (`DataFrame` → CSV/XLS/JSON/Parquet) i nema SQLite konekciju — dakle novi ugovor na tom
  driveru ili drugi driver. Arhitektonska odluka, traži DR.
- [ ] **`EntityFileDocumentProcessor` više ne radi ništa s entitetima** (nalaz 2026-09-05).
  Nakon uklanjanja sijanja i Tika ekstrakcije procesor samo pretražuje, klasificira rutu,
  filtrira po datumu i predaje jedinicu — ime je postalo netočno. Preimenovanje ide kroz
  gramatiku `NFRQ-ORG-02` (`Processor + <Subject> + <Operation>`), pa je i sam izbor imena
  predmet odluke, ne stila. Ime se navodi na **osam** mjesta u primjerima —
  `03_pdf_pii`, `04_pii_reduction`, `06_pii_complex_workflow` i `23_opensearch`, svaki u
  `.py` (registracija) i `.yaml` (`type:`) — pa preimenovanje mora ići kroz sve, ili uz
  prijelazni alias.
- [ ] **Migracija tip-hintova na PEP 585/604** — `pyupgrade --py311-plus`; 65 starih
  (`Optional`, `Union`, `List`, `Dict`, `Tuple`) prema 6 novih u `core/` + `concrete/`
- [ ] Implementirati SIEM forwarding (`audit/` prazan; iskoristiti `AsyncHandler` +
  `Audit.subscribe_handler()`)
- [ ] OSCAL katalogizacija — `component-definition`, `assessment-results`, dodatni katalozi
- [ ] Observability — Prometheus / OpenTelemetry za FSM tranzicije i throughput (uz V1–V6)
- [ ] **Razdvajanje specijalizacija u zaseban projekt** — `connections/`, `drivers/`,
  `processors/`, `strategies/`, `pipelines/`, `documents/` izlaze iz clean core paketa
  (§7). **Regulira `DR-WFL-002` + `NFR-SEC-03`:** matična distribucija = najteža ovisnost;
  per-distribucija manifest; symlink → editable install (PEP 420/660); supply-chain preko
  SBOM/lock. Worklist preseljenja: `helpers/converters/`, `mappers/schema_yaml_json.py`.
  - [ ] **Self-integritet + shadowing gate (`NFR-SEC-03` kriterij 5).** Za izgrađene artefakte
    verificirati wheel `RECORD` (per-file `sha256`) — ne raditi vlastiti digest-format. Za
    dev/editable stabla `RECORD` je prazan → `wem_lint` digest-scan (`FileDigest`) koji
    istodobno detektira namespace-sjenčanje/koliziju.
  - [ ] **Dovršiti PEP 420 namespace migraciju (nastavak 2026-07-12).** `wattleflow.helpers`
    je namespace (uklonjen workflow `helpers/__init__.py`; 32 lib fajla na eksplicitnim
    submodul uvozima; bez symlinkova, cross-distribucijske klase uvijek eksplicitno, nikad
    agregat/`__all__`). **Preostaje:** (1) migrirati `examples/processors/*` + ručno riješiti
    nepostojeće simbole (`FileType`→`wattleflow.constants`, `LocalPath`/`Preset`);
    (2) `examples/todo/*` je dijelom pre-broken — počistiti ili arhivirati; (3) isti princip
    na ostale dijeljene subtree-eve (`decorators` itd.). Memorija: `helpers-pep420-namespace`.
- [ ] **Testovi** — odgođeno do odluke o test frameworku (§4)

## `__slots__` i MRO (nalaz 2026-06-29, djelomično riješeno)

Klasa koja spaja `blackboard` i `originator` obitelj ne linearizira MRO ako korijenska baza
ne prethodi generičkom parametru. Riješeno Fixom B — preslagivanjem redoslijeda baza u
`concrete/`, bez diranja corea (§2.5). Redoslijed baza je zato **ograničenje, ne stil**;
komentar uz klasu to čuva na mjestu.

- [ ] **Fix A (niži prioritet)** — alternativa: uskladiti **core** `IOriginator` na
  `(Generic[State], IWattleflow, ABC)`, tj. isti redoslijed na sučelju `originator` obitelji. Dira autoritativni core (§2.5 → `DR-COR`) i ima širi domet;
  razmotriti ako se pojavi još C-builtin/MRO sudara.

## OSCAL crosswalk — čeka compliance sign-off

`oscal/crosswalk.py` prepisuje control-id iz izvorne taksonomije (NIST) u ciljnu (ISM) prije
provjere; unmapped ID-evi prolaze nepromijenjeni. Mapping je **kuriran compliance artefakt** —
ne izmišljati mapiranja.

- [ ] **Mapiranja su `PROPOSED`** (`status: proposed-requires-compliance-review`, 2026-05-31)
  i traže sign-off prije oslanjanja. Postgres deklaracija: `ac-3→ism-0445`, `ia-5→ism-1401`
  (E8 ML1); `sc-8→ism-0469`, `sc-13→ism-1080` (izvan Essential Eight opsega — protiv E8
  baselinea ispravno padaju).

## Konzistentnost i standardi

- [ ] **`tools/README.md` preskače izdanje.** Changelog ide 1.11.0 → (ništa) → 1.13.0; za
  `wem_lint 1.12.0` nema zapisa iako je promijenio opseg mjerenja (scope filtar). Dopisati ili
  deklarirati kao rupu.

- [ ] **Preimenovanje paketa** `wattleflow-workflow-processors` → `wattleflow-processors` je
  provedeno u kodu; preostaje uskladiti dokumentaciju i PyPI/GitHub opise.
- [ ] **Preostali lazy-loading deferrali** (§2.7): `strategies/__init__` (documents strategije),
  `pipelines/__init__` (nlp).

- [ ] **`REQUIRED` uz `ALLOWED` — preporuka, traži mjerenje vrijednosti (nalaz 2026-08-31).**
  `ALLOWED` deklarira *dopuštene* konfiguracijske ključeve; ništa ne deklarira **obvezne**.
  Posljedica: `WorkflowFactory` je repozitoriju bez `configuration.driver` tiho predavao
  `None`, a kvar se javljao tek u strategiji. Privremeno rješenje (dogovoreno 2026-08-31):
  tvornica čita `PresetGate.resolve(<klasa>)` i traži `driver` ako ga klasa deklarira — dakle
  *dopušteno* se čita kao *obvezno*, što je semantičko rastezanje i ovdje se vodi kao nalaz.
  Preporuka: druga deklaracija `REQUIRED = (...)` koju tvornica provodi jednako za sve
  komponente (veze, driveri, procesori, repozitoriji), umjesto provjere po tipu.
  **Prije odluke izmjeriti vrijednost:** prebrojati konfiguracijske ključeve koji su danas
  de facto obvezni po komponenti (`ALLOWED` naspram onoga što konstruktor asertira) — ako ih
  je malo, mehanizam ne zaslužuje vokabular. Širi kontrolirani vokabular → traži DR (D-03,
  D-12).

### Refaktoring: migracija tip-hintova

Potvrđeno i u `wattleflow-processors` (73 datoteke s `Optional`/`Union`/`List`/`Dict`):

1. Uvesti `from __future__ import annotations` gdje nedostaje.
2. `pyupgrade --py311-plus` (ili `ruff check --select UP --fix`), **jedan commit po pod-paketu**.
3. Ručna provjera runtime-evaluiranih anotacija (Pydantic/dataclass/`get_type_hints`).
4. `ruff`/`mypy` gate u CI nakon migracije.

### Tika kao connection/driver + driver-kanal za create-strategije (nalaz 2026-06-26)

**Kontekst.** `EntityFileDocumentProcessor` bezuvjetno ekstrahira sadržaj preko Tike i
prosljeđuje `content=` u `blackboard.create`. Arhitektonski pogrešno: ekstrakcija je
odgovornost create-strategije, ne procesora.

**Odluka korisnika (2026-06-26): odgođeno.** Smjer (ne implementirati bez dogovora):

- [ ] **ConnectionTika + DriverTika** (OSCAL obavezan). Server-management (jar staging, Java
  provjera, timeout) seli iz `OCRTextProcessor` u `ConnectionTika`.
- [ ] **CreateTextDocument** koristi `DriverTika` za ekstrakciju iz `filename`;
  `CreatePdfDocument` (fitz) ostaje za PDF.
- [ ] **EntityFileDocumentProcessor** prestaje zvati Tiku; prosljeđuje samo `filename`.
- [ ] **Otvoreno pitanje — driver-kanal za create-strategije.** Create-strategije nemaju
  pristup driveru (write-strategije ga imaju preko `kwargs.get("driver")`). Smjer: isti
  mehanizam; izvedba dira `concrete/` (§2.5 — tražiti odluku).
- [ ] **Usputno:** `ReadDocumentFile` se referira u yaml-ima (03/04/05) ali **ne postoji**.

**Interim:** `tika_timeout` default 300s; suvišan Tika poziv u 04 ostaje do refaktora.

## Deduplikacija koda — DRY refaktor (nalaz 2026-06-27, ČEKA DR)

**Status:** analiza dovršena; implementacija **zaustavljena na zahtjev korisnika** dok se ne
izradi DR (arhitektura + NFR) koji vodi sustavni pristup. Ne dirati kod do tada.

> **D1 RIJEŠEN + reorg pipelinea (2026-07-06, direktiva korisnika — D2–D13 i dalje DR-gated).**
> `ocr_tokens` → **`helpers/ocr.py` `OcrText.tokens()`** (lazy pytesseract + `safe_open`).
> Razdvojena ekstrakcija od redukcije, native od generičkih:
>
> | Klasa | Modul | Uloga |
> |---|---|---|
> | `PipelineOCRExtract` (`OCRPreflightMixin`) | `pipelines/ocr/ocr.py` | tekst-ekstrakcija (Tika/tesseract, format-agnostično) |
> | `PipelineReductSpans` | `pipelines/entity/reduct.py` | samo redukcija — OCR bbox spanovi, image-only |
> | `PipelineMacroRedaction` | `pipelines/entity/macros.py` | text macro redakcija (`pii_hits`) |
> | `PipelinePDFExtractText` | `pipelines/pdf/pdf.py` | PDF tekst-ekstrakcija (native + OCR dopuna) |
> | `PipelinePDFRedact` | `pipelines/pdf/pdf.py` | PDF redakcijski spanovi (fitz `search_for`) |
>
> `pipelines/png/` uklonjen. **Otvoreno:** pixel-apply write-strategija `WriteReductedPNG`
> nije implementirana — `PipelineReductSpans` piše spanove u metadata.
> *Napomena:* imena u ovoj tablici i worklist preimenovanja u `NFR-ORG-02` se razilaze —
> vidi „Usklađivanje dokumentacije".

| # | Klaster | ~Pojava | Lokacije (uzorak) |
|---|---|---|---|
| D2 | Tipovi (Tokens/RedactBoxes/…) | 3 modula | pdf/png pipelines + `strategies/documents/pdf.py` |
| D3 | Strategy `execute()` try/except→`StrategyException` | ~41 | sve `strategies/documents/*` |
| D4 | Driver resolution (3 varijante) | ~21 | kanon: `strategies/documents/pdf.py _resolve_driver` |
| D5 | `isinstance(caller/facade…)` preambule | ~38 | sve strategije (osim `pdf.py` → `Attribute.evaluate`) |
| D7 | Lazy import guard | ~34 | `connections/*`, `drivers/*`, `processors/youtube.py` |
| D8 | `raise DriverXxxError(caller=self, …)` wrapper | **112** | `drivers/*`, `connections/*` |
| D9 | Prazne `class DriverXError(DriverException): pass` | ~35 | kolizija `KafkaConnectionError` 2× |
| D10 | Record-document klase ~80% iste | 5 | `documents/{avro,orc,protobuf,solr,opensearch}.py` |
| D11 | Graph-document metode (8 verbatim) | 2 | `documents/graph.py`, `documents/youtube.py` |
| D12 | HTTP driver trio `_resolve_auth/_request/_json` | 3×3 | `drivers/{grafana,kibana,prometheus}.py` |
| D13 | FileScanner + source_path guard | 4 | `processors/{file,file_entity,tesseract,tika}.py` |

**Prijedlog — iskoristiti POSTOJEĆU infrastrukturu:** `FileType.detect/detect_content`,
`FormatterFactory`/`ParserFactory`, `helpers/converters` (Facade uzor), `Attribute.evaluate`,
`safe_open`, `Normaliser`/`TextMacros`, `decorators/preset.py` + `decorators/oscal/policy.py`,
`helpers/system.py`, `BaseWriteStrategy`.

- [~] **Helper/Facade:** **[x]** `helpers/ocr.py OcrText.tokens()` → **D1**;
  **[ ]** `helpers/system.py += require_module(name, pip=…)` → **D7**;
  **[ ]** `strategies/_support.py resolve_driver()` + `stamp_created/stored()` → **D4, D6**
- [ ] **Dekoratori:** `@strategy_guard` → **D3** (alternativa u core `GenericStrategy.call()`
  — §2.5, traži `DR-COR`); `@wrap_errors(DriverXxxError)` → **D8**
- [ ] **Mixini/baze:** `RecordDocument` → **D10**; `GraphDocumentMixin` → **D11**;
  `HttpJsonDriverMixin` → **D12**; `FileSourceMixin` → **D13**
- [ ] **Tipovi/konstante:** centralni `pipelines/types.py` → **D2**; `*_SUFFIX` iz `FileType`
- [ ] **D9:** ostaviti eksplicitne klase (greppabilnost), riješiti koliziju `KafkaConnectionError`

**Bugovi uočeni usput:** youtube write-strategija diže `PipelineException` (neuvezen) umjesto
`StrategyException`; `strategies/documents/text.py:149,195` `exec=e` umjesto `exc=e`;
`strategies/documents/solr.py:48-64` gradi `SolrDocument` 2×; `stored_at` nedosljedan
(`Now.utc()` vs `utc_time_stamp()`).

**Granice (§2.5):** `@strategy_guard` u coreu traži odluku korisnika; third-party helperi (OCR)
**moraju** u processors paket, lazy (§7.4). Redoslijed: (1) pipelines, (2) strategije, (3) driveri.

## NFR-ORG-01 / SEC-03 nalazi (wem_lint)

Rješenja **moraju poštovati zero-trust (§7.1)** i zahtijevaju strukturne izmjene. Stanje
vektora se ne prepisuje ovdje (§9, D-13) — pokreni `tools/wem_lint.py --snapshot` i referiraj
snimku u `documentation/workflow/conformance/`.

- [ ] **Ciklus i jednosmjerni proboj `concrete` ↔ `helpers`.** Shared helper uzvodno uvozi
  jezgru (`helpers/config/base.py` → `concrete.base`, uz povratni brid `concrete/workflow.py` →
  `helpers`), dok concrete uvozi `helpers.*`. **Opseg smanjen 2026-08-20:** `config_adapter` i
  `config_validator` preselili su u `wattleflow-processors` (`DR-WFL-012`), pa više nisu dio
  ovog nalaza. Smjer (TBD, dira `concrete/` → §2.5): zajedničke iznimke/konstante ispod
  helpersa ili lokalne iznimke u helpersima; cilj acikličnost shared sloja. **Postalo mjerljivo
  tek s `wem_lint 1.12.0`** — prije je scope filtar brisao module prije nego ih pravilo vidi.
- [ ] **Odluka o razini dok se ciklus ne razriješi.** `domain_acyclicity` je `error`, pa ti
  nalazi ruše build. Ostaviti tako, spustiti na `warning` ili dati deklarirani waiver — izmjena
  `rules` bloka u kriteriju, dakle **kroz DR** (D-03).
- [ ] **Jedno-potrošački dijeljeni helperi.** Prema ORG-01 kriteriju 1 pripadaju unutar domene
  koja ih koristi ili u njezin domain-internal shared modul; tolerirani su dok se ne presele.
- [ ] **§7.1 procurivanje third-partyja u jezgru:** `mappers/schema_yaml_json.py` →
  pandas/yaml/jsonschema (kandidat za `wattleflow-processors`). `concrete/logger.py`→pandas je
  **riješen 2026-07-22** (duck-typing `hasattr(shape, "columns")`).
- [ ] **18 processors-modula u workflow stablu** (converters/, formatters/, parsers/, protobuf,
  image_guard, generators, localmodels) — kandidati za seljenje (vezano uz „Razdvajanje
  specijalizacija"); `scope` filtar ih za sad isključuje iz opsega.

## Konfiguracijski moduli (stanje 2026-08-19, uz `DR-WFL-012`)

Riješeno tim zapisom: prekid uvoza `concrete/`, nemogućnost izvršenja alata konformnosti,
i deklarirana iznimka bez pokrića (`scope.guarded_optional` sada prazan, kriterij 0.8.2).
Ostaje:

- [ ] **Je li `DR-WFL-003` još živ?** Odluka o čuvanoj opcionalnoj ovisnosti nema više
  nijednog nositelja u ovoj distribuciji. Povući je ili je zadržati kao mehanizam za
  buduće slučajeve — kroz DR, ne šutnjom.
- [ ] **Processors nema C-snimku.** Alat je nad njim izvršen 2026-08-22 (**0 error**,
  warningi zatečeni), ali vektor nije arhiviran kao snimka — `processors/conformance/`
  još ne postoji.
- [ ] **Primjeri se ne uvoze** — `from wattleflow.helpers import …` pretpostavlja agregat
  `wattleflow.helpers` koji ne postoji ni u jednoj distribuciji (zatečeno). Par
  `01_processor_etl_direct.py` / `01_processor_synthetic_data.py` usklađen 2026-08-22;
  preostaje **11** datoteka u `examples/processors/` plus `examples/todo/`. Popis imena za
  preseljenje: `Attribute` → `concrete.helpers`, `TempPathHelper`/`Project` →
  `helpers.system`, `Normaliser` → `helpers.normaliser`, `TextStream` → `helpers.streams`,
  `TextMacros` → `helpers.macros`, `FileType` → `constants.filetype`, `ConfigAdapter` →
  `helpers.config_adapter`, `Config` → `YAMLConfig` iz `helpers.config_yaml`,
  `ProcessorMemento` → `GenericMemento` (`concrete.memento`), `DataFrameDocument` →
  `documents.dataframe`. Primjeri nisu pod verzijskom kontrolom i third-party stack nije
  instaliran, pa izvršne provjere nema — samo razrješavanje imena.
- [ ] **`get()` nosi dvije nespojive semantike** u konfiguraciji i adapteru; dok se ne
  usklade, nije dio ugovora `IConfig` (`DR-COR-016` §Otvoreno).
- [ ] **`DR-WFL-017` je prijedlog, a kod je već izveden** (jezgra `v0.0.1.2` + radno stablo,
  processors radno stablo). `wattleflow.enums`, `constants`, `decorators` i `helpers` sada su
  PEP 420 imena bez `__init__.py` u objema distribucijama; uvoz je eksplicitan submodul.
  Prevesti zapis u *prihvaćen* ili ga odbaciti — do tada registar nosi dvije nespojive tvrdnje
  o vlasništvu imena (`DR-WFL-016` t.2).
- [ ] **`DR-WFL-007` ne spominje dijeljeno ime.** `CLAUDE.md` §2.7 t.4 propisuje eksplicitan
  submodul za cross-distribucijski uvoz, ali zapis na koji upućuje bira samo između eager i
  odgođenog agregata — treći slučaj (*agregata nema jer ga nijedna distribucija ne posjeduje*)
  ondje nije zapisan. Dopuniti (`DR-WFL-017` §Otvoreno t.2).
- [ ] **`wattleflow-processors` nosi `constants/errors.py` i `constants/keys.py`** — ime koje
  jezgra posjeduje, a njegov kod ih više ne uvozi. `MANIFEST.in` ih šalje u sdist,
  `packages.find` ih isključuje iz wheela (`SEC-03` K12). Odlučiti brisanje.
- [ ] **Repovi `DR-WFL-016` (mrtvi vokabular).** Preseljen je, ne obrisan — odlučiti brisanje
  (`ConnectionStatus`, `EventLog`, `ProtectiveMarkings`, `WattleflowOSCAL`, `PipelineAction`,
  `PipelineType`, `ProvenanceHandler`, većina `errors.py`/`keys.py`). `Event.Classification`
  ostaje u jezgri iako sam pojam klasifikacije više nije ondje — pregledati kad se `Event`
  bude čistio.
- [ ] **Repovi `DR-WFL-015` (sloj usklađenosti u processorsu, `v0.0.0.99` / `v0.0.20`).**
  `constants.WattleflowOSCAL` ostao je u jezgri bez ijednog potrošača (seli ili se briše —
  mijenja javni API `constants`); `wattleflow-oscal` treba povući s PyPI-ja ili označiti
  napuštenim dok kolizija imena ne prestane biti moguća; TypeVar `Node` u `oscal/models.py`
  nije u vokabularu uloga (`NFR-ORG-03`); `namespaces` nije deklariran u
  `processors/pyproject.toml` (`SEC-03` K11).
- [ ] **Ostaci u dokumentacijskom stablu.** `hr/METHODOLOGIA copy.md` (stariji nacrt,
  slomljene relativne poveznice), `hr/FILOZOFIJAmd.superseeded` (nosi *Bilješku o sintezi*
  koje u `FILOZOFIJA.md` nema) i `OLD-TODO.md`. Odlučiti: prenijeti sadržaj pa obrisati.

## Verzijska kontrola i objava dokumentacije (nalaz 2026-08-03)

Praćenje je riješeno (§8): `documentation` v0.0.5 prati cijeli sadržaj, push na `origin` je
onemogućen. Preostaje ono što praćenje ne rješava:

- [ ] **Hrvatski doktrinarni tekst je već javan na GitHubu.** `.gitignore` je štitio samo
  `*/hr/*`, pa su `DOCTRINE.md`, `POSTULATE.md`, `dictionary.yaml`, cijela `04-DR/`
  serija, `workflow/analysis/` i `core/DR.md` objavljeni na
  `github.com/wattleflow/documentation` (grana `default`, do `v0.0.4`). Odluka: prihvatiti
  zatečeno stanje, ili povući repozitorij / prepisati povijest prije v1.0.
- [ ] **Put objave za UK English izdanje.** Kad `workflow/en/**` postoji, treba mehanizam koji
  objavljuje **samo** njega — kurirana javna grana ili zaseban javni repozitorij. Do tada je
  push namjerno zaključan, pa objava ne može nastati slučajno.
- [ ] **`documentation/MANIFEST.in` je mrtav.** U tom repozitoriju nema `pyproject.toml` ni
  `setup.py`, pa se ništa ne pakira; datoteka je uz to proturječna (`recursive-include workflow
  *.md` pa `prune workflow`) i sugerira zaštitu koje nema. Obrisati ili opravdati.
- [ ] **Neprefiksirane DR oznake u `workflow/hr/dr/`** — `DR-007-iterator.md`,
  `DR-ORG-05-observable.md`. Sada su praćene, pa se vidi da krše §8 („Neprefiksirana oznaka
  nije valjana"); dodati banner ili preimenovati.

## Audit: vlasništvo i lanac (nalaz 2026-09-09)

Iz rada nad `06_fetch_emails` i izmjene koja je potvrdu po dokumentu premjestila na procesora
([`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md),
**prijedlog** — kod je već izmijenjen).

- [ ] **`DR-WFL-021` t.7 nije proveden za blackboard i repozitorij.** Odluka traži ulazni `INFO`
  u `GenericRepository.write`, `RepositoryWithDriver.write` i `SmallBlackboard.write`; provjereno
  `command grep -rn 'self\.info('` 2026-09-09 — te metode nose **samo `DEBUG`**. Jedini `INFO` u
  `GenericRepository` stoji u `clear()`, što nije korak lanca. Ulazni zapis postoji samo u
  `DriverLocalStorage` i u mail write strategijama. Svjedočanstvo `DR-WFL-021` opisuje pet zapisa
  po dokumentu — kod ih ne daje. Odlučiti: provesti t.7, ili izmijeniti `DR-WFL-021` da prizna
  djelomično prijavljivanje. **Nije posljedica `DR-WFL-028`** — zatečeno je i starije.
- [ ] **`DR-WFL-028` nema svjedočanstvo izvođenja.** Provjere su statičke (grep, `wem_lint`,
  `unittest`, `ruff`); tvrdnja da tok glasi `Start → Processed×N → Completed` traži pokretanje.
  Do tada je oblik toka aspiracija (D-05), a odluka stoji na *prijedlog*.
- [ ] **Kriterij lintera i dalje mjeri po `DR-WFL-018`.** `audit_info_placement` i
  `audit_event_vocabulary` nisu usklađeni ni s `DR-WFL-021` ni s `DR-WFL-028`; `Event.Processed`
  nije u skupu faznih članova granice jedinice, a sloj pipelinea nije prebačen među izuzete.
  Dok traje razmak, C-snimka za `OBS-01/02/03` **ne postoji** — nepromijenjeno od `DR-WFL-021`.
- [ ] **`04-DR/` je gitignoriran u dokumentacijskom repozitoriju.** `.gitignore:15` ignorira cijeli
  direktorij, pa nova odluka (`DR-WFL-028`, 2026-09-09) nastaje **nepraćena**; zatečeni zapisi su u
  indeksu samo zato što su ranije forsirano dodani. `DR-WFL-INDEX.md` §Otvoreno tvrdi da ta serija
  „**smije** biti praćena" jer leži izvan `*/hr/*` — što je točno za putanju, a netočno za pravilo
  koje je doista na snazi. Uskladiti: ili izuzeti `04-DR/` iz `.gitignore`, ili ispraviti tvrdnju
  u indeksu.
- [ ] **`workflow/conformance/` ne postoji.** `CLAUDE.md` §3.1 i `NFRQ-OBS-01` §Verifikacija
  poveznicom upućuju na to stablo; direktorija nema, pa je poveznica slomljena. Uzrok je dosljedan
  (nijedna C-snimka još nije uzeta), ali dokument tvrdi postojanje spremišta koje nema. Ili
  otvoriti stablo s `README` koji kaže da je prazno, ili prepisati poveznicu u tekst.
- [ ] **Dijagram pohrane nije renderiran nakon izmjene.** `FRQ-PRC-15.22-persistence-sequence.puml`
  dobio je audit oznake 2026-09-09; PlantUML na toj platformi nije dostupan, pa je provjerena samo
  struktura. Renderirati pri prvoj prilici.

## Konfiguracija: preset i `write_context` (nalaz 2026-09-09)

- [ ] **`write_context` nema nijednog implementatora.** `GenericProcessor.write_context` vraća
  `{}`, `flush` ga prosljeđuje sve do `strategy.write(**kwargs)`, a `FRQ-PRC-15.22` EV08 ga opisuje
  kao postojeći put — ali **nijedna podklasa ga ne nadjačava**, pa put nikad nije izvršen.
  Mehanizam je dokumentiran i pozvan, nikad dokazan (D-05). Odlučiti: dati mu prvog implementatora,
  ili ga povući i izmijeniti EV08.
- [ ] **`read_content` je mrtav ključ bio u primjeru.** Ne postoji nigdje u kodu (`core`,
  `workflow`, `blackwattle`) — ostatak prije nego što je čitanje sadržaja prešlo na create
  strategiju. Uklonjen iz `06_fetch_emails.yaml` 2026-09-09.
- [ ] **Isti obrazac u tri druga primjera (izmjereno 2026-09-09).** Pretraga svih 19
  `examples/**.yaml` po `ALLOWED` uniji kroz lanac baza, uz priznavanje ključeva koje `__init__`
  potroši prije preseta (imenovani parametar ili `kwargs.pop/get`). Preostaje pet blokova:
  - `06_pii_complex_workflow.yaml` — `tika_timeout` na sva tri `EntityFileDocumentProcessor`-a;
    deklariraju ga `OCRTextProcessor` i `PipelineOCRExtractTika`, dakle **kriv sloj**, ne mrtav
    ključ. Isti oblik kao `skip_*` u `06_fetch_emails`.
  - `08_youtube.yaml` — `namespace` i `agents` na `YoutubeProcessor` (`ALLOWED = ["videos"]`).
    Oba su **mrtva**: `namespace` u `strategies/documents/youtube.py` je tvrdo kodiran
    `Namespace("urn:wattleflow:youtubegraph#")` i ne dolazi iz konfiguracije, a `agents` ne čita
    nitko. Primjer time izgleda kao da rotira user-agente, a ne rotira ih.
  - `24_elasticsearch.yaml` — `index` na `RepositoryWithDriver`; deklariraju ga `DriverElasticSearch`
    i `DriverOpenSearch`, dakle pripada driver bloku.
  Alat: `preset_scan.py` (ad-hoc, u scratchpadu). Kandidat je za `wem_lint` pravilo — konfiguracija
  se danas mjeri tek pri izvođenju, i to samo za blokove koji se doista instanciraju.
- [ ] **`skip_inline`/`skip_types`/`skip_below` bili su na krivom sloju.** Deklarira ih
  `PipelineMailExtractAttachment.ALLOWED` i čita `WriteEmailAttachments`; u primjeru su stajali na
  procesoru, gdje ih `PresetGate` odbacuje uz `warning`. Uklonjeni 2026-09-09 (privici su u tom
  workflowu ionako isključeni). Ako se privici uključe, idu u konfiguraciju pipelinea.
  `NFRQ-ORG-07` je pritom radio kako je propisano — nalaz je konfiguracijski, ne kodni.

## Otvorena pitanja za razgovor

- [ ] Planirani OSCAL katalozi izvan ASD ISM (NIST 800-53, ISO 27001, CIS)?
- [ ] Format metrika za dashboarde (Prometheus, OpenTelemetry, statsd)?
- [ ] Test framework (pytest, unittest, drugo) — odluka prije dokumentacijske faze 4
- [ ] Jedinstveni layout dokumentacije: koji registri ostaju u korijenu, a koji u `workflow/hr/`
