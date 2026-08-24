# FR-DRV-13 — Driver za jezični model

> **Oznaka je provizorna.** Kategorija `DRV` nije u vokabularu FR registra; uvođenje traži DR
> (D-12).

| | |
|---|---|
| **Status** | Prijedlog (2026-08-20) |
| **Nadređeni zahtjev** | [`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) — narativ i poslovna pravila `BR-01…BR-06` |
| **Predmet** | `DriverLanguageModel` — perzistencijske operacije nad modelom: **`read` / `write` / `update` / `download`** |
| **Sestrinski** | [`FR-CON-13.1`](FRQ-CON-13.1-huggingface-connection.md) i [`FR-CON-13.2`](FRQ-CON-13.2-remote-model-connection.md) — pristup koji ovaj driver koristi |
| **Zatečeno** | Ne postoji. Presedan oblika: `DriverClaude` (`load`/`close`/`read`/`write`/`complete`) za udaljeni model; od `CHG-PRC-2026-08-20-01` ondje su pristupni parametri odvojeni od parametara poziva (`ACCESS` / `CALL`), pa je preseljenje pristupa iza konekcije mehaničko |

## 1. Predmet

Driver je **jedino** mjesto koje dodiruje model. Nad pristupom koji drži konekcija izvodi četiri
operacije; koja je od njih moguća ovisi o konekciji, ne o pozivatelju (`BR-02`).

| operacija | značenje za jezični model |
|---|---|
| **`download`** | dohvat modela u lokalni cache kad ga ondje nema, a konekcija je mrežna (`BR-03`) |
| **`read`** | dobavljanje izlaza iz modela za predani sadržaj — inferencija |
| **`write`** | predaja sadržaja modelu i pohrana rezultata prema ugovoru pozivatelja |
| **`update`** | osvježenje lokalne kopije (nova revizija) ili stanja učitanog modela |

Model je učitan **jednom po driveru** i dijeljen; oslobađa se kad workflow završi. Time se
uklanja zatečeno stanje u kojem svaka instanca pipelinea drži vlastitu kopiju.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Konfiguracija (YAML) — registrira driver i veže ga na imenovanu konekciju | `type`, `name`, `configuration` (model, revizija, uređaj, cache) |
| **A2** | `Workflow` / `Processor` — traži pokretanje drivera (`BR-04`) | ime registriranog drivera |
| **A3** | `DriverManager` — vlasnik registriranih drivera | registar imenovanih drivera |
| **A4** | Konekcija — lokalna ili mrežna, isti ugovor ([`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) §4) | stanje pristupa |
| **A5** | `DriverLanguageModel` — predmet ovog zahtjeva | operacije nad modelom |
| **A6** | `Pipeline` — konzument (`BR-05`) | sadržaj dokumenta |

Model **nije akter**: ne pokreće ništa, ulazi kao sudionik i kao izvor kvara.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | Faktorija gradi workflow → registrira driver i veže ga na konekciju (`BR-01`) |
| **EV02** | A2 traži od A3 pokretanje drivera — prije obrade prvog dokumenta |
| **EV03** | A6 poziva `read` / `write` tijekom transformacije |
| **EV04** | Status modela je *nedostaje* ili *nepotpun* → dohvat, ako je dopušten (`BR-03`, `BR-08`) |
| **EV05** | Dohvat traje → istekao interval izvještavanja o napretku (`BR-09`) |
| **EV06** | Workflow završava → oslobađanje modela |

## 4. Preduvjeti

1. Driver je registriran i veže se na **postojeću** imenovanu konekciju (`BR-01`).
2. Konekcija je u stanju *povezan* (`FR-CON-13.1` §5 ili `FR-CON-13.2` §5). Driver **ne zna**
   koju je dobio — vidi samo zajednički ugovor.
3. Konfiguracija imenuje model i, gdje je primjenjivo, reviziju, uređaj i cache.
4. Runtime koji driver koristi je dostupan — inače je to kvar iz §6, ne izuzetak usred obrade.

## 5. Normalan tok

**Provjera na pokretanju (`BR-07`).** Sve do koraka 5 događa se **prije** obrade prvog dokumenta.

1. EV01 — faktorija instancira driver, veže ga na konekciju i registrira pod imenom.
2. EV02 — A2 traži pokretanje preko `DriverManager`-a; driver provjerava spremnost:
   runtime dostupan, konekcija povezana, **status modela** u lokalnom spremištu.
3. Status je jedan od tri: *prisutan* (i tražena revizija odgovara) · *nedostaje* ·
   *nepotpun*. Provjera je jeftina i ne dohvaća sadržaj.
4. Ako model nedostaje ili je nepotpun (EV04):
   - konfiguracija dopušta dohvat (`BR-08`) → driver pokreće **postojeći okidač knjižnice**
     (`from_pretrained` / `snapshot_download`), uz izvještaj o napretku (`BR-09`);
   - dohvat nije dopušten → dohvat se zabranjuje (`local_files_only` / `HF_HUB_OFFLINE`) i kvar
     nastupa prije obrade (`BR-06`).
5. Driver učitava model jednom (uređaj po konfiguraciji) i objavljuje stanje *spreman*.
6. EV03 — A6 predaje sadržaj: `read` vraća izlaz modela, `write` predaje sadržaj i pohranjuje
   rezultat prema ugovoru pozivatelja.
7. Rezultat se vraća pipelineu; pipeline radi transformaciju i ne zna gdje model živi.
8. EV06 — driver oslobađa model i podređene resurse.

**Normalan tok je postignut kad je pristup uspostavljen, model dostupan i transformacija
izvršena.**

### Okidač je knjižnicin, ne naš

Dohvat na zahtjev već postoji u `transformers` / `huggingface_hub`: `from_pretrained` provjeri
cache i preuzme što nedostaje. Driver taj mehanizam **ne replicira** — on ga samo **pomiče na
početak** workflowa i čini vidljivim:

| što driver dodaje | čime |
|---|---|
| rani okidač umjesto lijenog | poziv u EV02, prije obrade prvog dokumenta (`BR-07`) |
| jeftin status bez dohvata | `try_to_load_from_cache(repo_id, filename, cache_dir, revision)` |
| zabrana tihog dohvata | `local_files_only=True` / `HF_HUB_OFFLINE` kad `BR-08` ne dopušta |
| vidljiv napredak | `snapshot_download(..., tqdm_class=…)` premošten u audit zapis (`BR-09`) |
| pinana revizija | `revision=` iz konfiguracije, bez tihe zamjene |

Pipeline zato pri pokretanju zatiče **lokalni** model — preuzet ili zatečen — i nikad ne pokreće
dohvat sam.

## 6. Alternativni tokovi

Po zabludama distribuiranih sustava (Deutsch 1994; 8. Gosling) — razredi kvarova koje driver
mora razlikovati, jer se različito rješavaju.

| # | zabluda | slučaj u ovom driveru | ponašanje |
|---|---|---|---|
| 1 | mreža je pouzdana | preuzimanje pukne na pola; djelomičan snapshot u cacheu | nepotpun dohvat se ne smije prikazati kao uspjeh; nastavak ili čišćenje, pa kvar |
| 2 | latencija je nula | inferencija ili dohvat visi | timeout po operaciji, konfigurabilan; istek = kvar s razlogom |
| 3 | propusnost je beskonačna | model je 300 MB – 1,3 GB | dohvat je vidljivo stanje (napredak, veličina), a offline režim ga zabranjuje umjesto da čeka |
| 4 | mreža je sigurna | preuzeti sadržaj je izvršni format (`pickle`) | integritet se provjerava (revizija/sha); prednost `safetensors` |
| 5 | topologija je nepromjenjiva | model povučen ili preimenovan kod pružatelja | kvar imenuje traženi model i reviziju |
| 6 | postoji jedan administrator | cache dijeli drugi proces; istovremeni dohvat istog modela | dohvat je zaključan ili idempotentan; nema dvije polovične kopije |
| 7 | trošak transporta je nula | naplata po tokenu, kvota, ograničenje brzine | kvota je vlastiti razred kvara i ne ponavlja se slijepo |
| 8 | mreža je homogena | verzija runtimea/formata ne odgovara modelu; nema CUDA uređaja | nesukladnost se prijavljuje; pad na `cpu` je **odluka konfiguracije**, ne tiha zamjena |

| ostali uvjet | ponašanje |
|---|---|
| driver nije registriran ili veže nepostojeću konekciju | workflow **ne započinje** (`BR-01`, `BR-06`) |
| runtime nije instaliran | kvar prije obrade, s uputom što nedostaje (`BR-06`) |
| model nedostupan i nije dohvatljiv | kvar prije obrade (`BR-06`) |
| pipeline traži operaciju koju konekcija ne podržava | odbija se s razlogom, ne pokušava zaobilazno |
| ponovno pokretanje već spremnog drivera | idempotentno — model se ne učitava drugi put |
| model nedostaje, a dohvat nije dopušten | `LocalEntryNotFoundError` / `OfflineModeIsEnabled` → kvar prije obrade, s uputom što konfigurirati (`BR-08`) |
| repozitorij ili revizija ne postoje | `RepositoryNotFoundError` / `RevisionNotFoundError` → kvar imenuje traženo, bez zamjene |
| model je pod uvjetima pristupa (gated) | `GatedRepoError` → razred *prava*, ne *dosežnost*; traži prihvat uvjeta i token |
| dohvat prekinut ili prekoračio dopušteno trajanje | status ostaje *nepotpun*; nepotpun sadržaj se ne prikazuje kao spreman |
| napredak se ne mijenja dulje od intervala | izvještaj i dalje ide (proteklo vrijeme), da se zastoj razlikuje od tišine (`BR-09`) |

## 7. Rezultat

Model je dostupan i dijeljen kroz imenovani driver; pipeline transformira sadržaj bez znanja o
lokaciji modela i načinu učitavanja. Svaki kvar pristupa nastupa **prije** obrade prvog
dokumenta i nosi razred koji kaže što treba popraviti.

## 8. Kriteriji prihvaćanja

1. Driver izlaže `read` / `write` / `update` / `download`; učitavanje modela nije vidljivo
   pozivatelju.
2. Model se učitava **jednom po driveru** i dijeli između pipelinea (`BR-04`, `BR-05`).
3. Spremnost i **status modela** provjeravaju se prije obrade prvog dokumenta; neuspjeh
   zaustavlja workflow (`BR-06`, `BR-07`).
4. `download` je deklarirano stanje s poznatim ishodom; nepotpun dohvat nikad ne prolazi kao
   uspjeh (`BR-03`).
5. Kvarovi nose razred (runtime · dosežnost · autentikacija · kvota · integritet ·
   nesukladnost), ne samo poruku.
6. Pad na drugi uređaj ili drugi model ne događa se tiho — samo po konfiguraciji.
7. Dohvat se izvodi samo uz izričito dopuštenje (`BR-08`), i dok traje, zapis nosi napredak
   u konfiguriranom intervalu (`BR-09`) — čekanje je razlučivo od zastoja.
8. **[M]** Broj izravnih poziva učitavanja modela (`spacy.load`, `from_pretrained`,
   `GLiNER.from_pretrained`, …) u paketu `pipelines` je **0** (zatečeno: 8).

## 9. Verifikacija

| kriterij | metoda |
|---|---|
| 1, 2, 6 | pregled sučelja i konfiguracije |
| 3–5, 7 | test (po odabiru test-frameworka; do tada **pregled**) |
| 8 | analiza — AST skan, kandidat za `wem_lint` pravilo |

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-13` §6](../01-HLRQ/HLRQ-13-llm-models.md#6-nefunkcionalni-zahtjevi-i-oscal).
Ovdje samo ono što je specifično za driver.

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFR-SEC-01` | model učitan jednom i dijeljen je **hub**: dobitak u memoriji plaća se koncentracijom. Oslobađanje na kraju workflowa (§5 t.7) je dio te cijene |
| `NFR-SEC-02` | javna površina su četiri operacije; učitavanje, uređaj i cache ostaju unutarnji (§8 k.1) |
| `NFR-SEC-03` | preuzeti model je third-party artefakt: težine u `pickle` formatu su izvršni sadržaj — prednost `safetensors`, uz provjeru revizije/sha (§6 zabluda 4) |
| `NFR-SEC-06` | driver vidi **sadržaj dokumenata**; poruka greške ga ne smije citirati, kao ni kredencijal. Zapisuju se veličine i identifikatori, ne tekst |
| `NFR-ORG-07` | `ALLOWED` je deklarirana ulazna površina; podjela `ACCESS` / `CALL` u `DriverClaude` (`CHG-PRC-2026-08-20-01`) je prvi korak provedbe |
| OSCAL | `@oscal_driver` s istim skupom kao pripadna konekcija; deklaracija ne smije tvrditi kontrolu koju driver ne provodi |

## 11. Otvoreno

- **`write` nad modelom** — je li to pohrana rezultata inferencije (kroz repozitorij), predaja
  sadržaja modelu, ili fino podešavanje? Dok se ne odluči, semantika je pozivateljeva, a to je
  premalo za ugovor.
- **`update`** — osvježenje lokalne kopije (nova revizija) i osvježenje učitanog stanja nisu
  isto; možda su dvije operacije.
- Nasljeđuje se otvoreno iz [`HLRQ-13` §7](../01-HLRQ/HLRQ-13-llm-models.md#7-otvoreno): kanal
  pipeline→driver, imenovanje, OSCAL, provenijencija, referenca za zablude.
