# HLRQ-13 — Sposobnost prijevoda kroz transformaciju (jezični modeli)

> **Oznake su provizorne.** Razred `HLRQ` i kategorije `CON` / `DRV` nisu u vokabularu FR
> registra (postoji `ORG`, uz provizorni `AUD`); uvođenje traži DR (D-12).

| | |
|---|---|
| **Status** | Djelomično provedeno (2026-08-20) — `FR-CON-13.1` i `FR-DRV-13` u kodu (`CHG-PRC-2026-08-20-02`) |
| **Odluka** | [`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md) — konekcija po pod-sustavu; dohvat je okidač knjižnice |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `wattleflow-processors` — model runtime je third-party ovisnost (CLAUDE.md §7.4) |
| **Djeca** | [`FR-CON-13.1`](../02-FRQ/FRQ-CON-13.1-huggingface-connection.md) · [`FR-CON-13.2`](../02-FRQ/FRQ-CON-13.2-remote-model-connection.md) · [`FR-DRV-13`](../02-FRQ/FRQ-DRV-13-llm-model.md) · `FR-PIP-13` (kandidat, §4) |
| **Podloga** | [parametri po dobavljaču](../06-ANALYSIS/2026-08-20-model-vendor-parameters.md) · [granica konekcija/driver](../06-ANALYSIS/2026-08-20-connection-driver-boundary.md) · [proxy vs HuggingFace](../06-ANALYSIS/2026-08-20-proxy-vs-huggingface-connection.md) (2026-08-20) |
| **Sljedivost** | `NFR-ORG-04` (ontologija: `Connection`, `Driver`, `Pipeline`) · `NFR-SEC-03` (lokalnost distribucije, supply-chain) · `CLAUDE.md` §6.1, §7.4 · DR serije `DR-PRC` i `DR-WFL` — **nijedan zapis još nije otvoren** |

## 1. Narativ

Za korištenje lokalnih ili udaljenih jezičnih modela paket `processors` nema primitiv koji bi
osigurao **perzistenciju i pristup** modelu. Kad bi postojali driver i konekcija koji pružaju
sučelja za rad s modelima, pipeline bi se oslanjao na taj perzistencijski mehanizam i
transformaciju obavljao **čitanjem i pisanjem sadržaja kroz driver**. U toj konstelaciji uloga
konekcije je omogućiti pristup lokalnom ili udaljenom modelu, driver ga koristi za čitanje i
pisanje, a pipeline za transformaciju.

**Zašto.** Gotovi (off-the-shelf) alati za rad s modelima dolaze kao monolit: fiksni runtime,
fiksni pružatelj, sve-ili-ništa. Ta ograničenja uklanjamo razdvajanjem protoka podataka od
algoritma, kako bi rješenje ostalo **modularno, konfigurabilno i prilagodljivo** različitim
tehnološkim zahtjevima — zamjena modela ili pružatelja je izmjena konfiguracije, ne koda.

**Zatečeno.** Ništa od ovoga nije zatečeno. Postoji `DriverClaude` (udaljeni model:
`load`/`close`/`read`/`write`/`complete`) kao presedan; lokalnog parnjaka nema. Model se
učitava **unutar pipelinea** na 8 mjesta (`pipelines/nlp/`: `translate_hr`, `entities_en` ×3,
`entities_hr`, `entities_spacy` ×2), svaka instanca u vlastitu memoriju, bez oslobađanja i bez
provjere dostupnosti prije obrade.

## 2. Mjesto u dekompoziciji

Prijevod je **funkcija sustava kao cjeline**, uz bok sažimanju, ekstrakciji entiteta i
enkripciji. Kao funkcija, dijeli se na pod-podsustav od dva sloja:

| sloj | briga | primitivi | kategorija FR-a |
|---|---|---|---|
| **Perzistencija** | protok podataka: pristup, dohvat, pohrana | `Connection`, `Driver` | `CON`, `DRV` |
| **Transformacija** | sam prijevod: algoritam nad sadržajem | `Pipeline`, `Strategy` | `PIP` |

Budući da se u frameworku **sve transformacije izvode kroz pipeline**, ta granica daje
proširivost i ponovnu upotrebu: isti perzistencijski par poslužuje više pipelinea i više
transformacija uz minimalno ulaganje.

## 3. Ontologija funkcionalnosti (sjeme)

Popis postoji da imenovanje ne bi bilo stvar ukusa; ovo je **prijedlog osi**, ne usvojen
registar (proširenje vokabulara traži DR, D-12).

| funkcija sustava | perzistencija | transformacija |
|---|---|---|
| prijevod | `ConnectionHuggingFace` + `DriverLanguageModel` | `PipelineTranslate*` |
| sažimanje | isti par | `PipelineSummarise*` |
| ekstrakcija entiteta | isti par (ili spaCy konekcija) | `PipelineEntities*` |
| enkripcija | bez modela — ključ, ne model | strategija / `PipelineEncrypt*` |

Otvoreno: gdje registar živi (`dictionary.yaml` kao pojmovi diskursa, `tools/dictionary.json`
kao vokabular koda, ili zaseban registar funkcija) i tko ga održava.

## 4. Opseg

| oznaka | predmet | dokument |
|---|---|---|
| `FR-CON-13.1` | Konekcija prema **HuggingFace** pod-sustavu (cache + Hub); `offline` je način rada | [FR-CON-13.1](../02-FRQ/FRQ-CON-13.1-huggingface-connection.md) ✅ |
| `FR-CON-13.2` | Ostali dobavljači (Anthropic, Bedrock, Vertex, Foundry) — **pisan po staroj osi**, čeka prepis po pod-sustavima | [FR-CON-13.2](../02-FRQ/FRQ-CON-13.2-remote-model-connection.md) |
| `FR-DRV-13` | Driver nad konekcijom: `read` / `write` / `update` / `download` | [FR-DRV-13](../02-FRQ/FRQ-DRV-13-llm-model.md) |
| `FR-PIP-13` | Pipeline kao konzument usluge — kandidat, piše se kad se odluči kanal pipeline→driver (§7 t.2) | — |

**Os podjele: pod-sustav, ne izloženost** ([`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md)).
Konekcija u ovom frameworku omata klijentski objekt **jednog pod-sustava**; `huggingface_hub`
opslužuje i cache i Hub, pa bi ga rez na „lokalno/mrežno" razdvojio na dvije klase koje dijele
token i spremište. `offline` je zato način rada, a ne druga klasa. Različiti dobavljači ostaju
različite konekcije jer su različiti pod-sustavi.

### Zajednički ugovor konekcije

Obje konekcije zadovoljavaju **isti ugovor**, jer driver ne smije znati koju je dobio:

1. Sučelje je `connect` / `disconnect` + stanje; nijedna operacija nad sadržajem modela nije
   konekcijina (to je `FR-DRV-13`).
2. `connect` i `disconnect` su **idempotentni**.
3. Zamjena lokalne konekcije mrežnom (i obratno) je izmjena **konfiguracije**, bez izmjene koda
   pozivatelja (`BR-02`).
4. Kvar nosi **razred** — dosežnost · autentikacija · prava · kvota · nesukladnost — ne samo
   poruku; neriješen kvar zaustavlja workflow (`BR-06`).
5. Pristupni podaci ne stoje u YAML-u, nego kao reference razrješive kroz konfiguracijski lanac
   (`${dotenv:…}` / `${env:…}`).

## 5. Poslovna pravila

Vrijede za cijelu sposobnost; oba FR-a ih nasljeđuju i pozivaju se na oznaku.

| oznaka | pravilo |
|---|---|
| **BR-01** | Driver i konekcija moraju biti **registrirani u workflowu** i s valjanom konfiguracijom. |
| **BR-02** | Ovisno o konfiguriranoj konekciji, registrirani driver osigurava **pristup (read i write)** lokalnom ili mrežnom sustavu. |
| **BR-03** | Ako je konekcija mrežna, driver se — ovisno o konekciji — **spaja na model ili ga preuzima** (npr. HuggingFace model u datotekama). |
| **BR-04** | **Workflow ili procesor** traži pristup modelu od connection/driver managera, kako to ne bi radio svaki pipeline. |
| **BR-05** | **Pipeline je konzument** usluge drivera i konekcije. |
| **BR-06** | Ako problem s konekcijom ili pristupom konfiguriranom modelu **nije rješiv**, workflow ne započinje izvršavanje: prijavljuje grešku i prekida proces. |
| **BR-07** | Dostupnost modela provjerava se **pri pokretanju workflowa**, prije obrade prvog dokumenta; workflow ili procesor traži provjeru preko driver managera (`BR-04`). |
| **BR-08** | Model kojeg nema lokalno dohvaća se **samo kad konfiguracija to izrijekom dopušta**; bez dopuštenja nedostupan model je kvar (`BR-06`). |
| **BR-09** | Operacija koja traje — dohvat modela — **izvještava o napretku** u zapis u konfiguriranom intervalu (preneseno/ukupno, postotak, proteklo vrijeme), da se čekanje razlikuje od zastoja. |

> `BR-07…BR-09` su **prijedlog** (2026-08-20), izveden iz opisa procesa: provjera na početku,
> dohvat kao dopuštena radnja, vidljiv napredak. Potvrda je na autoru.

## 6. Nefunkcionalni zahtjevi i OSCAL

FR kaže što sustav radi; NFR ograničava **kako smije**. Ovi zahtjevi nisu preporuka — oni su ulaz
u kriterije prihvaćanja djece ovog HLRQ-a.

| NFR | što nalaže | posljedica za ovu sposobnost |
|---|---|---|
| `NFR-SEC-01` blast radius | ograniči dosežljivost iz kompromitirane komponente; least privilege | jedna dijeljena konekcija je **hub** — sve što driver dosegne ulazi u njezin `Blast`. Cijena dijeljenja modela je koncentracija: zato konekcija **po dobavljaču**, ne jedna za sve |
| `NFR-SEC-02` napadna površina | javno sučelje minimalno, `__all__` eksplicitan | ugovor konekcije je namjerno `connect`/`disconnect` + stanje; svaka dodatna metoda je trošak koji se brani, ne dodaje |
| `NFR-SEC-03` supply-chain i lokalnost | closure ⊆ tier distribucije; hash-pinned lock + SBOM; integritet vlastitih modula preko wheel `RECORD` | model runtime je third-party → cijela sposobnost pripada `wattleflow-processors`, lazy uvoz (§7.4). **Preuzeti model je third-party artefakt koji SBOM ne pokriva** — vidi §7 t.6 |
| `NFR-SEC-06` povjerljivost audit zapisa | bez `**kwargs` splata u zapis; redakcija ovisi o odredištu | pristupni podatak ne smije se pojaviti ni u zapisu ni u poruci greške; sadržaj dokumenta koji ide modelu ne citira se u greškama |
| `NFR-ORG-02` nomenklatura | ime imenuje ulogu; gole generičke imenice zabranjene | `ConnectionHuggingFace`, `DriverLanguageModel` — nikad `ModelManager` ili `Helper` |
| `NFR-ORG-04` sposobnost vs primitiv | cross-cutting sposobnost je helper, ne novi domenski primitiv | koriste se zatečeni primitivi (`Connection`, `Driver`, `Pipeline`); ontologija se ne proširuje |
| `NFR-ORG-08` deduplikacija | pravilo živi na jednom mjestu, ali **sličan oblik nije duplikat**: tri drivera s tri dijalekta su tri ugovora, ne jedan | ne spajati ih zbog sličnosti (`NFR.md`, prijedlog 2026-08-20) |
| `NFR-ORG-07` ulazna površina (`ALLOWED`) | konfiguracijski ključevi su deklarirani | podjela `ACCESS` / `CALL` u `DriverClaude` (`CHG-PRC-2026-08-20-01`) je provedba ovoga |

> **`NFR-ORG-07` nema vlastiti odjeljak u `NFR.md`.** Na njega se poziva `NFR-SEC-06` k.1, a
> `wem_lint` ga mjeri (`CLAUDE.md` §9) — registar ga ipak ne definira. Deklarirana rupa (D-11), ne
> pretpostavka.

### OSCAL — obveza, ne konvencija

`CLAUDE.md` §6.1: specijalizacije nose `@oscal_connection` / `@oscal_driver`; dekorator čita
`OSCAL_CONTROLS` ClassVar i izvršava `OSCALPolicy.verify()` sa semantikom `declared ⊆ baseline`,
uz crosswalk translaciju taksonomije. Svaka zatečena konekcija to već ispunjava:

| zatečeno | vrijednost |
|---|---|
| deklaracija | `OSCAL_CONTROLS: ClassVar[Tuple[str, ...]] = ("ac-3", "ia-5", "sc-8", "sc-13")` |
| nositelji | `postgres`, `kafka` (2×), `solr`, `elasticsearch`, `opensearch`, `spark`, `sftp_paramiko`, `proxy`, `aisstream`, `gfw` + driveri |
| dekorator | `@oscal_connection()` iznad klase |

**Posljedice za djecu ovog zahtjeva:**

1. `ConnectionHuggingFace` deklarira **puni skup** — nasljeđuje ga od `ProxyConnection` i sve
   četiri kontrole stvarno prolaze kroz njegovu sesiju: pristup (`ac-3`), račun (`ia-5`),
   prijenos (`sc-8`), kriptografija (`sc-13`). Isto vrijedi za `DriverLanguageModel`.
2. Deklaracija **ne smije tvrditi kontrolu koju klasa ne provodi**. Konekcija koja radi bez
   mreže i bez računa deklarirala bi podskup — takva u ovoj sposobnosti više ne postoji
   (`DR-PRC-001` t.1), pa pitanje podskupa otpada dok se ne pojavi pod-sustav bez prijenosa.
3. **Poznato ograničenje:** crosswalk mapiranja su `PROPOSED` i čekaju sign-off, a `sc-8→ism-0469`
   i `sc-13→ism-1080` su **izvan opsega Essential Eight** — protiv E8 baselinea ispravno padaju.
   Mrežna konekcija koja te kontrole treba pada gate dok se baseline ne proširi; to je zatečeno
   stanje, ne regresija koju uvodimo.
5. **Ispravljeno 2026-08-22:** OSCAL sloj je vendiran u `wattleflow-processors`, pa prepreke s
   ovisnošću o zasebnoj distribuciji nema — dekorirani se moduli uvoze u svakom okruženju u kojem
   je instalirana ova distribucija. Dekorateri su u uporabi na 15 komponenti, među njima
   `connections/postgres.py`, `drivers/postgres.py`, `processors/postgres.py` i
   `drivers/language_model.py`. Obveza je time izvediva; **provjera se i dalje ne izvodi** jer je
   `strict=False`, a nijedno mjesto ne predaje `oscal_policy=`
   ([`HLRQ-14`](HLRQ-14-oscal.md) §7 t.2).

## 7. Otvoreno


Odluke koje ovaj dokument ne donosi; svaka traži DR (D-03).

1. ~~Os podjele konekcija.~~ **Riješeno** [`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md)
   (2026-08-20): po pod-sustavu, `offline` je način rada. Preostaje posljedica — `FR-CON-13.2` je
   pisan po staroj osi i treba prepis po pod-sustavima (Anthropic, Bedrock, Vertex, Foundry).
2. **Numeracija i imenovanje datoteka.** Oblik: `HLRQ-<NN>-<tema>` (bez kategorije, jer HLRQ
   natkriljuje `CON` i `DRV`) i `FR-<KAT>-<NN>-<tema>`; broj je zajednički za djecu jedne
   sposobnosti. **Odluka (2026-08-20): `13`, po primjeru `examples/workflows/13_translate_hr.py`
   iz kojeg je zahtjev izveden — privremeno.** Cijena veze: broj primjera nije os registra
   zahtjeva; ako se primjer preimenuje, ukloni ili sposobnost preraste taj primjer, oznaka
   zavarava. Trajna shema (redni broj registra, neovisan o primjerima) ostaje otvorena.
3. **Kanal pipeline → driver.** Danas ne postoji: write-strategije driver dobivaju preko
   `kwargs.get("driver")`, procesoru ga injektira `WorkflowFactory._build_processors`, a
   pipeline dobiva samo svoj `configuration` blok. `BR-05` bez tog kanala nije izvediv; izvedba
   dira `concrete/` (CLAUDE.md §2.5) → zaseban DR, serija `DR-WFL`. Isto pitanje već stoji
   otvoreno za create-strategije (Tika stavka).
4. **Opseg pojma „model".** HF (seq2seq, token-classification), spaCy, GLiNER,
   sentence-transformers — da; Tesseract je binarni alat, ne isti razred.
5. **Imenovanje klasa — izvedeno:** `ConnectionHuggingFace`, `DriverLanguageModel`. Facet
   `model` i dalje ne postoji ni u `dictionary.json` ni u `dictionary-processors.yaml` →
   proširenje kroz DR (D-12).
6. **OSCAL obveza.** Izvedene klase nose `OSCAL_CONTROLS`, ali **dekorator nije primijenjen**.
   Razlog zapisan u kodu (paket nije deployan) **više ne stoji** — vidi §6 t.5; stvarni je razlog
   nedeklarirana ovisnost distribucije. Deklarirani dug, ne prešućen; isto vrijedi za
   `connections/proxy.py` i još devet modula ([`HLRQ-14`](HLRQ-14-oscal.md) §7 t.2).
7. **Provenijencija modela** (`NFR-SEC-03`): pinana revizija/sha, `cache_dir`, offline režim,
   `safetensors` umjesto `pickle` formata — kandidati za mjerljive kriterije prihvaćanja.
8. **Referenca za zablude distribuiranih sustava** (Deutsch 1994, +8. Gosling) nije u
   `LITERATURE.md`; dodavanje ključa u append-only registar traži DR (D-12).
