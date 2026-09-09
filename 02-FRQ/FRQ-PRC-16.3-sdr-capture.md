# FRQ-PRC-16.3 — Procesor prihvata s SDR uređaja

> **Oznaka je provizorna** koliko i razred `HLRQ` iznad nje (`CLAUDE.md` §3.6). Kategorija `PRC`
> je u vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Prijedlog — nije provedeno** (provjereno 2026-09-09) |
| **Odluka** | Nijedna; vidi [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §7 |
| **Nadređeni zahtjev** | [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) — `BR-01…BR-10` |
| **Predmet** | `ProcessorSdrCapture(GenericProcessor)` — vodi prolaz prihvata: generator nad driverom, dokument po bloku, granica prihvata i flusha |
| **Sestrinski** | [`FRQ-CON-16.1`](FRQ-CON-16.1-sdr-device.md) · [`FRQ-DRV-16.2`](FRQ-DRV-16.2-iq-stream.md) |
| **Kaskada** | P-11 (granica oko promjenjive odluke) · P-14 (specifikacija znanja, ne dnevnik) · P-08 (proza nosi uloge, rječnik imena) |
| **Ugovor prolaza** | [`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md) — procesor vlasnik prolaza; [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) — platno između stopa |
| **Izvedba** | — (predložena putanja: `processors/sdr.py`) |

## 1. Predmet

Procesor je **vlasnik prolaza** ([`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md)): vodi generator,
provodi svaku stavku kroz svaki pipeline, odlučuje granicu flusha i jedini je primitiv koji se
nastavlja iz mementa. Ovaj procesor tu ulogu ispunjava nad izvorom koji se **ne da zaustaviti** —
i time otvara dvije stvari koje zatečeni ugovor ne pokriva: granicu prihvata (§5 t.6) i
nastavljivost (§11 t.1).

Radi se o **prihvatu**, ne obradi: procesor ne demodulira i ne dekodira. Transformacija je
pipeline (`FRQ-PIP-16.5`, kandidat).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `Workflow` — pokreće procesor i drži registre | konfiguracija prolaza |
| **A2** | Procesor prihvata — predmet ovog zahtjeva | granica prihvata, granica flusha |
| **A3** | Driver nad konekcijom — izvor blokova | blokovi s opisom, brojač gubitaka |
| **A4** | `Blackboard` — platno između stope uređaja i stope spremišta | dokumenti u nastajanju |
| **A5** | `Pipeline`(i) — konzumenti svake stavke | transformacije nad blokom |

Registrirani `Repository` nije akter ovog toka: procesor odlučuje **granicu** flusha, a pisanje
vodi platno prema write-strategiji ([`FRQ-PRC-15.22`](FRQ-PRC-15.22-document-flow.md)).

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | A1 pokreće procesor (`start`) |
| **EV02** | Generator je dobio blok od A3 |
| **EV03** | Dosegnuta granica flusha (broj stavki ili ciklus) |
| **EV04** | Dosegnuta granica prihvata — trajanje, broj blokova ili vanjski signal (`BR-08`) |
| **EV05** | A3 prijavljuje kvar: uređaj nestao (`BR-09`) ili gubitak iznad praga |

## 4. Preduvjeti

1. Konekcija i driver registrirani i provjereni **prije** prvog bloka (`BR-01`, `BR-06`).
2. Granica prihvata je deklarirana; prolaz bez granice nije dopušten (`BR-08`).
3. Barem jedan pipeline i barem jedno platno registrirani, po ugovoru iz `FRQ-PRC-15.3`.

## 5. Normalan tok

1. EV01 — A2 provjerava dostupnost uređaja preko drivera; nedostupan uređaj zaustavlja workflow
   prije prvog bloka (`BR-06`).
2. A2 otvara generator (`create_generator`) nad `A3.read`.
3. EV02 — za svaki blok A2 traži od A4 stvaranje dokumenta; sadržaj je blok, metapodaci su njegov
   opis iz `FRQ-DRV-16.2` §5 t.3, **uvećan za brojač gubitaka** zatečen u trenutku stvaranja.
4. A2 provodi stavku kroz svaki registrirani pipeline (A5), redom.
5. EV03 — A2 objavljuje granicu flusha; platno predaje seriju spremištu.
6. EV04 — granica prihvata: A2 zatvara generator, driver zatvara tok, konekcija oslobađa uređaj.
7. A2 zaključuje prolaz **sažetkom**: trajanje, broj blokova, broj uzoraka, gubici, djelotvorni
   parametri. To je jedinica posla za zapis (`NFRQ-OBS-03`), ne blok.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| uređaj nedostupan pri pokretanju | workflow ne započinje (`BR-06`); nijedan dokument nije stvoren |
| gubitak blokova ispod praga | prolaz se nastavlja; gubitak putuje u metapodacima dokumenta (§5 t.3) |
| gubitak iznad praga (EV05) | prolaz staje kao **kvar**; već stvoreni dokumenti ostaju, prolaz se označava parcijalnim |
| uređaj nestaje usred prolaza (`BR-09`) | isto kao gore: parcijalan rezultat, označen, ne tiho zatvoren |
| pipeline pada nad jednom stavkom | po ugovoru [`FRQ-PIP-15.2`](FRQ-PIP-15.2-pipeline.md) — jedan razred kvara; **prihvat se ne zaustavlja radi jedne stavke**, jer zaustavljanje gubi tok koji se ne može ponoviti |
| obrada je sporija od uređaja | gubitak nastaje na uređaju i broji se (`FRQ-DRV-16.2` §6); procesor ga ne skriva usporavanjem čitanja |
| granica prihvata nije deklarirana | kvar konfiguracije pri gradnji, ne beskonačan prolaz |

## 7. Rezultat

Prolaz daje niz dokumenata s blokovima uzoraka i njihovim opisom, jedan sažetak prolaza, i
**iskaz o cjelovitosti**: koliko je toka prihvaćeno, a koliko izgubljeno. Parcijalan prolaz je
označen kao parcijalan — nikad predan kao potpun.

## 8. Kriteriji prihvaćanja

Nijedan nije zadovoljen — koda nema.

1. `create_generator` isporučuje jednu stavku po bloku; ništa se ne skuplja preko granice flusha.
2. Dostupnost uređaja provjerena prije prvog dokumenta (`BR-06`).
3. Metapodaci dokumenta nose opis bloka **i** brojač gubitaka u trenutku stvaranja.
4. Prolaz staje po deklariranoj granici; prolaz bez granice odbijen pri gradnji (`BR-08`).
5. Kvar jedne stavke ne zaustavlja prihvat; kvar uređaja i gubitak iznad praga zaustavljaju.
6. Parcijalan prolaz nosi oznaku parcijalnosti u sažetku (`BR-09`).
7. Zapis: jedan sažetak po prolazu; bez retka po bloku (`NFRQ-OBS-03`).
8. Sadržaj uzoraka se ne pojavljuje u zapisu ni u poruci kvara (`NFRQ-SEC-06`).

## 9. Verifikacija

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 3 | `unittest` s lažnim driverom koji isporučuje N blokova; pregled metapodataka dokumenata | **nije izvedeno** — nema koda (2026-09-09) |
| 2, 4 | lažni driver koji prijavljuje nedostupnost; konfiguracija bez granice | nije izvedeno |
| 5, 6 | lažni pipeline koji pada; lažni driver koji prekida tok | nije izvedeno |
| 7, 8 | brojanje redaka zapisa i pretraga zapisa na sadržaj bloka | nije izvedeno |

Svaka tvrdnja uz **mutacijsku provjeru** ([`DR-WFL-023`](../04-DR/DR-WFL-023-test-framework-is-stdlib-unittest.md)).

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-16` §6](../01-HLRQ/HLRQ-16-sdr-capture.md#6-nefunkcionalni-zahtjevi-i-oscal).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-OBS-03` | jedinica posla za zapis je prolaz; sažetak, ne redak po bloku |
| `NFRQ-SEC-06` | ni uzorci ni dekodirani sadržaj ne ulaze u zapis; ulazi opis prihvata |
| `NFRQ-ORG-02` | ime imenuje ulogu i predmet; sámo ime nosi rječnik, ne ovaj zapis (`CLAUDE.md` §3.3) |
| `NFRQ-ORG-04` | koristi zatečene primitive; prihvat nije novi primitiv |
| OSCAL | `@oscal_processor` nad specijalizacijom, uz podskup iz `HLRQ-16` §6; vrata su inertna dok je `strict=False` ([`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.2) |

## 11. Otvoreno

1. **Nastavak iz mementa nije izvediv u zatečenom smislu.** `FRQ-PRC-15.3` daje procesoru jedinom
   sposobnost nastavka, ali uživo tok **nema odmotljivu poziciju**: uzorci koji su prošli dok je
   proces stajao ne postoje nigdje. Memento ovdje može bilježiti samo *što je učinjeno*
   (parametri, obuhvaćeno vrijeme, gubici), ne *odakle nastaviti*. Je li to i dalje memento po
   ugovoru ili izuzeće koje ugovor mora priznati — traži DR (D-03), i dira `concrete/`
   (`CLAUDE.md` §2.5).
2. **Granica prihvata kao vanjski signal** (`BR-08`): zaustavljanje izvana nema mehanizam u
   zatečenom skupu radnji procesora. Do odluke, izvedive su granice **trajanje** i **broj
   blokova**.
3. **Više obrada nad istim prihvatom.** Dva pipelinea nad istim blokom rade; dva *procesora* nad
   istim uređajem ne, jer je zauzeće isključivo ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
   §2). Je li rješenje jedan procesor s više pipelinea ili razdvajanje toka — otvoreno.
4. **Prag gubitka** dijeli se s [`FRQ-DRV-16.2`](FRQ-DRV-16.2-iq-stream.md) §11 t.1; definicija
   živi na jednom mjestu (D-12), a mjesto još nije odabrano.
