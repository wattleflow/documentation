# FRQ-DRV-16.2 — Driver za razmjenu uzoraka s SDR uređajem

> **Oznaka je provizorna** koliko i razred `HLRQ` iznad nje (`CLAUDE.md` §3.6). Kategorija `DRV`
> je u vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Prijedlog — nije provedeno** (provjereno 2026-09-09) |
| **Odluka** | [`DR-PRC-003`](../04-DR/DR-PRC-003-sdr-access-layer.md) — **prijedlog**: driver po pristupnom mehanizmu, format uzorka se ispituje. Ostalo otvoreno ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §7 t.1, t.4) |
| **Podloga** | [raznolikost SDR uređaja](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md) (2026-09-09) |
| **Nadređeni zahtjev** | [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) — `BR-01…BR-10` |
| **Predmet** | `DriverSdrDevice(GenericDriver)` — `read` blokova IQ uzoraka i `write` na uređaju koji odašilje; jedan driver, dva protokola |
| **Dijagrami** | [putanja čitanja](FRQ-DRV-16.2-read-sequence.puml) (sekvencijski) · [dekompozicija](../01-HLRQ/HLRQ-16-sdr-decomposition.puml) (klasni) — pogledi, ne izvor istine (D-13) |
| **Kaskada** | P-11 (granica oko promjenjive odluke) · P-14 (specifikacija znanja, ne dnevnik) · P-08 (proza nosi uloge, rječnik imena) |
| **Norme** | `NFRQ-ORG-08` k.2, k.4 (obvezujuće preko `BR-13`) · `NFRQ-ORG-01` · `NFRQ-SEC-07` (za `write`) · `NFRQ-OBS-03` |
| **Sestrinski** | [`FRQ-CON-16.1`](FRQ-CON-16.1-sdr-device.md) (pristup uređaju) · [`FRQ-PRC-16.3`](FRQ-PRC-16.3-sdr-capture.md) (vođenje prolaza) |
| **Izvedba** | — (predložena putanja: `drivers/sdr.py`) |

## 1. Predmet

Driver je **jedina** komponenta koja dodiruje tok uzoraka. Konekcija drži uređaj i parametre
(`FRQ-CON-16.1`), procesor vodi prolaz (`FRQ-PRC-16.3`), a pipeline transformira ono što driver
isporuči. Uzorak je kompleksan (I i Q), pa je jedinica isporuke **blok** — niz uzoraka s jednim
opisom uzorkovanja, a ne pojedini uzorak.

Driver je, kao i konekcija, **po pristupnom mehanizmu** ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
§4a): format uzorka i puna skala razlikuju se među uređajima, ali su **odgovor uređaja**, ne grana
u kodu. Driver ih preuzima od konekcije (`BR-11`) i ne nosi tablicu po modelu.

**Oba smjera nose isti javni API, kao svaki driver u frameworku.** Presedan je zatečeni driver za
brokera poruka (`drivers/kafka.py`):
jedan driver, `read` i `write`, dva različita protokola ispod (Consumer/Producer) i tip konekcije
po smjeru. SDR se na to preslikava izravno — s jednom razlikom koja se mora poštovati: Kafka
consumer i producer **mogu postojati istovremeno**, a poluduplex uređaj ne može
([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §1). Smjer je zato stanje konekcije, ne dvije
usporedne sesije istog drivera.

Gdje uređaj **ne prijavljuje** odašiljanje, `write` pada **tipom**, ne tihim ne-radom
(`NFRQ-SEC-07` k.1) — po pravilu da tip pozivnog mjesta iskazuje koja je operacija smislena
([`FRQ-STR-15.4`](FRQ-STR-15.4-strategy.md)).

### Što se dijeli, a što ne (`BR-13`)

| putanja | ugovor | izvedba |
|---|---|---|
| `read` | isti za svaki uređaj | **jedna** izvedba; razlika ulazi kroz ispitane vrijednosti i kroz **parser** koji tvornica vrati za ispitani format |
| `write` | drugi ugovor: drugi protokol, stanje, ovlast | **odvojena** izvedba uz **formatter**; ne spaja se s `read` (`NFRQ-ORG-08` k.4 — spajanje bi bilo regresija) |

Pretvorba bajtova u uzorke i natrag je **helper**, ne driverska grana: obitelj parsera i formattera
uz tvornicu koja bira po tipu sadržaja, kako to framework već radi za ostale formate.
Smještaj po `NFRQ-ORG-01`: format uzoraka služi ovoj domeni, pa helper živi uz nju dok ga ne
zatraži druga domena.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Procesor prihvata — pokreće i zaustavlja čitanje | granica prihvata, veličina bloka |
| **A2** | Upravitelj drivera — vlasnik registriranih drivera | registar imenovanih drivera |
| **A3** | Konekcija prema uređaju — zauzet uređaj i djelotvorni parametri | stopa uzorkovanja, središnja frekvencija |
| **A4** | Driver nad tom konekcijom — predmet ovog zahtjeva | blokovi uzoraka i brojač gubitaka |
| **A5** | Helperi pretvorbe — parser i formatter uzoraka | uzorci iz bajtova i natrag |

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | Faktory registrira driver i injektira mu konekciju (`BR-01`) |
| **EV02** | A1 traži prvi blok → driver prelazi u stanje *živ* (`ensure_live`) |
| **EV03** | Uređaj je proizveo blok; međuspremnik ga nudi na preuzimanje |
| **EV04** | Međuspremnik je prekoračen — blok je izgubljen prije preuzimanja (`BR-07`) |
| **EV05** | A1 zaustavlja prihvat (granica iz `BR-08`) ili uređaj nestaje (`BR-09`) |
| **EV06** | A1 traži `write` — samo na uređaju koji odašiljanje prijavljuje i uz razriješenu ovlast (`NFRQ-SEC-07`) |

## 4. Preduvjeti

1. Driver i konekcija registrirani u workflowu (`BR-01`); konekcija je u stanju *povezan*.
2. Veličina bloka je deklarirana i **usklađena sa stopom**: blok mora trajati dulje nego što traje
   njegova obrada, inače je gubitak sustavan, a ne slučajan.
3. Djelotvorni parametri **i ispitane sposobnosti** (izvorni format uzorka, puna skala) očitani su
   pri uspostavi (`BR-05`, `BR-11`) — driver ih **ne** traži ponovno od uređaja i ne pretpostavlja
   ni tražene vrijednosti ni format.

## 5. Normalan tok

1. EV01 — driver dobiva konekciju; vlastitu ne otvara i uređaj ne konfigurira.
2. EV02 — `ensure_live`; driver otvara tok uzoraka nad zauzetim uređajem.
3. EV03 — driver preuzima blok i pridružuje mu **opis**: redni broj, vrijeme prvog uzorka, broj
   uzoraka, djelotvorna stopa i središnja frekvencija (iz A3), izvorni format uzorka **i puna
   skala**. Puna skala je dio opisa jer ista brojčana vrijednost znači različitu razinu na
   8-bitnom i na 14-bitnom uređaju; bez nje blok nije usporediv među uređajima.
4. `read` isporučuje blokove **kao generator** — jedan blok po iteraciji, bez skupljanja u
   memoriju; tok je neograničen dok ga A1 ne zaustavi (`BR-08`).
5. EV04 — driver **broji** izgubljene blokove i uzorke i objavljuje brojač uz svaki sljedeći opis
   (`BR-07`); gubitak ne prekida prihvat sam po sebi.
6. EV05 — `close` zatvara tok; brojači ulaze u završni zapis prolaza.

**Isporuka je blok s opisom, ne goli međuspremnik.** Blok bez djelotvorne stope i vremena nije
interpretabilan ni jednim pipelineom.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| konekcija nije u stanju *povezan* | `read` ne pokušava čitati; kvar imenuje konekciju, ne uređaj |
| prekoračenje međuspremnika (EV04) | brojač raste, prihvat se nastavlja, nalaz ide u zapis (`BR-07`) |
| **trajno** prekoračenje iznad praga | prihvat se zaustavlja kao kvar: podaci su sustavno nepotpuni, a tihi nastavak lažno svjedoči o cjelovitosti |
| uređaj nestaje s magistrale | tok pada; parcijalni rezultat se **označava** parcijalnim (`BR-09`) |
| `write` na uređaju koji ne prijavljuje odašiljanje | odbijeno **tipom** — putanja za pozivatelja ne postoji (`NFRQ-SEC-07` k.1) |
| `write` bez razriješene ovlasti | odbijeno; poruka imenuje **koja** ovlast nedostaje, ne njezin sadržaj (`NFRQ-SEC-07` k.3) |
| `write` dok je konekcija u prijamu | odbijeno — poluduplex, smjer je stanje uređaja (`BR-12`); driver ga ne prebacuje sam |
| zaustavljanje usred bloka | blok se ili isporuči cijel ili odbaci; polublok se ne isporučuje |
| ponovni `close` / `reset` | idempotentno, po ugovoru temeljnog drivera |

## 7. Rezultat

Procesor dobiva niz blokova, svaki s opisom dovoljnim za interpretaciju, i **brojač gubitaka** koji
kaže koliko je toka propušteno. Prihvat bez gubitaka i prihvat s 12 % gubitaka razlikuju se u
podacima, ne samo u zapisu.

## 8. Kriteriji prihvaćanja

Nijedan nije zadovoljen — koda nema.

1. `read` isporučuje blokove kao generator; ništa se ne skuplja u memoriju preko jednog bloka.
2. Svaki blok nosi opis: redni broj, vrijeme, broj uzoraka, djelotvorna stopa, središnja
   frekvencija, format **i puna skala**.
2a. Format i puna skala dolaze iz ispitanog (`BR-11`); nijedan format nije konstanta u kodu i
   nijedna grana ne postoji po modelu uređaja.
3. Djelotvorni parametri dolaze **iz konekcije**; driver ne pita uređaj i ne pretpostavlja tražene
   vrijednosti (`BR-05`).
4. Izgubljeni blokovi i uzorci se broje i objavljuju (`BR-07`); brojač je čitljiv i nakon `close`.
5. Prekoračenje iznad deklariranog praga zaustavlja prihvat kao kvar.
6. `write` postoji samo gdje ga sposobnosti prijavljuju, i ondje traži razriješenu ovlast; inače
   pada tipom (`NFRQ-SEC-07` k.1–k.3).
6a. `read` je **jedna** izvedba za sve uređaje: nijedna grana ne postoji po proizvođaču ni po
   modelu, a pretvorba formata živi u helperu koji tvornica vrati (`BR-13`, `NFRQ-ORG-08` k.2).
6b. `read` i `write` **nisu** spojeni u zajedničku izvedbu unatoč sličnom obliku poziva
   (`NFRQ-ORG-08` k.4).
7. Volumen zapisa: jedinica posla je **prolaz**, ne blok (`NFRQ-OBS-03`) — bez retka po bloku.
8. Modul se uvozi bez driverske knjižnice (test maskiranja, `NFRQ-SEC-03`).

## 9. Verifikacija

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2, 3 | `unittest` nad lažnom konekcijom koja isporučuje unaprijed pripremljene blokove | **nije izvedeno** — nema koda (2026-09-09) |
| 6, 6b | lažne sposobnosti sa i bez smjera odašiljanja; poziv `write` bez ovlasti | nije izvedeno |
| 6a | dva lažna uređaja s **različitim** formatom i brojem stupnjeva kroz **istu** putanju čitanja; mutacija: grana po imenu uređaja mora oboriti test | nije izvedeno |
| 4, 5 | lažna konekcija koja namjerno preskače blokove; provjera brojača i praga | nije izvedeno |
| 7 | brojanje redaka zapisa po prolazu, uz kriterij iz `NFRQ-OBS-03` | nije izvedeno |
| 8 | test maskiranja | nije izvedeno |

Svaka tvrdnja uz **mutacijsku provjeru** ([`DR-WFL-023`](../04-DR/DR-WFL-023-test-framework-is-stdlib-unittest.md)).

> **Slijepa pjega (D-11):** kriterij 5 mjeri se lažnim gubicima. Odgovara li deklarirani prag
> stvarnom ponašanju uređaja pod opterećenjem, ovim se **ne** provjerava.

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-16` §6](../01-HLRQ/HLRQ-16-sdr-capture.md#6-nefunkcionalni-zahtjevi-i-oscal).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-SEC-06` | u zapis ide **opis** bloka (broj, vrijeme, veličina), nikad sadržaj uzoraka |
| `NFRQ-OBS-03` | zapis po bloku je poplava pri milijunima uzoraka u sekundi — sažetak po prolazu i pragovi |
| `NFRQ-SEC-03` | driverska knjižnica je odgođeni uvoz |
| `NFRQ-ORG-02` | ime imenuje ulogu; sámo ime nosi rječnik, ne ovaj zapis (`CLAUDE.md` §3.3). Zapis akronima čeka [`DR-WFL-004`](../04-DR/DR-WFL-004-acronym-identifier-casing.md) |
| `NFRQ-ORG-04` | gubitak uzoraka je svojstvo isporuke, ne novi primitiv |
| OSCAL | driver nasljeđuje podskup konekcije (`HLRQ-16` §6); bez sign-offa deklaracija se ne upisuje |

## 11. Otvoreno

1. **Prag gubitka iz kriterija 5.** Koliko je „previše" i mjeri li se po bloku, po prozoru ili
   kumulativno. `[M]` mjera pod poveljom `NFRQ-DEF-02`; bez nje kriterij nije mjerljiv.
2. **Nosi li driver izvorni format ili normalizira.** Opis ga u oba slučaja nosi (k. 2), ali je li
   isporučeni blok u izvornom formatu uređaja ili sveden na zajednički — otvoreno. Normalizacija
   olakšava pipeline i košta prolaz kroz podatke; izvorni format je jeftin i prebacuje razliku na
   potrošača. Vezano uz `FRQ-DOC-16.4`, koji nije napisan.
3. **Gdje živi ovlast za `write`** — na konekciji (uz smjer, kao stanje) ili na driveru (uz poziv).
   `NFRQ-SEC-07` §5 t.1 traži oblik reference, ne mjesto; mjesto je ovdje otvoreno.
4. **Protutlak** (`HLRQ-16` §7 t.4): odbaciti najstariji blok, najnoviji, ili zaustaviti prihvat.
   Ovaj zapis pretpostavlja **gubitak na strani uređaja uz brojanje**, jer se tok ne da usporiti;
   ako DR odluči drukčije, §5 t.5 i §6 se prepisuju.
