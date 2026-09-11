# HLRQ-16 — Razmjena uzoraka s radijskim uređajem (SDR)

> **Oznaka je provizorna.** Broj `16` je sljedeći slobodan u indeksu, ali razred `HLRQ` još nije
> u registru zahtjeva (`CLAUDE.md` §3.6); uvođenje traži DR (D-12). Kategorije `CON`, `DRV` i
> `PRC` **jesu** u vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Djelomično provedeno 2026-09-11:** faza 1 (RTL2832U, prijam) — konekcija, driver i procesor u `blackwattle/src/`, ovjereni lažnom obitelji; ni uređaj ni knjižnica u testu nisu bili (`FRQ-*-16.*` §9). Ranije: **prijedlog.** Provjereno 2026-09-09: u `blackwattle/src/` nema nijedne SDR, USB ni uređajne komponente (pretraga `src/` po imenima SDR i USB knjižnica → prazno) |
| **Naziv** | ~~`SDRScanner`~~ — **povučen 2026-09-11** (autor): sposobnost ne dobiva vlastito ime; klase nose imena po konvenciji `NFRQ-ORG-02` (konekcija i driver po predmetu SDR), a razdvajanje čitanja, pisanja ili skena, ako zatreba, pripada sloju konekcije (§7 t.9). Izvorni opis: softverski definiran radio kao računalni radijski skener, podrijetlom iz masovno proizvedenih DVB-T dongla s čipom RTL2832U ([RTL-SDR.com](https://www.rtl-sdr.com/about-rtl-sdr/)). Ulazak u rječnik diskursa traži DR (D-12) — nije otvoren |
| **Odluka** | **Nijedan DR nije otvoren.** Os podjele (pristupni mehanizam, ne proizvođač) i profil modela prijedlog su ovog zapisa (§4a, `BR-11`); odluka traži DR (D-03). Ostale odluke su u §7 |
| **Podloga** | [raznolikost SDR uređaja](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md) (2026-09-09; izmjereno na RTL2832U + R820T) · [tržište uređaja](../06-ANALYSIS/2026-09-11-sdr-device-market.md) (2026-09-11) |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `blackwattle` — pristup uređaju traži third-party knjižnicu, dakle izvan clean core tiera (`CLAUDE.md` §7.1, §7.4). Vidi §7 t.1 i t.5 |
| **Djeca** | [`FRQ-CON-16.1`](../02-FRQ/FRQ-CON-16.1-sdr-device.md) · [`FRQ-DRV-16.2`](../02-FRQ/FRQ-DRV-16.2-iq-stream.md) · [`FRQ-PRC-16.3`](../02-FRQ/FRQ-PRC-16.3-sdr-capture.md) · `FRQ-DOC-16.4`, `FRQ-PIP-16.5` (kandidati, §4) |
| **Dijagrami** | [dekompozicija](HLRQ-16-sdr-decomposition.puml) (klasni) · [sastavljanje](HLRQ-16-assembly-sequence.puml) (sekvencijski) · konekcija: [klase](../02-FRQ/FRQ-CON-16.1-class.puml), [stanje uređaja](../02-FRQ/FRQ-CON-16.1-device-state.puml) · driver: [klase](../02-FRQ/FRQ-DRV-16.2-class.puml), [čitanje](../02-FRQ/FRQ-DRV-16.2-read-sequence.puml), [slanje](../02-FRQ/FRQ-DRV-16.2-write-sequence.puml) · procesor: [klase](../02-FRQ/FRQ-PRC-16.3-class.puml), [prolaz](../02-FRQ/FRQ-PRC-16.3-capture-sequence.puml) — pogledi, ne izvor istine (D-13) |
| **Norme** | `NFRQ-ORG-08` (dekompozicija i ponovna uporaba — obvezujuće, `BR-13`) · `NFRQ-ORG-01` (lokalnost helpera) · `NFRQ-SEC-07` (ovlast za odašiljanje) |
| **Presedan** | [`HLRQ-13`](HLRQ-13-llm-models.md) §4 — zajednički ugovor konekcije; ovaj zahtjev ga nasljeđuje i proširuje na uređaj |
| **Kaskada** | filozofija → *policy* (`CLAUDE.md`) → doktrina → metoda → **registar**. Nosivi postulati: **P-11** (modul je granica oko odluke koja se može promijeniti — korijen `BR-13`) · **P-14** (zapis je specifikacija znanja, ne dnevnik rada) · **P-21** (spajanje bez ugovora nije kompozicija) · **P-08** (imenovanje je semiotička politika) · **P-20** (prikaz nikad nije izvor istine) |
| **Sljedivost** | `NFRQ-ORG-04` (ontologija: `Connection`, `Driver`, `Processor`) · `NFRQ-SEC-01` · `NFRQ-SEC-03` · `NFRQ-OBS-03` (volumen zapisa) · [`FRQ-PRC-15.3`](../02-FRQ/FRQ-PRC-15.3-processor.md) (ugovor prolaza) · [`FRQ-BBD-15.1`](../02-FRQ/FRQ-BBD-15.1-blackboard.md) (platno između stopa) |

> **Ovaj zapis nije dnevnik rada nego specifikacija znanja (P-14).** Registar namjerno nosi **oba
> smjera**: zapise koji dokumentiraju **zatečeni kod** obrnutim inženjerstvom (`CLAUDE.md` §4 t.1)
> i zapise koji, kao ovaj, **prethode kodu**. Oblik zapisa je isti u oba slučaja — razlikuju se
> samo **Status** i **§Verifikacija**, jer se ondje vidi je li tvrdnja izmjerena ili tek postavljena
> (D-05). Dokumentacija je time dio dizajna: pregledava se, mijenja i odobrava **prije** koda, a
> ono što je već sagrađeno ulazi u isti registar bez druge forme.
>
> **Imena klasa u ovom tekstu ne stoje** (`CLAUDE.md` §3.3, P-08). Proza govori o ulogama i
> obiteljima; imena nose rječnik i [dijagrami](HLRQ-16-sdr-decomposition.puml). Posljedica je
> praktična: preimenovanje mijenja jedan zapis u rječniku, ne N mjesta u tekstovima — a novo ime
> ulazi kroz odluku, ne pojavom u rečenici.

## 1. Narativ

Workflow treba čitati podatke s **radijskog prijamnika na USB priključku** (SDR): uređaj isporučuje
neprekidan tok kompleksnih uzoraka (IQ), a sve što se iz njih dobiva — demodulacija, dekodiranje,
spektar — transformacija je nad tim tokom. Kad bi postojali konekcija koja drži uređaj i driver
koji s njega čita blokove uzoraka, procesor bi vodio prolaz, a pipeline bi obavljao transformaciju
**čitanjem sadržaja kroz driver** — ista konstelacija kao kod jezičnih modela
([`HLRQ-13`](HLRQ-13-llm-models.md) §1).

**Zašto.** Gotovi SDR alati (`rtl_fm`, `rtl_power`, GNU Radio graf) vežu prihvat, demodulaciju i
zapis u jedan proces s fiksnim izlazom. Razdvajanjem protoka od algoritma isti par
konekcija+driver poslužuje više obrada — dekodiranje APT slike, sken spektra, ADS-B — a promjena
frekvencije, pojačanja ili uređaja postaje **izmjena konfiguracije, ne koda**.

**Opseg su oba smjera, ali ne kao jedan ugovor.** Zahtjev pokriva **prijam** i **odašiljanje**.
Prijam je zajednički svim uređajima; odašiljanje ima drugi protokol, drugo stanje uređaja i drugu
ovlast, pa je unutar iste sposobnosti **odvojen ugovor** (`BR-12`), a ne način rada prijama.

**Poluduplex, ne puni duplex.** Prema proizvođaču HackRF One „can transmit or receive but not both
at the same time" ([dokumentacija](https://hackrf.readthedocs.io/en/latest/hackrf_one.html),
provjereno 2026-09-09). Smjer je zato **stanje uređaja**, ne svojstvo drivera niti dvije
istovremene sesije — vidi [dijagram stanja](../02-FRQ/FRQ-CON-16.1-device-state.puml).

**Zatečeno.** Ništa od ovoga nije zatečeno (§Status). Presedan za neprekidan izvor je
zatečena konekcija za tok brodskih poruka (`connections/aisstream.py`): konekcija drži
konfiguraciju, a **procesor** vodi životni ciklus sesije —
što je u napetosti s ugovorom konekcije iz `HLRQ-13` §4 (§7 t.3).

## 2. Zašto uređaj nije samo još jedan pod-sustav

Zatečene konekcije dosežu **mrežu** ili **sustav datoteka**. Lokalni uređaj nije ni jedno, i tri
njegova svojstva mijenjaju ugovor, ne samo izvedbu:

| svojstvo | posljedica |
|---|---|
| **isključivo zauzeće** | uređaj drži **jedan** proces; druga konekcija nad istim uređajem nije sporija, nego nemoguća |
| **tok se ne da zaustaviti** | uzorci nastaju u stvarnom vremenu; blok koji se ne preuzme na vrijeme je **izgubljen**, ne odgođen |
| **traženo ≠ postignuto** | uređaj zaokružuje ili odbija frekvenciju, stopu i pojačanje; djelotvorna vrijednost čita se natrag, ne pretpostavlja |
| **sposobnosti se razlikuju po jedinici** | model pojačanja, dopuštene stope, format uzorka i ulazi razlikuju se među modelima — **deklarira ih profil**, a ispitano ga provjerava (§4a, `BR-11`) |

Prva dva svojstva zatečeni framework ne pokriva: `hot_swap` postoji na konekciji, ali protutlaka
(*backpressure*) nema — blackboard je platno između stope po jedinici i stope serije
([`FRQ-BBD-15.1`](../02-FRQ/FRQ-BBD-15.1-blackboard.md)), a izvor koji se ne da usporiti je nov
slučaj (§7 t.4).

## 3. Mjesto u dekompoziciji

| sloj | briga | primitiv | kategorija |
|---|---|---|---|
| **Pristup** | otvaranje uređaja, isključivo zauzeće, parametri tunera | `Connection` | `CON` |
| **Dohvat** | čitanje blokova uzoraka, brojanje gubitaka | `Driver` | `DRV` |
| **Vođenje prolaza** | generator, platno, granica flusha, granica prihvata | `Processor` | `PRC` |
| **Transformacija** | demodulacija, dekodiranje, sken | `Pipeline` | `PIP` |

Ontologija se **ne proširuje** (`NFRQ-ORG-04`): uređaj je pod-sustav kojem konekcija daje pristup,
a ne novi domenski primitiv.

## 4. Opseg

| oznaka | predmet | dokument |
|---|---|---|
| `FRQ-CON-16.1` | Konekcija prema uređaju: otvaranje po serijskom broju, primjena i **očitavanje** parametara tunera | [FRQ-CON-16.1](../02-FRQ/FRQ-CON-16.1-sdr-device.md) |
| `FRQ-DRV-16.2` | Driver nad tom konekcijom: `read` blokova IQ uzoraka, evidencija prekoračenja | [FRQ-DRV-16.2](../02-FRQ/FRQ-DRV-16.2-iq-stream.md) |
| `FRQ-PRC-16.3` | Procesor prihvata: generator nad driverom, dokument po bloku, granica prihvata i flusha | [FRQ-PRC-16.3](../02-FRQ/FRQ-PRC-16.3-sdr-capture.md) |
| `FRQ-DOC-16.4` | Dokument koji nosi blok uzoraka i njegove metapodatke — **kandidat**; nijedan zatečeni razred dokumenta ne nosi sirovi binarni međuspremnik uz opis uzorkovanja | — |
| `FRQ-PRC-16.6` | Prolaz **odašiljanja** — kandidat; `FRQ-PRC-16.3` pokriva samo prihvat, a odašiljanje je drugi prolaz s drugom granicom i ovlašću (`BR-12`) | — |
| `FRQ-PIP-16.5` | Pipeline kao konzument — **kandidat**. Demodulacija je transformacija, dakle pipeline, a njezini se koraci (filtar, demodulator, dekoder) grade kao **helperi** po `BR-13`; koji su i kako se slažu ovisi o dizajnu workflowa i piše se s njim, ne unaprijed. Čeka i kanal pipeline→driver otvoren u [`HLRQ-13`](HLRQ-13-llm-models.md) §7 t.3 | — |

**Os podjele: pristupni mehanizam, ne proizvođač** (prijedlog, bez DR-a). Jedna *instanca* konekcije = jedan fizički prijamnik, jer je zauzeće isključivo (§2);
dva prijamnika u istom workflowu su dvije registrirane konekcije **iste klase**. Nova se klasa
piše kad se promijeni ugovor ili mehanizam, nikad zato što se razlikuje model uređaja — vidi §4a.

### 4a. Više proizvođača bez drivera po uređaju

Naivni odgovor — par konekcija+driver po uređaju — daje N klasa s istim ponašanjem i različitim
konstantama (`NFRQ-ORG-08` duplikat, `NFRQ-SEC-02` površina bez dobitka). Analiza pokazuje da je
os pogrešna: od devet osi razlike među proizvođačima **sedam su podatak**, a kod traže samo dvije.

| razina promjene | odgovor | koliko klasa |
|---|---|---|
| **ugovor** — što se od uređaja može tražiti | nova konekcija/driver | rijetko |
| **mehanizam** — čime se do uređaja dolazi (knjižnica domaćina) | **prilagodnik po obitelji** uz zajedničku konekciju i driver | jednom po knjižnici |
| **vrijednost** — ime stupnja, raspon, format, adresa | **profil modela** u konfiguraciji, provjeren ispitivanjem | nijedna |

**Test:** *mijenja li razlika ono što se od uređaja može tražiti, ili samo ono što se dobiva kao
odgovor?* Prvo je klasa, drugo je konfiguracija. Uz apstrakcijski sloj to je **jedan** par za
cijelu obitelj uređaja; uz knjižnicu po proizvođaču jedan po knjižnici — ni u jednom slučaju jedan
po *modelu*.

**Provjereno na sljedećem izglednom uređaju.** Od HackRF One prijamna strana ne traži nijednu novu
klasu: širi doseg (1 MHz – 6 GHz), tri stupnja pojačanja umjesto jednog i drugi format uzorka sve
su vrijednosti **profila** (`BR-11`). Klasu traži tek njegovo **odašiljanje** — jer mijenja
ugovor, ne vrijednost (`BR-12`, [analiza](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md) §7).

**Gdje razlika ulazi u kod — točno dvije točke** (`BR-13`, [klasni dijagram](HLRQ-16-sdr-decomposition.puml)):

| što se razlikuje | gdje ulazi | što **ne** nastaje |
|---|---|---|
| doseg, stope, stupnjevi pojačanja, ulazi | **profil modela**, provjeren ispitivanjem (`BR-11`) | grana po uređaju |
| format uzorka i puna skala | **helper pretvorbe** — parser u smjeru čitanja, formatter u smjeru pisanja — koji tvornica vrati za format iz profila | klasa po proizvođaču |
| ima li uređaj odašiljanje | skup smjerova u sposobnostima; `write` postoji ili pada tipom | druga konekcija za isti uređaj |

Prijamna putanja je time **jedna sekvenca za sve uređaje**
([dijagram](../02-FRQ/FRQ-DRV-16.2-read-sequence.puml)); jedina točka u kojoj razlika ulazi je izbor
helpera pretvorbe. To je zatečeni obrazac frameworka — obitelj parsera i formattera s tvornicom
koja bira po tipu sadržaja — a ne nov mehanizam. Imena nose rječnik i
[dijagrami](HLRQ-16-sdr-decomposition.puml), ne ovaj tekst (§3.3).

Ovo je prijedlog, ne odluka — DR nije otvoren (D-03): **koji**
mehanizam ostaje otvoreno (§7 t.1). Tvrdnja da je put preko alata obitelji (`rtl_*`) „mjerenjem otpao" **povučena je 2026-09-11**:
zaključak je izveden iz teksta pomoći, a ne izmjeren. Izvorni kod alata pokazuje: uređaj se može
adresirati **serijskim brojem**, ali alat uzima prvi pogodak i dvoznačnost ne prijavljuje; frekvencija
i stopa u ispisu su **tražene**, ne očitane vrijednosti, pa `BR-05` preko alata ostaje djelomičan.
Put preko alata ponovno je opcija ([analiza pristupa](../06-ANALYSIS/2026-09-11-sdr-access-paths.md)).

### 4b. Opseg uređaja i nužna funkcionalnost

**Opseg (prijedlog, [analiza tržišta](../06-ANALYSIS/2026-09-11-sdr-device-market.md)).** Jedinica
podrške je obitelj — knjižnica domaćina — a model unutar nje je **profil u konfiguraciji**, ne klasa.

| faza | obitelj | profili | smjer |
|---|---|---|---|
| 1 | RTL2832U (`librtlsdr`) | 6 tunera + inačica ploče V4 = **7** | prijam |
| 2 | HackRF (`libhackrf`) | One, Pro = **2** | prijam; odašiljanje uz `NFRQ-SEC-07` |

Ukupno **9 profila i 2 prilagodnika**. Izvan opsega do odluke: ostale obitelji (prilagodnik po
obitelji) i tri razlike koje su **ugovor**, a ne profil — puni duplex, više koherentnih kanala i
zatvoreni binarni API proizvođača (analiza §4).

**Nužna funkcionalnost za ispravan rad** — opseg, ne koraci; korake nose djeca:

| # | funkcija | pravilo | nosi |
|---|---|---|---|
| 1 | pronaći uređaj i razriješiti selektor na točno jedan | `BR-02` | `FRQ-CON-16.1` |
| 2 | prepoznati model, potvrditi profil ispitivanjem, potvrditi da mehanizam podržava model | `BR-11`, `BR-14` | `FRQ-CON-16.1` |
| 3 | zauzeti uređaj isključivo i osloboditi ga i pri kvaru | `BR-03` | `FRQ-CON-16.1` |
| 4 | postaviti parametre i očitati djelotvorne | `BR-04`, `BR-05` | `FRQ-CON-16.1` |
| 5 | isporučivati blokove s opisom (format, puna skala, vrijeme, redni broj) | `BR-13` | `FRQ-DRV-16.2` |
| 6 | pretvoriti format uzorka helperom izabranim po formatu | `BR-13` | `FRQ-DRV-16.2` |
| 7 | brojati izgubljene uzorke i zaustaviti se iznad praga | `BR-07` | `FRQ-DRV-16.2`, `FRQ-PRC-16.3` |
| 8 | završiti po deklariranoj granici; označiti parcijalan prolaz | `BR-08`, `BR-09` | `FRQ-PRC-16.3` |
| 9 | predati blok platnu kao dokument | — | `FRQ-DOC-16.4` (**nije napisan**) |
| 10 | zapisati prolaz sažetkom, ne po bloku | `NFRQ-OBS-03` | `FRQ-PRC-16.3` |
| 11 | *transceiver:* smjer kao stanje uređaja | `BR-12` | `FRQ-CON-16.1` |
| 12 | *transceiver:* ovlast prije emisije, zapis svake emisije | `NFRQ-SEC-07` | `FRQ-CON-16.1`, `FRQ-DRV-16.2` |
| 13 | *transceiver:* isporuka blokova za odašiljanje | `BR-12`, `BR-13` | `FRQ-DRV-16.2`; strategija zapisa u repozitoriju (`FRQ-PRC-16.6` povučen 2026-09-11) |
| 14 | *skener:* promijeniti frekvenciju unutar prolaza po planu ugađanja, uz provjeru raspona i odbacivanje blokova dok se petlja ne zaključa — **prijedlog**; razdvajanje skena pripada sloju konekcije (autor, 2026-09-11; §7 t.9), razred B [analize primjena](../06-ANALYSIS/2026-09-11-sdr-applications.md) | `BR-15` | `FRQ-CON-16.1`, `FRQ-DRV-16.2` |

## 5. Poslovna pravila

Vrijede za cijelu sposobnost; djeca ih nasljeđuju i pozivaju se na oznaku.

| oznaka | pravilo |
|---|---|
| **BR-01** | Konekcija i driver moraju biti **registrirani u workflowu** i s valjanom konfiguracijom. |
| **BR-02** | Uređaj se adresira **selektorom** — skupom ključ-vrijednost čiji oblik pripada mehanizmu (serijski broj, URI, ključ proizvođača). Selektor mora razriješiti **točno jedan** uređaj: nula je *dosežnost*, više od jednog *dvoznačnost*. „Uzmi prvi” nije dopušteno — redni indeks se mijenja pri ponovnom uključenju i tiho pokazuje na drugi uređaj. |
| **BR-03** | Zauzeće je **isključivo**: konekcija koja ne može dobiti uređaj ne čeka neodređeno, nego prijavljuje razred kvara *zauzetost*. |
| **BR-04** | Parametri tunera (stopa uzorkovanja, središnja frekvencija, pojačanje, korekcija takta) su **konfiguracija**, ne kod. Pojačanje je **karta imenovanih stupnjeva** uz zaseban ključ načina (`auto`/`manual`) — skalar je poseban slučaj uređaja s jednim stupnjem, ne opći oblik. |
| **BR-05** | Nakon primjene parametra čita se **djelotvorna vrijednost** s uređaja i ona ulazi u zapis; tražena vrijednost nije dokaz postignute (`NFRQ-DEF-02`). |
| **BR-06** | Dostupnost uređaja provjerava se **pri pokretanju workflowa**, prije prvog bloka; neriješen kvar zaustavlja workflow, kao `HLRQ-13` `BR-06`. |
| **BR-07** | **Izgubljeni uzorci se broje i prijavljuju.** Prekoračenje međuspremnika je nalaz o vjerodostojnosti podataka, ne tehnički detalj koji se prešućuje. |
| **BR-08** | Prihvat završava po **deklariranoj granici** (trajanje, broj blokova, vanjski signal), nikad „dok ne stane"; granica je konfiguracija. |
| **BR-09** | Iznenadno odspajanje uređaja je **kvar prihvata**, ne uredan kraj: parcijalni rezultat se označava kao takav. |
| **BR-10** | Snimljeni spektar može sadržavati tuđe komunikacije; što se smije zapisati i zadržati ograničeno je propisom, ne konfiguracijom (§7 t.6). |
| **BR-11** | Sposobnosti modela (doseg, dopuštene stope, stupnjevi pojačanja, formati uzorka, ulazi, smjerovi) **deklarira profil** u konfiguraciji; što mehanizam zna reći, **ispituje se** i mora se slagati s profilom. Neslaganje je kvar uspostave, a ne tiho odabrana strana; vrijednost instance izvan profila je kvar konfiguracije. |
| **BR-12** | **Smjer je stanje uređaja, ne parametar poziva.** Prijam i odašiljanje se na poluduplex uređaju isključuju. Odašiljanje je odvojen ugovor: drugi protokol i vlastita ovlast (`NFRQ-SEC-07`). Uređaj koji ga ne prijavljuje među svojim sposobnostima nema dohvatljivu putanju pisanja — `write` ondje pada tipom, ne tiho. |
| **BR-13** | **Ono što dijeli ugovor, dijeli i kod.** Granica komponente povlači se oko **odluke koja se može promijeniti** (P-11) — ovdje je to model uređaja — pa razlika među uređajima ostaje skrivena iza profila, prilagodnika i helpera, a ne curi u ostatak sustava. Prijamna putanja je jedna izvedba za sve uređaje; razlika među njima ulazi kao **vrijednost profila** (`BR-11`) i kao **helper koji tvornica vrati** za taj format — nikad kao grana po proizvođaču. Obrnuto vrijedi jednako: putanje koje služe **različitim ugovorima** (čitanje i pisanje) ne spajaju se, jer spajanje bez zajedničkog ugovora nije kompozicija nego konkatenacija (P-21). Norma: `NFRQ-ORG-08` k.2 i k.4; smještaj helpera: `NFRQ-ORG-01`. |
| **BR-14** | **Mehanizam mora podržavati deklarirani model.** Unutar iste obitelji inačica knjižnice odlučuje radi li model ispravno — RTL-SDR V4 sa standardnim driverom daje signale na krivoj frekvenciji (proizvođač). Provjera prethodi prvom bloku, a neuspjeh je **glasan** kvar uspostave: tih kvar, u kojem podaci izgledaju valjano a nisu, najgori je ishod ove sposobnosti. |
| **BR-15** | **Raspon frekvencija je granica, ne preporuka.** Raspon koji uređaj podržava mora biti konfigurabilan za svaki uređaj (profil modela) i, gdje uređaj to zna reći, dostupan čitanjem s uređaja (`BR-11`). Provjerava se na **dva mjesta**: *pri učitavanju konfiguracije* — frekvencija ili plan ugađanja izvan raspona je kvar konfiguracije i workflow ne započinje; *pri svakom zahtjevu za čitanje na novoj frekvenciji* — driver provjerava je li tražena frekvencija u dopuštenom rasponu, a ako nije, **ne ugađa uređaj** i vraća kvar razreda *izvan raspona* koji imenuje traženu vrijednost i dopušteni raspon. |

> `BR-05`, `BR-07`, `BR-09` su **prijedlog** (2026-09-09), izveden iz svojstava uređaja iz §2, ne
> iz zatečenog koda — koda nema. `BR-05` je otad **izmjeren** (29 diskretnih vrijednosti
> pojačanja na R820T), pa stoji na svjedočanstvu. `BR-02`, `BR-04`, `BR-11` i `BR-14` prijedlog su
> ovog zapisa; DR nije otvoren, pa su i pravila prijedlog. `BR-12` počiva na provjerenoj dokumentaciji transceivera, ne na mjerenju —
> nijedan takav uređaj nije bio na mjernoj platformi. Potvrda je na autoru (D-05).

## 6. Nefunkcionalni zahtjevi i OSCAL

| NFR | posljedica za ovu sposobnost |
|---|---|
| `NFRQ-SEC-01` blast radius | konekcija daje pristup **fizičkom prijamniku**; kompromitirana komponenta dobiva mogućnost slušanja, ne samo podatak. Jedna konekcija po uređaju drži doseg na jednom uređaju |
| `NFRQ-SEC-02` napadna površina | ugovor ostaje `connect`/`disconnect` + stanje; parametri tunera su konfiguracija (`ALLOWED`), ne nove metode |
| `NFRQ-SEC-03` supply-chain i lokalnost | driverska knjižnica i `libusb` su third-party → cijela sposobnost pripada `blackwattle`, uz odgođeni uvoz. Binarna sistemska ovisnost (`librtlsdr`) **nije pokrivena** ni lockom ni SBOM-om — deklarirana rupa (D-11) |
| `NFRQ-SEC-06` povjerljivost zapisa | ni sadržaj uzoraka ni dekodirani sadržaj ne citiraju se u zapisu ni u poruci greške; zapisuje se opis prihvata (frekvencija, stopa, broj blokova), ne sadržaj |
| `NFRQ-OBS-03` volumen zapisa | pri 2,4 MS/s blok od 256 kiB traje ~55 ms; zapis po bloku je poplava. Jedinica posla za zapis je **prolaz**, ne blok — sažetak i pragovi, ne redak po bloku |
| `NFRQ-ORG-02` nomenklatura | ime imenuje **ulogu**, nikad ne bude gola generička imenica („upravitelj"). Imena se ovdje ne navode — nosi ih rječnik, i to je mehanizam protiv širenja ontologije (`CLAUDE.md` §3.3). Zapis akronima u identifikatorima visi o [`DR-WFL-004`](../04-DR/DR-WFL-004-acronym-identifier-casing.md); do odluke lint upozorava i ne ruši build (§2.8) |
| `NFRQ-ORG-04` sposobnost vs primitiv | koriste se zatečeni primitivi; ontologija se ne proširuje (§3) |

### OSCAL — prvi slučaj podskupa

Zatečene konekcije deklariraju puni skup `("ac-3", "ia-5", "sc-8", "sc-13")`. **Lokalni uređaj
nema ni račun ni prijenos preko mreže ni kriptografiju**, pa bi deklaracija tog skupa bila tvrdnja
o kontroli koju klasa ne provodi — što `HLRQ-13` §6 t.2 izrijekom zabranjuje, uz napomenu da takav
pod-sustav „više ne postoji". **Ovaj zahtjev je taj slučaj.** Kandidat je podskup s pristupom
(`ac-3`); potvrda skupa traži sign-off, jer proširenje ili sužavanje deklaracije nije stvar
procjene (§7 t.9).

Vrata su i dalje **inertna** dok je `strict=False` i dok nijedno mjesto ne predaje `oscal_policy=`
([`HLRQ-14`](HLRQ-14-oscal.md) §7 t.2) — zatečeno stanje, ne izuzeće koje ovaj zahtjev uvodi.

## 7. Otvoreno

Odluke koje ovaj dokument **ne donosi**; svaka traži DR (D-03).

1. **Pristupni mehanizam — djelomično riješeno 2026-09-11.** Autor je odbio apstrakcijski sloj
   treće strane koji je predložio agent, jer izbor nije bio obrazložen. Predloženo umjesto toga (prijedlog, bez DR-a): framework je sam apstrakcija —
   zajednička konekcija i driver, **prilagodnik po obitelji** kao **helper** bez temeljne klase
   primitiva (`NFRQ-ORG-04`; obitelj adaptera iz jezgre u kriteriju pripada dokumentima), razlike modela u **profilu** (`BR-11`). Put preko alata obitelji ponovno je opcija — tvrdnja da je „mjerenjem otpao" povučena je 2026-09-11 (§4a).
   **Otvoreno:** opseg obitelji (§4b).
   **Odluka autora 2026-09-11:** za obitelj RTL2832U mehanizam je knjižnica `pyrtlsdr`, kao vanjska
   knjižnica u `blackwattle` (put A [analize pristupnih puteva](../06-ANALYSIS/2026-09-11-sdr-access-paths.md)).
   Lokalna provjera (analiza §6): uz sistemsku knjižnicu uvoz **pada**, a uz ugrađenu radi —
   potvrda `BR-14` na razini uvoza. Licenca i varijanta s ugrađenom knjižnicom su otvorene (t.2).
2. **Skupina extras u `pyproject.toml`.** Zatečene su `security`, `sql`, `search`, `streaming`,
   `cloud`, `data`, `documents`, `nlp`, `media`. Nova skupina `sdr` ili smještaj u `streaming` —
   otvoreno; nova skupina je proširenje deklarirane površine distribucije. Autor 2026-09-11: `pyrtlsdr` ulazi
   među vanjske knjižnice; otvoreni su skupina, varijanta s ugrađenom knjižnicom i licenca (GPL-3.0-or-later
   uz distribuciju pod Apache 2.0 — ovdje se ne tumači).
   **Odlučeno 2026-09-11 (autor):** skupina `sdr` s `pyrtlsdr[lib]`, najnovije inačice kao donje granice;
   licenca je **prihvatljiva** i označena u `pyproject.toml` (GPL-3.0-or-later za paket, GPL-2.0-or-later
   za knjižnicu uređaja). Skupina nije u `all`: licenca i ugrađena binarna datoteka biraju se imenom.
3. **Tko vodi životni ciklus sesije.** `HLRQ-13` §4 daje ga konekciji; zaglavlje
   `connections/aisstream.py` daje ga **procesoru** za neprekidan izvor. Dva presedana za isti
   oblik problema; SDR pada u isti razred i ne smije se odlučiti prešutno.
4. **Protutlak.** Što se događa kad platno ne stigne primiti blok: odbaci najstariji, odbaci
   najnoviji, ili zaustavi prihvat. Odluka dira `concrete/` (`CLAUDE.md` §2.5) → serija `DR-WFL`.
5. **Ime distribucije — razriješeno 2026-09-09.** Nalaz je glasio da `CLAUDE.md` §1 i §7.4
   govore o `wattleflow-processors`, a distribucija u ovom stablu zove se `blackwattle`
   (`pyproject.toml`, `name = "blackwattle"`). Razilaženje je zatvoreno provedbom
   [`DR-PRC-002`](../04-DR/DR-PRC-002-blackwattle-rename.md) u proznom sloju: policy i registri
   od tog datuma govore `blackwattle`. **Ostaje otvoreno pakiranje** — PyPI ime i
   `pyproject.toml` kodnog stabla nisu obuhvaćeni tom odlukom (t.3), pa razilaženje teksta i
   artefakta traje i vodi se u `workflow/TODO.md`.
6. **Propisna granica prihvata (`BR-10`).** Što se smije primiti, zapisati i zadržati uređeno je
   izvan ovog registra (*Radiocommunications Act 1992*, *Telecommunications (Interception and
   Access) Act 1979*, *Privacy Act 1988* APP 3/APP 11 — usp.
   [`DR-WFL-008`](../04-DR/DR-WFL-008-audit-record-content.md)). Ovdje se **ne tumači propis**;
   bilježi se da granica postoji, da je pravna, i da bez nje `BR-10` nema kriterij prihvaćanja.
7. **OSCAL podskup** (§6): koje kontrole lokalni uređaj stvarno provodi. Prvi slučaj podskupa u
   registru; bez sign-offa deklaracija ostaje neupisana, a ne pogođena.
8. **Nastavak prihvata iz mementa.** `FRQ-PRC-15.3` daje procesoru jedinom sposobnost nastavka;
   uživo tok nema odmotljivu poziciju. Vidi [`FRQ-PRC-16.3`](../02-FRQ/FRQ-PRC-16.3-sdr-capture.md) §11.
9. **Razdvajanje čitanja, pisanja i skena — sloj konekcije** (autor, 2026-09-11). Ako se smjerovi
   ili sken moraju razdvajati, razlika živi u sloju konekcije (stanje i konfiguracija uređaja, `BR-12`), ne u
   novim imenima ni u vlastitom imenu sposobnosti. **Odlučeno i provedeno 2026-09-11** (autor: „ažuriraj"):
   jedan procesor prihvata; plan ugađanja je konfiguracija konekcije; odašiljanje stiže kroz repozitorij
   čija strategija zapisa zove `write` drivera — procesora po smjeru nema. Dijagrami usklađeni s kodom.
