# FRQ-CON-13.2 — Mrežna konekcija prema jezičnom modelu (više dobavljača)

> **Oznaka je provizorna.** Kategorija `CON` nije u vokabularu FR registra; uvođenje traži DR
> (D-12).

> **Pisan po staroj osi (lokalno/mrežno).** [`DR-PRC-001`](../04-DR/DR-PRC-001-model-access-boundary.md)
> je 2026-08-20 odlučio podjelu **po pod-sustavu**; ovaj dokument čeka prepis na konekcije po
> dobavljaču (Anthropic, Bedrock, Vertex, Foundry). Sadržaj §1–§9 vrijedi kao zahtjev, ime
> `ConnectionRemoteModel` ne.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-20) |
| **Nadređeni zahtjev** | [`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) — narativ, `BR-01…BR-06`, **zajednički ugovor konekcije** (§4) |
| **Predmet** | `ConnectionRemoteModel` — pristup modelu kod **pružatelja**; `connect` / `disconnect` uz razrješavanje dobavljača |
| **Sestrinski** | [`FRQ-CON-13.1`](FRQ-CON-13.1-huggingface.md) (HuggingFace) · [`FRQ-DRV-13`](FRQ-DRV-13-llm-model.md) (operacije nad modelom) |
| **Zatečeno** | Ne postoji kao konekcija. Postoji `DriverClaude` — **driver** koji danas sam nosi endpoint, autentikaciju i dijalekt jednog dobavljača (§11 t.1) |

## 1. Predmet

Pristup je **udaljena strana**: HuggingFace Hub, komercijalni pružatelj (Anthropic, OpenAI…) ili
poslužitelj modela (Ollama, vLLM, TGI) — i kad je taj poslužitelj na istom stroju, jer ima
endpoint i protokol.

Ono što ovu konekciju razlikuje od lokalne nije mreža nego **razrješavanje dobavljača**: svaki
nosi vlastiti endpoint, način autentikacije i dijalekt sučelja. Konfiguracija imenuje dobavljača,
konekcija razrješava njegov pristup, a driver i pipeline ostaju isti (`BR-02`).

| što se razrješava | primjer razlike |
|---|---|
| endpoint | Hub URL · URL pružatelja · `localhost` poslužitelj |
| autentikacija | token · API ključ · bez autentikacije |
| dijalekt | REST shema pružatelja · protokol poslužitelja |
| ograničenja | kvota, brzina, veličina zahtjeva |

**Razrješavanje je zatvoreno na registrirane dobavljače**: nepoznat dobavljač je kvar
konfiguracije, ne pokušaj „najboljeg pogotka".

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Konfiguracija (YAML) — registrira konekciju i imenuje dobavljača | `type`, `name`, `configuration` (dobavljač, endpoint, referenca na pristupni podatak, model) |
| **A2** | `Workflow` / `Processor` — pokreće zahtjev za pristupom (`BR-04`) | ime registrirane konekcije |
| **A3** | `ConnectionManager` — vlasnik registriranih konekcija | registar imenovanih konekcija |
| **A4** | `ConnectionRemoteModel` — predmet ovog zahtjeva | stanje pristupa, razriješeni dobavljač |
| **A5** | Registar dobavljača — preslikava ime dobavljača u endpoint, autentikaciju i dijalekt | podržani dobavljači |

Pružatelj (Hub, API, poslužitelj) **nije akter** — ne pokreće ništa; sudionik je toka i izvor
kvara.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | Faktorija gradi workflow → registrira konekciju iz konfiguracije (`BR-01`) |
| **EV02** | A2 traži pristup od `ConnectionManager`-a (`BR-04`) |
| **EV03** | Driver traži stanje konekcije prije operacije nad modelom |
| **EV04** | Pristup istekne ili se izgubi tijekom rada |
| **EV05** | Workflow završava ili se ruši → oslobađanje pristupa |

## 4. Preduvjeti

1. Konekcija je registrirana s valjanom konfiguracijom (`BR-01`).
2. Konfiguracija **imenuje dobavljača**; dobavljač je u registru A5.
3. Pristupni podatak dolazi kao referenca razrješiva kroz konfiguracijski lanac
   (`${dotenv:…}` / `${env:…}`), nikad kao literal u YAML-u.
4. Vremenska ograničenja (`connect` timeout, politika ponavljanja) su konfigurirana, ne
   podrazumijevana kao beskonačna.

## 5. Normalan tok

1. EV01 — faktorija instancira konekciju i registrira je pod imenom.
2. A2 traži pristup od A3 po imenu.
3. A4 razrješava **dobavljača** kroz A5: endpoint, način autentikacije, dijalekt, ograničenja.
4. A4 razrješava pristupni podatak kroz konfiguracijski lanac.
5. A4 izvodi `connect`: potvrđuje dosežnost endpointa i prolaz autentikacije.
6. A4 potvrđuje da je **imenovani model** dostupan kod tog dobavljača (i revizija, kad je
   zadana).
7. A4 prelazi u stanje *povezan* i objavljuje ga (`Operation.Connect`); razriješeni dobavljač je
   dio stanja koje driver čita.
8. EV05 — `disconnect` zatvara sjednicu i oslobađa što je konekcija držala.

**Normalan tok je postignut kad je dobavljač razriješen, endpoint dosežan i model kod njega
potvrđen** — dohvat i inferencija su `FRQ-DRV-13`.

## 6. Alternativni tokovi

Po zabludama distribuiranih sustava (Deutsch 1994; 8. Gosling) — ovdje sve vrijede, jer mreža
postoji.

| # | zabluda | slučaj | ponašanje |
|---|---|---|---|
| 1 | mreža je pouzdana | endpoint padne između `connect` i uporabe | gubitak stanja se objavljuje (EV04); ponavljanje po deklariranoj politici, pa kvar |
| 2 | latencija je nula | `connect` visi | timeout je konfiguriran; istek = kvar s razlogom, ne beskonačno čekanje |
| 3 | propusnost je beskonačna | — dohvat sadržaja nije posao konekcije | konekcija ne prenosi model (`FRQ-DRV-13`) |
| 4 | mreža je sigurna | istekao token, nevaljan ključ, TLS greška | razred *autentikacija* razlikuje se od *dosežnosti* |
| 5 | topologija je nepromjenjiva | pružatelj preselio endpoint ili ugasio verziju sučelja | kvar imenuje razriješeni endpoint i dobavljača |
| 6 | postoji jedan administrator | ključ opozvan izvana; model povučen kod pružatelja | kvar imenuje dobavljača; ne pokušava se drugim dobavljačem |
| 7 | trošak transporta je nula | kvota, naplata, ograničenje brzine | razred *kvota* — ne ponavlja se slijepo |
| 8 | mreža je homogena | dijalekt dobavljača ne odgovara očekivanom | razred *nesukladnost*; zaobilaženje nije dopušteno |

| ostali uvjet | ponašanje |
|---|---|
| dobavljač nije u registru A5 | kvar konfiguracije **prije** mrežnog poziva (`BR-01`, `BR-06`) |
| pristupni podatak se ne razrješava | kvar konfiguracije; vrijednost se ne ispisuje u zapis |
| imenovani model ne postoji kod dobavljača | kvar imenuje model i dobavljača; bez zamjene drugim modelom |
| ponovni `connect` / `disconnect` nad istim stanjem | idempotentno (`HLRQ-13` §4 t.2) |

Nijedan kvar ne prelazi tiho na drugog dobavljača ni na lokalnu konekciju: **zamjena je izmjena
konfiguracije, ne runtime odluka.**

## 7. Rezultat

Dobavljač je razriješen, pristup uspostavljen i imenovan, model kod dobavljača potvrđen; driver
ga koristi ne znajući čiji je. Kvarovi nose razred (dosežnost · autentikacija · prava · kvota ·
nesukladnost) i zaustavljaju workflow prije obrade (`BR-06`).

## 8. Kriteriji prihvaćanja

1. Zadovoljen zajednički ugovor konekcije (`HLRQ-13` §4, t.1–5).
2. Dodavanje novog dobavljača je **unos u registar A5** — bez izmjene drivera i pipelinea.
3. Nepoznat dobavljač i nerazriješen pristupni podatak padaju prije prvog mrežnog poziva.
4. Pristupni podatak se ne pojavljuje u audit zapisu ni u poruci greške.
5. Nema tihe zamjene: ni dobavljača, ni modela, ni revizije.
6. Svako čekanje ima gornju granicu; politika ponavljanja je deklarirana, ne implicitna.

## 9. Verifikacija

| kriterij | metoda |
|---|---|
| 1, 2 | pregled sučelja i registra dobavljača |
| 3–6 | test (po odabiru test-frameworka; do tada **pregled**) |
| 4 | analiza — skan zapisa i poruka na pristupne podatke |

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-13` §6](../01-HLRQ/HLRQ-13-llm-models.md#6-nefunkcionalni-zahtjevi-i-oscal).
Ovdje samo ono što je specifično za mrežni pristup.

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-SEC-01` | **konekcija po dobavljaču**, ne jedna za sve: kompromitacija jednog pristupa ne smije dosegnuti ostale |
| `NFRQ-SEC-02` | registar dobavljača (A5) je unutarnji; javno sučelje ostaje `connect` / `disconnect` + stanje |
| `NFRQ-SEC-03` | provenijencija je **nejednaka po dobavljaču**: pinanje revizije postoji kod HF-a i Vertexa, drugdje ne. Jamstvo se deklarira po dobavljaču, ne obećava jednako svima (§11 t.4) |
| `NFRQ-SEC-06` | pristupni podatak ne ulazi ni u zapis ni u poruku greške (§8 k.4); kvota i endpoint smiju |
| OSCAL | puni skup: `ac-3` (pristup), `ia-5` (kredencijal), `sc-8` (prijenos), `sc-13` (kriptografija). `sc-8`/`sc-13` su izvan E8 baselinea i protiv njega padaju — zatečeno ograničenje crosswalka (`HLRQ-13` §6, t.3) |

## 11. Otvoreno

> Podloga: [analiza parametara po dobavljaču](../06-ANALYSIS/2026-08-20-model-vendor-parameters.md) — minimalni i maksimalni skup parametara za Anthropic, HuggingFace, AWS
> Bedrock, Azure Foundry i GCP Vertex, presjek zajedničkog i prijedlog ugovora (§4 ondje).

1. **Granica dobavljača između konekcije i drivera.** `DriverClaude` danas nosi endpoint,
   autentikaciju i dijalekt jednog dobavljača. Ako dobavljača razrješava konekcija, što ostaje
   driveru — samo operacije, ili i dijalekt? Bez odgovora logika dobavljača završava na oba
   mjesta. **Ovo je glavno otvoreno pitanje ovog zahtjeva.**
2. **Oblik registra dobavljača (A5)** — mapa u kodu, konfiguracija, ili strategija po
   dobavljaču. Vezano uz t.1.
3. **Poslužitelj na istom stroju** (Ollama/vLLM/TGI) vođen je ovdje jer ima endpoint; potvrditi
   da je to željena granica prema [`FRQ-CON-13.1`](FRQ-CON-13.1-huggingface.md).
4. Nasljeđuje se otvoreno iz [`HLRQ-13` §7](../01-HLRQ/HLRQ-13-llm-models.md#7-otvoreno).
