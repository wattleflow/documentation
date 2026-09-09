# FRQ-MAIL-02 — Prepoznavanje prepiske u tekstu oštećenom OCR-om

> **Oznaka je provizorna**, kao i kod [`FRQ-MAIL-01`](FRQ-MAIL-01-message-parser.md): `MAIL` je
> sposobnost, ne ontološki primitiv, i nema nadređeni HLRQ. Broj ne referira roditelja.

| | |
|---|---|
| **Status** | **Prijedlog. Nije provedeno.** Mjereno je samo na **sintetičkom** korpusu (§9) — dakle nalaz o izvedivosti, ne o učinku |
| **Norme** | [`NFRQ-ORG-10`](../03-NFRQ/NFRQ-ORG-10-learned-artefact-EN.md) · [`NFRQ-SEC-04`](../03-NFRQ/NFRQ-SEC-04-detection-operating-point-EN.md) · [`NFRQ-DEF-02`](../03-NFRQ/NFRQ-DEF-02-measurement-charter-EN.md) · [`NFRQ-ORG-01`](../03-NFRQ/NFRQ-ORG-01-helper-locality-EN.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) |
| **Sestrinski** | [`FRQ-MAIL-01`](FRQ-MAIL-01-message-parser.md) — čitanje spremnika poruke; ovaj zahtjev **ne čita spremnik nego tekst** |
| **Predmet** | Naučeni klasifikator (TF-IDF znakovni n-grami + linearni model) koji odlučuje **samo ondje gdje determinističko prepoznavanje otkaže** |
| **Jezik predmeta** | **UK English isključivo.** Hrvatske oznake ispisa nisu u opsegu (odluka 2026-09-05) |

## 1. Opseg

Tekst izvučen iz PDF-a nosi ili **prepisku** (ispisan ili izvezen e-mail) ili **dokument**.
Razlika je nosiva jer određuje što se dalje s njim radi: prepiska ima strane, vrijeme i predmet
koje `FRQ-MAIL-01` zna pročitati iz bloka glavica; dokument ih nema.

Na **čitljivom** tekstu razliku nosi determinističko prepoznavanje: suvisao blok oznaka
(`From`, `Sent`, `To`, `Cc`, `Subject`) koji **počinje** na vrhu i sadrži barem jedan `addr-spec`.
Taj put je predmet zasebnog zapisa i **ovdje se ne opisuje** (D-13).

Ovaj zahtjev pokriva **jedini slučaj u kojem to otkaže**: skenirani ispis, gdje OCR lomi upravo
oznake na kojima pravilo stoji — `From:` → `Fr0m:`, `Subject:` → `Subjcct:`. Tada se oznaka ne
podudara, blok se ne nađe, i prepiska se tiho vodi kao dokument.

**Izvan opsega:** čitljiv tekst (ondje odlučuje pravilo, `NFRQ-ORG-10` k.1), izvlačenje samog
sadržaja (`DriverPdf`), i rasklapanje pronađenog bloka (`MailMessage.from_header_block`).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | determinističko prepoznavanje bloka glavica | tekst → nalaz + **ocjena oštećenja** |
| **A2** | `GenericPipeline` — vlasnik transformacije; poziva klasifikaciju i žigoše nalaz | facade |
| **A3** | naučeni klasifikator — predmet ovog zahtjeva | glava teksta → razred + pouzdanost |
| **A4** | artefakt modela + njegov manifest | putanja iz konfiguracije |
| **A5** | postupak učenja — **izvan izvođenja**, nikad u cjevovodu | označeni korpus |
| **A6** | `MailMessage` — rasklapa blok kad je prepiska potvrđena | tekst bloka |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | pipeline dobije dokument s izvučenim tekstom |
| **EV02** | A1 ne nađe blok, **a** tekst nosi znakove OCR oštećenja |
| **EV03** | model je učitan i njegov digest se slaže → klasifikacija |
| **EV04** | model nedostaje, nečitljiv je ili mu digest ne odgovara → `WARNING`, ostaje nalaz od A1 |
| **EV05** | razred je „prepiska" uz pouzdanost iznad radne točke → A6 rasklapa blok |
| **EV06** | pouzdanost ispod radne točke → nalaz je „neodlučeno", ne „dokument" |

## 4. Preduvjeti

1. Tekst je već izvučen; ovaj zahtjev ne čita datoteku (`FRQ-PRC-15.22`).
2. A1 je pokrenut **prvi** i nije odlučio — `NFRQ-ORG-10` k.1 zabranjuje model na čistom ulazu.
3. Artefakt modela i manifest postoje na konfiguriranoj putanji; inače vrijedi EV04.
4. `scikit-learn` je dostupan. Ovisnost pripada `wattleflow-processors`, koji je **deklarirana
   iznimka** od zero-trust opsega (`CLAUDE.md` §7.4), pa nije prekršaj `NFRQ-SEC-03`.

## 5. Normalan tok

1. Pipeline uzme tekst iz dokumenta i pozove A1.
2. A1 vrati nalaz i **ocjenu oštećenja** — udio redaka koji *nalikuju* oznaci a ne podudaraju se
   točno. Ocjena je ono što razdvaja „nema bloka" od „blok je nečitljiv".
3. Ako je nalaz odlučen, tu se staje. **Model se ne poziva.**
4. Inače se uzme **glava** teksta — prvih ~15 nepraznih redaka. Mjereno je da signal živi ondje;
   cijeli dokument unosi šum iz tijela.
5. Klasifikator vrati razred i pouzdanost. Vektorizacija je `char_wb` 3–5-grami: znakovni n-gram
   preživljava zamjenu jednog znaka, riječni ne — `Fr0m` i `From` dijele `_Fr`, `rom`.
6. Iznad radne točke i za razred „prepiska", blok se preda `MailMessage.from_header_block`.
7. Dokument se žigoše: razred, pouzdanost, **identifikator modela** i dokazi kojima je A1
   raspolagao (`NFRQ-ORG-10` k.6).

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| A1 odlučio | model se ne poziva | jeftin i čitljiv put ostaje zadani |
| nema modela / digest ne odgovara | `WARNING`, ostaje nalaz od A1 | prolaz ne pada (`NFRQ-ORG-10` k.9) |
| pouzdanost ispod radne točke | nalaz **„neodlučeno"** | odsutnost dokaza nije dokaz odsutnosti |
| `scikit-learn` nije instaliran | `WARNING` pri konstrukciji, model se ne učitava | slaba instalacija radi bez njega |
| model kaže „prepiska", blok se ne da rasklopiti | žig ostaje, strane se ne upisuju | razred i sadržaj su dvije tvrdnje |

## 7. Rezultat

Skenirana prepiska koja bi inače prošla kao dokument dobiva razred i mjeru pouzdanosti, uz trag
koji dopušta da se nalaz preispita **bez modela**. Čitljiv tekst prolazi istim putem kao i prije.

## 8. Kriteriji prihvaćanja

1. Model se ne poziva kad je A1 odlučio. — *provjera pregledom pozivnog mjesta*
2. Značajke su **glava** teksta, ne cijeli dokument. — *pregled*
3. Vektorizacija je znakovna (`char_wb` 3–5), ne riječna. — *pregled*
4. **`[M]`** Na **stvarnom** izdvojenom skupu model nadmašuje determinističku osnovicu za deklariranu
   razliku; mjeri se **po razredu** (preciznost, odziv), nikad jednim brojem. ❌ **nema stvarnog korpusa**
5. Artefakt nosi manifest po `NFRQ-ORG-10` k.4. ❌ nema artefakta
6. Nedostupan model ne ruši prolaz. — *provjerivo kad postoji nositelj*
7. Radna točka je odabrana iz omjera cijene FP/FN i prevalencije (`NFRQ-SEC-04` k.1), ne „strože". ❌ otvoreno
8. Nalaz nosi identifikator modela i dokaze od A1. — *pregled zapisa*
9. Nijedna clean core distribucija ne ovisi o `scikit-learn`. — `wem_lint` `clean_core_imports`

## 9. Verifikacija — izmjereno, i što ta mjera vrijedi

Sintetički korpus, 520 dokumenata (Outlook / Gmail / Apple Mail ispisi kao pozitivi; ugovor,
izvještaj, **izvještaj s citiranim e-mailom u prilogu**, **interni dopis** i pismo kao negativi),
30% za test:

| pristup | čist tekst | µs/dok | OCR 4% | OCR 8% | OCR 15% |
|---|---|---|---|---|---|
| pravilo, blok bez uvjeta o adresi | 0.897 | 12.5 | 0.865 | — | — |
| **pravilo, uz `addr-spec` u bloku** | **1.000** | **8.2** | 0.968 | 0.949 | 0.904 |
| TF-IDF riječi, cijeli tekst | 1.000 | 30.6 | 1.000 | — | — |
| **TF-IDF `char_wb` 3–5, samo glava** | 1.000 | 178.9 | 1.000 | — | — |

Dva nalaza koja određuju opseg ovog zahtjeva:

- **Na čistom tekstu model nema prednost.** Razmak prve i druge retke nije bio granica pristupa
  nego **jedan nedostajući uvjet**: pravilo je padalo na internom dopisu (60/60), koji ima blok
  glavica ali **nema adresa**. Zato `NFRQ-ORG-10` k.1 i kriterij 1 ovdje.
- **Vrijednost modela je isključivo u degradaciji**: pravilo pada s 1.000 na 0.904 pri 15% šuma,
  znakovni n-grami drže.

> **Ove brojke nisu svjedočanstvo o učinku (D-05).** Korpus je sintetički i ograničene
> raznolikosti, pa model dijelom uči generator napamet — 1.000 je **napuhana**. Vrijede kao dokaz
> **izvedivosti i smjera**, ne kao mjera. Kriterij 4 ostaje neispunjen dok ne postoji stvarni skup.

**Trojka reproducibilnosti (D-10):** alat — `scikit-learn` 1.9.0, skripta korpusa i evaluacije;
kriterij — §8 gore; platforma — CPython 3.11.15, Linux/WSL2, 2026-09-05. **Mjereno stablo:**
sintetički korpus, nijedna produkcijska datoteka.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-10` | determinističko prvo; manifest; ponovljivo učenje; deklarirana neprozirnost |
| `NFRQ-SEC-04` | prag se bira iz omjera cijene i prevalencije; poboljšanje je pomak ROC krivulje, ne praga |
| `NFRQ-DEF-02` | točnost je `[M]`, omjerna skala, po razredu, dijagnostička — vrata samo kroz DR |
| `NFRQ-ORG-01` | klasifikator je sposobnost dviju domena → dijeljeni helper, uz vokabular u `mail` |
| `NFRQ-SEC-03` | `scikit-learn` pripada `wattleflow-processors` (§7.4 iznimka), nikad clean coreu |
| `NFRQ-OBS-01/03` | pipeline prijavljuje stavku; klasifikator šuti osim `WARNING`-a iz EV04 |

## 11. Otvoreno

1. **Nema stvarnog korpusa** — i nije odlučeno smije li postojati. Učenje na pravoj prepisci
   povlači *Privacy Act 1988* (Cth), APP 3 i APP 11, koje projekt već uzima kao sidro
   (`DR-WFL-008`). Bez te odluke kriterij 4 je neispunjiv, a bez njega model nije prihvatljiv.
2. **Ocjena oštećenja nije definirana.** „Redak nalikuje oznaci a ne podudara se" traži mjeru
   udaljenosti (Levenshtein na prefiksu?) i prag. Dok je nema, EV02 se ne može strojno odlučiti.
3. **Radna točka** (k.7) — omjer cijene lažno pozitivnog i lažno negativnog nalaza nije poznat
   jer ovisi o tome što se s razredom dalje radi.
4. **Razred „neodlučeno" nije u vokabularu.** Trovrijedni nalaz traži ime i za treću vrijednost;
   proširenje vokabulara ide kroz DR (D-12).
5. **Gdje živi artefakt** — u paketu, uz konfiguraciju, ili se preuzima? Prva opcija narušava
   `MANIFEST.in` kao jedinu kontrolnu točku pakiranja (`DR-WFL-006`); treća uvodi mrežu u put
   učitavanja.
