# TODO — wattleflow-workflow

Worklist; organizirano po cijeni izvedbe i riziku. Odvojen od `CLAUDE.md` (policy sloj):
policy nosi normu na snazi, ovaj popis nosi stanje rada. Zatvorena stavka **seli u**
[`DONE.md`](DONE.md) s datumom i provjerom kojom je zatvorena; povijest i dalje nose git i
`DR-WFL` serija, ali zatvaranje mora biti ponovljivo bez čitanja diffa.

> **Revizija 2026-08-24.** Popis je prošao provjeru nad `wattleflow` (core),
> `wattleflow-workflow`, `wattleflow-processors` i `documentation`. Zatvorene stavke su u
> `DONE.md`; stavke koje su djelomično riješene svedene su ovdje **na ostatak**, a ne
> ostavljene u zatečenom opsegu.

## Zahtjevi — pokretanje funkcionalne dokumentacije

- [ ] **Većina FR-ova je u statusu *prijedlog*.** Indeks
  [`02-FRQ/0-FRQ-EN.md`](../02-FRQ/0-FRQ-EN.md) vodi gotovo sve unose kao prijedlog; status se
  mijenja kroz DR, ne prešutno (D-03). Kategorija `FR-ORG` je prazna — vidi §Razlamanje registara.
  Faza 3 iz `CLAUDE.md` §4 do tada nije zatvorena.

## Stanje izmješteno iz policyja (2026-08-16)

Brojke i oznake stanja izašle su iz `CLAUDE.md` po §9/D-13; ovdje im je mjesto.

- [ ] **`@staticmethod`/`@classmethod` migracija (`NFR-ORG-05`, §2.9) — ostatak je `cad`.**
  `wattleflow-workflow` i `wattleflow-processors` su prošli (uz odgođene slučajeve zbog
  `cls`-param iznimke). `wattleflow-cad` nije mjeren istim testom; popis kandidata izvesti
  AST pretragom, ne prepisivati ovdje (§9, D-13).
- [ ] **Odgođeni agregati (`DR-WFL-007`, §2.7) — ostatak.** Na oblik `_EXPORTS` prešli su
  `connections`, `drivers`, `pipelines`, `processors`. Ostaju `strategies` (+ `cryptography`,
  `documents`, `helpers` podpaketi), `blackboards`, `documents`, `oscal`, `repositories` i
  `helpers.converters`. Kvar je latentan — izlazi tek na užoj instalaciji. Brojevno stanje
  vodi `DR-WFL-007` §Otvoreno.
- [ ] **Test maskiranja u lint/CI (`DR-WFL-007` §Otvoreno, prioritet).** Bez njega je §4
  postupak, a ne gate. Izvor popisa opcionalnih biblioteka mora biti distribucijski manifest
  (`DR-WFL-002` §2.2), ne ručni popis u testu.
- [ ] **Sinkronost `_EXPORTS` i stvarnih modula.** Ime u mapi bez podmodula pada tek pri
  pristupu; podmodul bez unosa je nevidljiv. Oboje je AST-provjerljivo i pripada istom
  pravilu kao gornja stavka.
- [ ] **Aspiracije bez mehanizma (§6.2 SIEM, §6.4 observability).** Ostaju označene kao
  aspiracija (D-05) dok se ne odluči transport i format; tada prelaze u zahtjev ili se povlače.
- [ ] **Proširiti §9 zabranu** i na verziju alata i na oznake faza — danas zabranjuje samo
  „brojeve nalaza", pa su verzija linta i `✅/🔄/⏳` prolazili kroz nju. Izmjena policyja → DR.
- [ ] **`tools/README.md` zaglavlje zaostaje za alatom** — navodi `Verzija: 1.14.0`, a
  `wem_lint.__version__` je `1.16.0` (changelog ima unose za 1.15.0 i 1.16.0). Prezentacija
  se versionira odvojeno od kriterija (D-13), ali zaglavlje alata mora pratiti alat.
- [ ] **Vlasništvo `conformance/` direktorija po distribuciji.** Snimka processorsa živi u
  `workflow/conformance/`; `processors/conformance/` ne postoji. Odlučiti: jedan zajednički
  direktorij (pa preimenovati) ili po jedan po distribuciji (pa preseliti snimku).

## Razlamanje registara (2026-08-24)

Registri `HLRQ`/`FRQ`/`NFRQ` razlomljeni su na zapis po zahtjevu u korijenske kategorije
`01-HLRQ/`, `02-FRQ/`, `03-NFRQ/`, po obrascu `kategorija-broj-opis-JEZIK.md` (vidi `DONE.md`).
Ostaje:

- [ ] **Razlamanje je izmjena registra → traži DR (D-03).** Zapis mora pokriti: obrazac imena
  (`kategorija-broj-opis-JEZIK.md`), preseljenje `Zajedničkih definicija` u `NFR-DEF-01`,
  povelje `[M]` u `NFR-DEF-02`, Dodatka A u `NFR-APX-01`, i **uvođenje EN izdanja prije v1.0**
  (vidi sljedeću stavku). Do zapisa je razlamanje **zatečeno stanje**, ne odobrena norma.
- [ ] **EN izdanje postoji prije v1.0 — uskladiti s `CLAUDE.md` §3.2 i §8.** Policy kaže da
  doktrinarni tekst ostaje hrvatski **do v1.0**, a EN izdanje je **zaseban, namjeran čin
  objave**. Sada u stablu stoji `-EN.md` za cijeli sloj zahtjeva. Odlučiti: (a) proširiti §3.2
  da dopusti dvojezični registar uz HR kao autoritativan, ili (b) povući EN datoteke do v1.0.
  Do odluke EN nosi banner „HR je autoritativan" — što je deklaracija, ne odobrenje (D-05).
- [ ] **Dvojezičnost je nova prilika za drift (D-12/D-13).** HR i EN nose istu normu u dvije
  datoteke; nijedan alat ne provjerava da su u koraku. Kandidat: lint koji uspoređuje strukturu
  (broj kriterija, oznake, statusi) para `-HR`/`-EN`. Do tada deklarirana slijepa pjega (D-11).
- [ ] **`NFR-ORG-07` nema unos, a mjeri se kao `error`.** `preset_allowed_declaration` ruši build
  u objema distribucijama, a `NFR-ORG-08` i `NFR-SEC-06` k.1 ga citiraju kao izvor. Napisati
  `NFR-ORG-07-input-surface-{HR,EN}.md` — traži DR (D-03).
- [ ] **`FR-ORG` kategorija je prazna.** Zatečeni registar nosio je samo zaglavlje
  `## FR-ORG-01 — ...`; razlamanje ga je zamijenilo deklariranom rupom u `02-FRQ/0-FRQ-EN.md`.
  Popuniti po ISO/IEC/IEEE 29148 ili povući kategoriju.
- [ ] **Preostale kategorije bez naslijeđene strukture.** `04-DR` nosi samo `DR-PRC-001` dok
  `DR-WFL` serija (21 zapisa) i `DR-COR` serija još žive u `workflow/dr/` odnosno
  `workflow/core/dr/`; `05-METHOD/dqi.md` ne slijedi obrazac imena (`METHOD-01-dqi.md`);
  `06-ANALYSIS` i `07-CHANGES` su datumski, što je vlastiti obrazac i treba ga potvrditi ili
  uskladiti. Seljenje `DR-WFL` serije lomi ~40 referenci pa traži isti postupak kao prva stavka.

## Nalazi konsolidacije (2026-08-24)

- [ ] **`02-FRQ/` datoteke su preimenovane u `FRQ-*`, a zahtjev se i dalje zove `FR-*`.** Ime
  datoteke sada nosi ime *registra*, ne *zahtjeva* — različito od `03-NFRQ/NFR-*` i od naslova
  unutar samih datoteka (`# FR-CON-13.1 — …`). Poveznice su 2026-08-24 poravnate s datotekama
  koje stvarno postoje; odlučiti koji je obrazac norma (`FR-*` kao u zaglavljima, ili `FRQ-*`)
  i provesti ga u jednom potezu — to je izmjena obrasca imena, dakle DR (D-03).
- [ ] **`D-09` i `D-10` su na snazi, a izvor im je `DR-COR-014` u statusu *predložen*.** Norma
  na snazi izvedena iz neprihvaćene odluke je nalaz pod D-02 (`DOCTRINE` §Bilješke t.2).
  Prihvatiti `DR-COR-014` ili prekvalificirati članke.
- [ ] **Kodifikacija doktrine nema DR zapis.** Bivše `DR-018 (kandidat)` bilo je izvor gotovo
  svakom članku; zapis nikad nije otvoren. Isto vrijedi za povlačenje osam članaka
  2026-08-24 (v0.1 → v0.2). Deklarirano u `DOCTRINE` §Bilješke t.1; otvara se kad dođe red na
  reviziju DR serija.
- [ ] **Dopuna doktrine čeka konsolidaciju sloja zahtjeva.** Registar je namjerno malen
  (9 članaka); novi ulazi tek uz mehanizam i signal. Kandidati koji su pali na tom testu vode
  se u tablici *Povučeni članci* — ne brišu se iz vidokruga, čekaju nositelja.
- [ ] **`FILOZOFIJA.md` (416 linija, HR) i `PHILOSOPHY.md` (271, EN) su se razišli.** Nije
  prijevod nego dvije verzije istog teksta; §3.2 kaže da HR vlada. Odlučiti koja je izvor pa
  drugu svesti na prikaz ili stub — isti postupak kao za `LITERATURA`/`POSTULATI`.

## Usklađivanje dokumentacije (nalaz 2026-07-30, revidiran 2026-08-24)

Nalazi iz analize `PHILOSOPHY` / `METHODOLOGY` / `DOCTRINE` / `FR` / `NFR`. Sve su izmjene
registara → idu kroz DR (D-03).

- [ ] **Akronimi — `DR-WFL-004` je i dalje otvoren.** Kriteriji sada **nose suspenziju**
  (`NFR-ORG-02` §4, `NFR-ORG-03` k.4 — vidi `DONE.md`), pa metoda više ne propisuje
  neodlučeno. Ostaje sama odluka: `PDF` vs `Pdf`. Do nje pravilo stoji na `WARNING` uz
  deklarirani waiver i **ne provodi se** masovnim preimenovanjem (D-02).
- [ ] **`ADR` ostaci izvan registra zahtjeva.** `DOCTRINE.md` i `METHODOLOGY.md` su čisti
  (provjereno 2026-08-24); ostaju spomeni u `DR-COR-*` „Prijelaz s prethodnih oznaka" tablicama,
  što je dopuštena povijesna referenca — `ADR` je zabranjen samo za **nove** zapise (§8).
- [ ] **`DR-WFL-002` §5, zadnja natuknica:** „Umbrella distribucije (**budući `DR-WFL-003`**)"
  — zaostatak predrename numeracije. `DR-WFL-003` je čuvana opcionalna ovisnost; registar
  umbrella distribucija nema zapisa ni kandidata. Preimenovati u „budući DR" bez broja i
  otvoriti kandidata, ili povući stavku.
- [ ] **Test maskiranja na razini paketa nije ničiji kriterij.** `NFR-SEC-03` kriterij 1 sada
  ga navodi kao metodu verifikacije **na razini modula** (`DR-WFL-003`), ali paketna razina
  (`DR-WFL-007` §4, §2.7 t.3 — *uvozivost paketa pod maskiranim opcionalnim tierom*) nema
  kriterij, pa je `wem_lint` ne mjeri. Kandidat: novi kriterij u `NFR-SEC-03` ili `NFR-SEC-02`
  (import-time closure je učitani kod = napadna površina). Traži DR (D-03).
- [ ] **Agregatni rub nestaje iz AST-a (`DR-WFL-007`).** `dictionary.yaml` već bilježi da
  ORG-01 fan-in ne razrješava agregatne uvoze pa je donja granica; s `__getattr__` agregatom
  ruba nema ni u AST-u — mapa `_EXPORTS` je jedini statički trag, a `wem_lint` je ne čita
  (`grep _EXPORTS tools/wem_lint.py` je prazan). Bez toga `drivers`/`connections`/`processors`
  postaju nevidljivi grafu ovisnosti.
- [ ] **`PHILOSOPHY.,md`** (zarez u nazivu datoteke) — pojave u `POSTULATE.md`,
  `workflow/hr/{POSTULATI,LITERATURA,METHODOLOGIA}.md`, `DR-WFL-002`, `DR-WFL-003`,
  `workflow/changes/2026-07-28-…`. Nijedna ne razrješava u postojeću putanju.
- [ ] **Prebrojati unakrsne reference na `METHODOLOGIA`.** Dokument danas ima §1–§10 (+5.1,
  7.1) — §11 iz starijih referenci ne postoji. Reference provjeriti pretragom, ne prepravljati
  napamet.
- [ ] **`POSTULATE.md` / `POSTULATI.md` zaglavlje** navodi nadređene v0.4 / v0.3 → v0.4.1 /
  v0.3.1 (i `PHILOSOPHY.,md` → `PHILOSOPHY.md`, `dictionary.json` → `dictionary.yaml`).
- [ ] **Kolizija DR oznaka u nacrtima:** `core/dr/DR.md` i `workflow/concrete/DR.md` nose
  neprefiksirani `DR-001` sa statusom `proposed` (indeks ih vodi kao nacrte, jedan
  „sadržajno proturječan") → staviti banner „superseded" u same datoteke.
- [ ] **`DR-WFL-006` vs kod:** odluka pina `wattleflow>=0.0.0.46`, `workflow/pyproject.toml`
  ima `>=0.0.0.50` → zabilježiti kao verziju zapisa (D-03).
- [ ] **Nedovršene doktrinarne tvrdnje:** (a) `DOCTRINE` tvrdi da proza citira norme
  D-oznakama — nijedna D-oznaka nije u `PHILOSOPHY`/`METHODOLOGIA`; (b) `DOCTRINE`:242
  koristi **H4-DQI** kao doktrinarnu hipotezu, a `PHILOSOPHY` §Hipoteze ima samo H1–H3.
  Metoda je H4 već deklarirala kao kandidata (`processors/method/dqi.md`) — uskladiti
  doktrinu s tim ili upisati H4 u registar hipoteza.

## Metode

- [ ] **Registar metoda ne postoji**, pa dokument metode nema oznaku i referira se putanjom.
  Uz registar ide i izdvajanje drugog referentnog primjera (`METHODOLOGIA` §10, wem_lint) —
  taj pripada ovoj distribuciji, za razliku od DQI-ja.
- [ ] **DQI nalazi pripadaju `wattleflow-processors`** — metoda i implementacija su tamo
  (`processors/method/dqi.md`, `pipelines/quality/dqi.py`). Prenijeti u worklist te
  distribucije kad ga dobije. Ostatak nalaza: implementirana je **jedna dimenzija od šest**,
  `dqi.py` nema modulski `__all__` (paketni `quality/__init__.py` ga ima), a V1–V6 protokol
  valjanosti stoji na *otvoreno*. (Uvoz modula više ne puca — `helpers.dtime` je razriješen.)

## Veće (značajna cijena ili arhitektonske odluke)

- [ ] **Migracija tip-hintova na PEP 585/604 — ostatak je jezgra.** `wattleflow-workflow` je
  čist. Ostaju `core/concurrent.py` i `core/transactional.py` (`Optional`, `Tuple`, `Dict`) —
  dira autoritativni core (§2.5 → `DR-COR`). Za `wattleflow-processors` vidi zaseban odjeljak.
- [ ] Implementirati SIEM forwarding (`audit/` prazan; iskoristiti `AsyncHandler` +
  `Audit.subscribe_handler()`)
- [ ] OSCAL katalogizacija — `component-definition`, `assessment-results`, dodatni katalozi
- [ ] Observability — Prometheus / OpenTelemetry za FSM tranzicije i throughput (uz V1–V6)
- [ ] **Supply-chain za ne-core distribucije (`NFR-SEC-03` kriterij 4).** Hash-pinned lock +
  SBOM (CycloneDX/SPDX) po distribuciji; SBOM validator vođen core politikom. Razdvajanje
  specijalizacija je izvršeno (vidi `DONE.md`), ovo je njegov neisporučeni dio.
- [ ] **Self-integritet + shadowing gate (`NFR-SEC-03` kriterij 5).** Za izgrađene artefakte
  verificirati wheel `RECORD` (per-file `sha256`) — ne raditi vlastiti digest-format. Za
  dev/editable stabla `RECORD` je prazan → `wem_lint` digest-scan (`FileDigest`) koji
  istodobno detektira namespace-sjenčanje/koliziju. Alat danas `RECORD` samo spominje u
  komentaru; pravila nema.
- [ ] **Dovršiti PEP 420 namespace migraciju — ostatak su primjeri.** Stabla su migrirana
  (vidi `DONE.md`). Preostaje: (1) `examples/processors/*` — **11** datoteka i dalje uvozi
  agregat `from wattleflow.helpers import …` (02, 03, 04, 06, 07, 08, 09, 10, 11, 12, 13) uz
  ručno razrješavanje nepostojećih simbola (`FileType`→`wattleflow.constants`,
  `LocalPath`/`Preset`); (2) `examples/todo/*` je dijelom pre-broken — počistiti ili
  arhivirati. Popis preseljenja imena: `Attribute` → `concrete.helpers`,
  `TempPathHelper`/`Project` → `helpers.system`, `Normaliser` → `helpers.normaliser`,
  `TextStream` → `helpers.streams`, `TextMacros` → `helpers.macros`, `FileType` →
  `constants.filetype`, `ConfigAdapter` → `helpers.config_adapter`, `Config` → `YAMLConfig`
  iz `helpers.config_yaml`, `ProcessorMemento` → `GenericMemento` (`concrete.memento`),
  `DataFrameDocument` → `documents.dataframe`. Primjeri nisu pod verzijskom kontrolom i
  third-party stack nije instaliran, pa izvršne provjere nema — samo razrješavanje imena.
  Memorija: `helpers-pep420-namespace`.
- [ ] **Testovi** — odgođeno do odluke o test frameworku (§4). Provjereno 2026-08-24: nijedna
  distribucija nema `tests/` ni `[tool.pytest]` blok.

## `__slots__` i MRO (nalaz 2026-06-29, djelomično riješeno)

Klasa koja spaja `blackboard` i `originator` obitelj ne linearizira MRO ako korijenska baza
ne prethodi generičkom parametru. Riješeno Fixom B — preslagivanjem redoslijeda baza u
`concrete/`, bez diranja corea (§2.5). Redoslijed baza je zato **ograničenje, ne stil**;
komentar uz klasu to čuva na mjestu.

- [ ] **Fix A (niži prioritet)** — alternativa: uskladiti **core** `IOriginator` na
  `(Generic[State], IWattleflow, ABC)`, tj. isti redoslijed na sučelju `originator` obitelji.
  Danas je `core/behavioural.py:217` `(IWattleflow, Generic[State], ABC)`. Dira autoritativni
  core (§2.5 → `DR-COR`) i ima širi domet; razmotriti ako se pojavi još C-builtin/MRO sudara.

## OSCAL crosswalk — čeka compliance sign-off

`oscal/crosswalk.py` prepisuje control-id iz izvorne taksonomije (NIST) u ciljnu (ISM) prije
provjere; unmapped ID-evi prolaze nepromijenjeni. Mapping je **kuriran compliance artefakt** —
ne izmišljati mapiranja.

- [ ] **Mapiranja su `PROPOSED`** (`resources/crosswalk/nist-sp800-53_to_asd-ism.json`,
  `status: proposed-requires-compliance-review`, 2026-05-31) i traže sign-off prije
  oslanjanja. Postgres deklaracija: `ac-3→ism-0445`, `ia-5→ism-1401` (E8 ML1);
  `sc-8→ism-0469`, `sc-13→ism-1080` (izvan Essential Eight opsega — protiv E8 baselinea
  ispravno padaju).

## Konzistentnost i standardi

- [ ] **Objava preimenovanog paketa.** Kod i dokumentacija nose `wattleflow-processors`;
  preostaje uskladiti PyPI i GitHub opise (nije provjerljivo iz repozitorija).

### Refaktoring: migracija tip-hintova u `wattleflow-processors`

Provjereno 2026-08-24: **103** datoteke s `Optional`/`Union`/`List[`/`Dict[`/`Tuple[`.

1. Uvesti `from __future__ import annotations` gdje nedostaje.
2. `pyupgrade --py311-plus` (ili `ruff check --select UP --fix`), **jedan commit po pod-paketu**.
3. Ručna provjera runtime-evaluiranih anotacija (Pydantic/dataclass/`get_type_hints`).
4. `ruff`/`mypy` gate u CI nakon migracije.

### Tika kao connection/driver + driver-kanal za create-strategije (nalaz 2026-06-26)

**Kontekst.** `EntityFileDocumentProcessor` ekstrahira sadržaj preko Tike i prosljeđuje
`content=` u `blackboard.create`. Arhitektonski pogrešno: ekstrakcija je odgovornost
create-strategije, ne procesora. (Od tada je ekstrakcija barem **opt-in** — `extract: true`
— pa procesor po zadanom više ne zove Tiku; sama arhitektura nije promijenjena.)

**Odluka korisnika (2026-06-26): odgođeno.** Smjer (ne implementirati bez dogovora):

- [ ] **ConnectionTika + DriverTika** (OSCAL obavezan). Server-management (jar staging, Java
  provjera, timeout) seli iz procesora u `ConnectionTika`.
- [ ] **CreateTextDocument** koristi `DriverTika` za ekstrakciju iz `filename`;
  `CreatePdfDocument` (fitz) ostaje za PDF.
- [ ] **EntityFileDocumentProcessor** prestaje zvati Tiku; prosljeđuje samo `filename`.
- [ ] **Otvoreno pitanje — driver-kanal za create-strategije.** Create-strategije nemaju
  pristup driveru (write-strategije ga imaju preko `kwargs.get("driver")`). Smjer: isti
  mehanizam; izvedba dira `concrete/` (§2.5 — tražiti odluku).

**Interim:** `tika_timeout` default 300s; suvišan Tika poziv u 04 ostaje do refaktora.

## Deduplikacija koda — DRY refaktor (nalaz 2026-06-27, ČEKA DR)

**Status:** analiza dovršena; implementacija **zaustavljena na zahtjev korisnika** dok se ne
izradi DR (arhitektura + NFR) koji vodi sustavni pristup. Ne dirati kod do tada.

> **D1 je riješen** zajedno s reorgom pipelinea (2026-07-06) — vidi `DONE.md`. **Otvoreno iz
> tog reorga:** pixel-apply write-strategija `WriteReductedPNG` nije implementirana —
> `PipelineReductSpans` piše spanove u metadata.

| # | Klaster | ~Pojava | Lokacije (uzorak) |
|---|---|---|---|
| D2 | Tipovi (Tokens/RedactBoxes/…) | 3 modula | pdf/png pipelines + `strategies/documents/pdf.py` |
| D3 | Strategy `execute()` try/except→`StrategyException` | ~41 | sve `strategies/documents/*` |
| D4 | Driver resolution (3 varijante) | ~21 | kanon: `strategies/documents/pdf.py _resolve_driver` |
| D5 | `isinstance(caller/facade…)` preambule | ~38 | sve strategije (osim `pdf.py` → `Attribute.evaluate`) |
| D6 | Metadata pečat `created_by/at`,`stored_by/at` | ~21 | create/write strategije |
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

- [ ] **Helper/Facade:** `helpers/system.py += require_module(name, pip=…)` → **D7**;
  `strategies/_support.py resolve_driver()` + `stamp_created/stored()` → **D4, D6**
  (provjereno 2026-08-24: nijedan od njih još ne postoji)
- [ ] **Dekoratori:** `@strategy_guard` → **D3** (alternativa u core `GenericStrategy.call()`
  — §2.5, traži `DR-COR`); `@wrap_errors(DriverXxxError)` → **D8**
- [ ] **Mixini/baze:** `RecordDocument` → **D10**; `GraphDocumentMixin` → **D11**;
  `HttpJsonDriverMixin` → **D12**; `FileSourceMixin` → **D13**
- [ ] **Tipovi/konstante:** centralni `pipelines/types.py` → **D2**; `*_SUFFIX` iz `FileType`
- [ ] **D9:** ostaviti eksplicitne klase (greppabilnost), riješiti koliziju `KafkaConnectionError`

**Preostali bug uočen usput:** `stored_at` je nedosljedan — `strategies/documents/json.py:268`
koristi `Now.utc()`, sve ostale write-strategije `document.utc_time_stamp()`. (Ostala tri buga
iz izvornog nalaza su ispravljena — vidi `DONE.md`.)

**Granice (§2.5):** `@strategy_guard` u coreu traži odluku korisnika; third-party helperi (OCR)
**moraju** u processors paket, lazy (§7.4). Redoslijed: (1) pipelines, (2) strategije, (3) driveri.

## NFR-ORG-01 / SEC-03 nalazi (wem_lint)

Stanje vektora se ne prepisuje ovdje (§9, D-13) — pokreni `tools/wem_lint.py --snapshot` i
referiraj snimku u `documentation/workflow/conformance/`. Nalazi nad clean core stablom su
zatvoreni (vidi `DONE.md`); ovdje ostaje `wattleflow-processors`.

- [ ] **`wattleflow-processors` nosi zatečene warninge** (`OBS-01/02/03` audit zapis,
  `ORG-01` neiskorišteni dijeljeni helperi, `ORG-02` package-alias i pipeline-gramatika,
  `ORG-03` typevar-role, `SEC-03` foreign-import i manifest) — snimka
  `2026-08-23-processors.json`. Vektor je zelen jer nijedno pravilo nije `error`; worklist je
  sadržaj snimke, ne prepis brojki ovamo.

## Konfiguracijski moduli (stanje 2026-08-19, uz `DR-WFL-012`)

- [ ] **Je li `DR-WFL-003` još živ?** Odluka o čuvanoj opcionalnoj ovisnosti nema više
  nijednog nositelja u ovoj distribuciji (`scope.guarded_optional` je prazan, potvrđeno i u
  `NFR-SEC-03` kriteriju 1). Povući je ili je zadržati kao mehanizam za buduće slučajeve —
  kroz DR, ne šutnjom.
- [ ] **`get()` nosi dvije nespojive semantike** u konfiguraciji i adapteru; dok se ne
  usklade, nije dio ugovora `IConfig` (`DR-COR-016` §Otvoreno).
- [ ] **`DR-WFL-017` je prijedlog, a kod je već izveden.** `wattleflow.enums`, `constants`,
  `decorators` i `helpers` su PEP 420 imena bez `__init__.py` u objema distribucijama; uvoz
  je eksplicitan submodul. Zapis je i dalje **Prijedlog (nacrt, 2026-08-22)**. Prevesti ga u
  *prihvaćen* ili ga odbaciti — do tada registar nosi dvije nespojive tvrdnje o vlasništvu
  imena (`DR-WFL-016` t.2).
- [ ] **`DR-WFL-007` ne spominje dijeljeno ime.** `CLAUDE.md` §2.7 t.4 propisuje eksplicitan
  submodul za cross-distribucijski uvoz, ali zapis na koji upućuje bira samo između eager i
  odgođenog agregata — treći slučaj (*agregata nema jer ga nijedna distribucija ne posjeduje*)
  ondje nije zapisan. Dopuniti (`DR-WFL-017` §Otvoreno t.2).
- [ ] **`wattleflow-processors` nosi `constants/keys.py`** — ime koje jezgra posjeduje, a
  njegov kod ga ne uvozi (`grep` nalazi samo vlastito zaglavlje modula). `MANIFEST.in` ga
  šalje u sdist. Odlučiti brisanje. (`constants/errors.py` je obrisan — vidi `DONE.md`.)
- [ ] **Repovi `DR-WFL-016` (mrtvi vokabular).** Preseljen je, ne obrisan — simboli danas
  žive u `processors/enums/{audit,pipeline}.py`: `ConnectionStatus`, `EventLog`,
  `ProtectiveMarkings`, `WattleflowOSCAL`, `PipelineAction`, `PipelineType`,
  `ProvenanceHandler`. Odlučiti brisanje. `Event.Classification` ostaje u jezgri
  (`workflow/enums/event.py:38`) iako sam pojam klasifikacije više nije ondje — pregledati
  kad se `Event` bude čistio.
- [ ] **Rep `DR-WFL-015`:** `wattleflow-oscal` treba povući s PyPI-ja ili označiti napuštenim
  dok kolizija imena ne prestane biti moguća. (Ostala tri repa su zatvorena — vidi `DONE.md`.)
- [ ] **Ostaci u dokumentacijskom stablu.** `hr/METHODOLOGIA copy.md` (stariji nacrt,
  slomljene relativne poveznice, još govori o „ADR"), `hr/METHODOLOGIA.md.superseded`,
  `hr/FILOZOFIJAmd.superseeded` (nosi *Bilješku o sintezi* koje u `FILOZOFIJA.md` nema),
  `README.bak` i `OLD-TODO.md`. Odlučiti: prenijeti sadržaj pa obrisati.

## Verzijska kontrola i objava dokumentacije (nalaz 2026-08-03)

Praćenje je riješeno (§8): `documentation` prati cijeli sadržaj, push na `origin` je
onemogućen (`git remote -v` → `DISABLED-local-only-repository`). Preostaje ono što praćenje
ne rješava:

- [ ] **Hrvatski doktrinarni tekst je već javan na GitHubu.** `.gitignore` je štitio samo
  `*/hr/*`, pa su `DOCTRINE.md`, `POSTULATE.md`, `dictionary.yaml`, cijela `workflow/dr/`
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
- [ ] **Serija `DR-PRC` nema indeks.** `documentation/processors/dr/` nosi
  `DR-PRC-001-model-access-boundary.md`, a `DR-WFL-INDEX.md` vodi samo WFL seriju —
  deklarirana rupa (D-11, `CLAUDE.md` §8).

## Otvorena pitanja za razgovor

- [ ] Planirani OSCAL katalozi izvan ASD ISM (NIST 800-53, ISO 27001, CIS)?
- [ ] Format metrika za dashboarde (Prometheus, OpenTelemetry, statsd)?
- [ ] Test framework (pytest, unittest, drugo) — odluka prije dokumentacijske faze 4
- [ ] Jedinstveni layout dokumentacije: koji registri ostaju u korijenu, a koji u `workflow/hr/`
