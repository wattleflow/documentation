# FR-CON-13.1 — Konekcija prema HuggingFace pod-sustavu

> **Oznaka je provizorna.** Kategorija `CON` nije u vokabularu FR registra; uvođenje traži DR
> (D-12).

| | |
|---|---|
| **Status** | Provedeno (2026-08-20) — `CHG-PRC-2026-08-20-02` |
| **Odluka** | [`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md) — konekcija po pod-sustavu, nasljeđivanje od `ProxyConnection` |
| **Nadređeni zahtjev** | [`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) — narativ, `BR-01…BR-09`, zajednički ugovor konekcije (§4) |
| **Predmet** | `ConnectionHuggingFace(ProxyConnection)` — pristup cacheu na disku i Hubu iza njega; `connect` / `disconnect` |
| **Sestrinski** | [`FR-DRV-13`](FRQ-DRV-13-llm-model.md) (operacije nad modelom) · [`FR-CON-13.2`](FRQ-CON-13.2-remote-model-connection.md) (ostali dobavljači — po staroj osi, `DR-PRC-001` §Cijena) |
| **Izvedba** | `connections/huggingface.py` |

## 1. Predmet

HuggingFace je **jedan pod-sustav**: cache na disku i Hub iza njega, s jednom knjižnicom koja ih
obje opslužuje. Konekcija zato pokriva oboje, a `offline` je njezin **način rada**, ne druga klasa
([`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md) t.1).

| dolazi iz `ProxyConnection` | dodaje ova konekcija |
|---|---|
| proxy (`proxy_url`, auth basic/NTLM/Kerberos, `no_proxy`) | `cache_dir` — gdje je lokalno spremište |
| TLS (`verify_ssl`, `ca_bundle`, `client_cert`/`client_key`) | `offline` — smije li se izaći na mrežu |
| `token` → `Authorization: Bearer` = **HF račun** | `endpoint` — koji Hub odgovara |
| `timeout`, `headers`, redakcija tajni u zapisu | |

`connect()` i dalje yielda `requests.Session` — ugovor roditelja se ne mijenja. Sesija je ono što
driver preda knjižnici kao HTTP pozadinu (`FR-DRV-13` §5 t.2).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Konfiguracija (YAML) — registrira konekciju imenom i tipom | `type`, `name`, `configuration`; tajne kao reference |
| **A2** | `Workflow` / `Processor` — pokreće zahtjev za pristupom (`BR-04`) | ime registrirane konekcije |
| **A3** | `ConnectionManager` — vlasnik registriranih konekcija | registar imenovanih konekcija |
| **A4** | `ConnectionHuggingFace` — predmet ovog zahtjeva | sesija i stanje pristupa |

Cache i Hub **nisu akteri** — ne pokreću ništa; sudionici su toka i izvori kvara.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | Faktory gradi workflow → registrira konekciju iz konfiguracije (`BR-01`) |
| **EV02** | A2 traži pristup od `ConnectionManager`-a (`BR-04`) |
| **EV03** | Driver traži sesiju i postavke prije operacije nad modelom |
| **EV04** | Workflow završava ili se ruši → oslobađanje pristupa |

## 4. Preduvjeti

1. Konekcija je registrirana s valjanom konfiguracijom (`BR-01`).
2. `cache_dir` je zadan ili se izvodi iz okoline (`HF_HUB_CACHE` → `TRANSFORMERS_CACHE` →
   `$HF_HOME/hub`), preko `helpers.localmodels.DownloadedModels` — jedno mjesto za to pravilo
   (`NFR-ORG-08`).
3. `token` dolazi kao referenca razrješiva kroz konfiguracijski lanac (`${dotenv:…}` / `${env:…}`),
   nikad kao literal u YAML-u.
4. `offline` je deklariran kad se od okoline očekuje rad bez mreže.

## 5. Normalan tok

1. EV01 — faktorija instancira konekciju i registrira je pod imenom.
2. EV02 — A2 traži pristup od A3 po imenu.
3. A4 gradi sesiju naslijeđenim koracima: proxy → TLS → autentikacija (`token`) → zaglavlja.
4. A4 u `_configure_service` potvrđuje da je **spremište upotrebljivo** (postoji ili se može
   stvoriti, i direktorij je). Postoji li *imenovani model*, pitanje je za driver
   (`FR-DRV-13` §5 t.3) — konekcija daje pristup, ne odgovara na pitanja o sadržaju.
5. Proba se **ne izvodi** (`PROBE_ENABLED_DEFAULT = False`): generički endpoint ne govori ništa o
   Hubu, a stanje se objavljuje zapisom (cache, endpoint, `offline`, je li autentificirano).
6. A4 prelazi u stanje *povezan*; A3 vraća konekciju, driver je smije koristiti (EV03).
7. EV04 — `disconnect` oslobađa sesiju.

**Normalan tok je postignut kad je sesija izgrađena i spremište potvrđeno.** Mreža se u ovom toku
**ne dodiruje** — prvi mrežni poziv radi driver.

## 6. Alternativni tokovi

Zablude distribuiranih sustava ovdje ne nastupaju jer `connect()` ne izlazi na mrežu; mrežni
kvarovi pripadaju driveru (`FR-DRV-13` §6). Kvarovi ove konekcije su lokalni i konfiguracijski:

| uvjet | ponašanje |
|---|---|
| konekcija nije registrirana ili joj je konfiguracija nevaljana | workflow **ne započinje** (`BR-01`, `BR-06`) |
| `cache_dir` se ne može stvoriti ili nije direktorij | `HuggingFaceConnectionError` s putanjom; razred *dosežnost* |
| nema prava pisanja u spremište | isti razred kvara s razlogom iz sustava datoteka; razlikuje se od nepostojanja |
| `token` se ne razrješava | kvar konfiguracije; vrijednost se **ne** ispisuje u zapis (`NFR-SEC-06`) |
| `offline: true`, a model nedostaje | nije kvar konekcije — driver odbija dohvat (`BR-08`, `FR-DRV-13` §6) |
| proxy je nedostupan | vidi se tek na prvom pozivu drivera; konekcija ne provjerava (`t.5` gore) |
| ponovni `connect` / `disconnect` nad istim stanjem | idempotentno (`HLRQ-13` §4 t.2) |

## 7. Rezultat

Sesija je izgrađena, spremište potvrđeno, pristup imenovan i dijeljen: svaki driver u workflowu
koristi **istu** konekciju, s istim proxyjem, TLS materijalom i računom. Neupotrebljivo spremište
ili nerazriješena tajna zaustavljaju workflow prije obrade (`BR-06`).

## 8. Kriteriji prihvaćanja

1. Zadovoljen zajednički ugovor konekcije (`HLRQ-13` §4, t.1–5); `connect()` yielda
   `requests.Session`. ✅
2. `ALLOWED` je roditeljski popis **plus** `cache_dir`, `offline`, `endpoint` — ništa skriveno. ✅
3. HF račun je naslijeđeni `token`; druge lokacije za istu tajnu nema. ✅
4. `connect` ne dodiruje mrežu, ni radi provjere sadržaja spremišta. ✅
5. Neupotrebljivo spremište pada prije uporabe, s putanjom u poruci. ✅
6. Konekcija ne odgovara na pitanje je li model prisutan. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-20) |
|---|---|---|
| 1, 3 | instanciranje + pregled sesije | `Session` s postavljenim `Authorization` |
| 2 | usporedba popisa | razlika točno `["cache_dir", "offline", "endpoint"]` |
| 4 | pregled putanje `create_connection` | proba isključena, bez mrežnog poziva |
| 5 | negativan slučaj (`/proc/1/nope`) | `HuggingFaceConnectionError` s putanjom |
| 6 | pregled sučelja | metode statusa modela nema |
| uvoz | test maskiranja (`huggingface_hub`, `transformers`, `tqdm`) | modul se uvozi (`NFR-SEC-03`) |

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-13` §6](../01-HLRQ/HLRQ-13-llm-models.md#6-nefunkcionalni-zahtjevi-i-oscal).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFR-SEC-01` | jedna konekcija po pod-sustavu; kompromitacija HF pristupa ne doseže druge dobavljače |
| `NFR-SEC-02` | nasljeđivanje donosi i roditeljske ključeve — konfiguracijska površina raste; prihvaćena cijena (`DR-PRC-001` §Cijena) |
| `NFR-SEC-03` | `huggingface_hub` je lazy uvoz; modul se uvozi i bez njega |
| `NFR-SEC-06` | `token` se maskira u zapisu (naslijeđeni `_safe_log_kwargs`); zapisuje se samo *je li* autentificirano |
| `NFR-ORG-08` | razrješavanje cachea ne prepisuje se — koristi `DownloadedModels` |
| OSCAL | `OSCAL_CONTROLS` naslijeđen nepromijenjen (`ac-3`, `ia-5`, `sc-8`, `sc-13`) — pristup, kredencijal, prijenos i kriptografija stvarno prolaze kroz ovu sesiju. Dekorator nije primijenjen jer `wattleflow.oscal` nije deployan (`HLRQ-13` §6 t.4) |

## 11. Otvoreno

1. **`offline` ne postavlja `HF_HUB_OFFLINE`.** Zastavica danas zaustavlja *driverov* dohvat, ali
   knjižnicu pozvanu izravno iz pipelinea ne sprječava da izađe na mrežu. Kandidat: izvoz kroz
   `runtime:` blok ili `local_files_only` na svakom pozivu.
2. **Definicija „valjanog modela"** je heuristika u driveru (config + težine); mjesto joj je
   ovdje ili u `FR-DRV-13`, uz sign-off.
3. `FR-CON-13.2` je i dalje pisan po osi lokalno/mrežno — treba ga prepisati po pod-sustavima
   (`DR-PRC-001` §Cijena).
