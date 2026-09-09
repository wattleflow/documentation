# FRQ-CON-16.1 — Konekcija prema SDR uređaju na USB priključku

> **Oznaka je provizorna** koliko i razred `HLRQ` iznad nje (`CLAUDE.md` §3.6). Kategorija `CON`
> je u vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Prijedlog — nije provedeno** (provjereno 2026-09-09; u `blackwattle/src/` nema uređajne komponente) |
| **Odluka** | [`DR-PRC-003`](../04-DR/DR-PRC-003-sdr-access-layer.md) — **prijedlog**: klasa po pristupnom mehanizmu, selektor umjesto skalara, ispitivanje sposobnosti. Izbor mehanizma i dalje otvoren ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §7 t.1) |
| **Podloga** | [raznolikost SDR uređaja](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md) (2026-09-09) |
| **Dijagrami** | [stanje uređaja](FRQ-CON-16.1-device-state.puml) (stanja) · [dekompozicija](../01-HLRQ/HLRQ-16-sdr-decomposition.puml) (klasni) — pogledi, ne izvor istine (D-13) |
| **Kaskada** | P-11 (granica oko promjenjive odluke) · P-14 (specifikacija znanja, ne dnevnik) · P-08 (proza nosi uloge, rječnik imena) |
| **Norme** | `NFRQ-ORG-07` (`ALLOWED`) · `NFRQ-ORG-08` (obvezujuće preko `BR-13`) · `NFRQ-SEC-07` (smjer i ovlast) · `NFRQ-SEC-01`, `NFRQ-SEC-06` |
| **Nadređeni zahtjev** | [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) — narativ, `BR-01…BR-10`, svojstva uređaja (§2) |
| **Predmet** | `ConnectionSdrDevice(GenericConnection)` — isključivo zauzeće prijamnika, ispitivanje sposobnosti i primjena parametara; `connect` / `disconnect`. **Jedna klasa po mehanizmu, ne po proizvođaču** |
| **Sestrinski** | [`FRQ-DRV-16.2`](FRQ-DRV-16.2-iq-stream.md) (dohvat uzoraka) · [`FRQ-PRC-16.3`](FRQ-PRC-16.3-sdr-capture.md) (vođenje prolaza) |
| **Izvedba** | — (predložena putanja: `connections/sdr.py`) |

## 1. Predmet

Prijamnik je **jedan pod-sustav** kojem konekcija daje pristup. Za razliku od mrežnih konekcija,
zauzeće je isključivo i stanje *povezan* znači da uređaj **nitko drugi ne može dobiti**
([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §2). Konekcija drži uređaj i njegove parametre;
uzorci su driverovi (`FRQ-DRV-16.2`).

Klasa je **po pristupnom mehanizmu**, ne po proizvođaču ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
§4a): razlika među modelima uređaja iscrpljuje se u imenima i rasponima, a to su vrijednosti koje
konekcija **ispituje** (`BR-11`), ne konstante koje nosi u kodu.

| dolazi iz temeljne konekcije | dodaje ova specijalizacija |
|---|---|
| stanje pristupa i njegovi prijelazi | `device` — **selektor**, skup ključ-vrijednost (`BR-02`) |
| `connect` / `disconnect` / `create_connection` kao ugovor | `sample_rate`, `center_freq` — točka i širina prihvata |
| `context()`, `hot_swap`, `reset` | `gain_mode` (`auto`/`manual`) i `gain` — **karta imenovanih stupnjeva** (`BR-04`) |
| `ALLOWED` kao deklarirana ulazna površina (`NFRQ-ORG-07`) | `antenna`, `ppm`, `device_timeout` |
| — | `direction` — traženi smjer; poluduplex ga čini **stanjem**, ne parametrom poziva (`BR-12`) |

Oblik konfiguracije koji iz toga slijedi — uređaj s jednim stupnjem i uređaj s tri razlikuju se
**samo u broju unosa karte**, ne u obliku ni u klasi:

```yaml
device:    {driver: rtlsdr, serial: "82895453"}   # selektor; oblik pripada mehanizmu
gain_mode: manual
gain:      {TUNER: 28.0}                          # troetapni uređaj: {LNA: 24, VGA: 20, AMP: 0}
```

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Konfiguracija (YAML) — registrira konekciju imenom i tipom | `type`, `name`, `configuration` |
| **A2** | `Workflow` / `Processor` — pokreće zahtjev za pristupom | ime registrirane konekcije |
| **A3** | Upravitelj konekcija — vlasnik registriranih konekcija | registar imenovanih konekcija |
| **A4** | Konekcija prema uređaju — predmet ovog zahtjeva | zauzet uređaj i djelotvorni parametri |

Uređaj i USB podsustav **nisu akteri** — ništa ne pokreću; sudionici su toka i izvori kvara.

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | Faktory gradi workflow → registrira konekciju iz konfiguracije (`BR-01`) |
| **EV02** | A2 traži pristup od upravitelja konekcija prije prvog bloka (`BR-06`) |
| **EV03** | Driver traži zauzet uređaj i djelotvorne parametre prije čitanja |
| **EV04** | Workflow završava ili se ruši → oslobađanje uređaja |
| **EV05** | Uređaj nestaje s magistrale tijekom prihvata (`BR-09`) |

## 4. Preduvjeti

1. Konekcija je registrirana s valjanom konfiguracijom (`BR-01`).
2. `device` selektor je zadan i razrješava **točno jedan** uređaj; goli redni indeks se ne
   prihvaća kao adresa (`BR-02`).
3. Korisnik pod kojim proces radi ima prava nad uređajnim čvorom (udev pravilo); prava su
   preduvjet okoline, ne nešto što konekcija podiže.
4. Parametri tunera su deklarirani ili imaju deklarirano zadano; nezadana središnja frekvencija
   nije upotrebljiva pretpostavka.
5. Imena stupnjeva pojačanja i ulaza **ne** provjeravaju se pri gradnji workflowa — provjeravaju se
   pri uspostavi, protiv onoga što uređaj prijavi (`BR-11`). Cijena je kasniji kvar; dobitak je da
   se o odsutnom uređaju ne tvrdi ništa.
6. `direction: transmit` traži uređaj koji odašiljanje prijavljuje **i** razriješenu ovlast; bez
   oboje konekcija ne ulazi u to stanje (`NFRQ-SEC-07` k.2, k.3).

## 5. Normalan tok

1. EV01 — faktory instancira konekciju i registrira je pod zadanim imenom.
2. EV02 — A2 traži pristup od A3 po imenu.
3. A4 **nabraja** prisutne uređaje i primjenjuje selektor. Nula pogodaka je *dosežnost*, više od
   jedne *dvoznačnost*; ni u jednom slučaju se ne pogađa (`BR-02`). Nabrajanje prethodi zauzimanju
   i uspijeva i kad uređaj drži drugi proces, pa poruka može imenovati **koji** je uređaj zauzet.
4. A4 zauzima uređaj i **ispituje njegove sposobnosti**: stupnjeve pojačanja i njihove raspone,
   dopuštene stope (kao **skup raspona**, ne min–max), frekvencijski doseg, ulaze, izvorni format
   uzorka i punu skalu (`BR-11`).
5. A4 provjerava konfiguraciju **protiv ispitanog**: ime stupnja ili ulaza koje uređaj ne
   prijavljuje je kvar konfiguracije, s popisom onoga što uređaj nudi.
6. A4 primjenjuje parametre redom: stopa uzorkovanja → središnja frekvencija → pojačanje → `ppm`.
7. A4 **čita natrag djelotvorne vrijednosti** i objavljuje ih u zapis (`BR-05`); razlika između
   tražene i postignute vrijednosti je podatak o prihvatu, ne šum. Izmjereno: R820T prihvaća 29
   diskretnih vrijednosti pojačanja, pa tražena vrijednost redovito **nije** postignuta.
8. A4 prelazi u stanje *povezan* u traženom **smjeru**; prijam i odašiljanje se isključuju, pa je
   prijelaz između njih izričita radnja, ne posljedica poziva (`BR-12`,
   [dijagram stanja](FRQ-CON-16.1-device-state.puml)).
9. A3 vraća konekciju, driver je smije koristiti (EV03).
10. EV04 — `disconnect` oslobađa uređaj i vraća ga na raspolaganje drugom procesu.

**Normalan tok je postignut kad je uređaj zauzet, sposobnosti ispitane i djelotvorni parametri
očitani.** Nijedan uzorak u ovom toku nije pročitan — prvo čitanje radi driver.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| konekcija nije registrirana ili joj je konfiguracija nevaljana | workflow **ne započinje** (`BR-01`, `BR-06`) |
| selektor ne pogađa nijedan uređaj | razred *dosežnost*, sa selektorom i popisom prisutnih uređaja u poruci |
| selektor pogađa više uređaja | razred *dvoznačnost* — nikad se ne uzima prvi (`BR-02`) |
| konfiguracija imenuje stupanj ili ulaz koji uređaj nema | kvar konfiguracije **pri uspostavi**, s popisom onoga što uređaj nudi (`BR-11`) |
| ispitivanje vrati nepotpun popis sposobnosti | valjana konfiguracija biva odbijena; poruka mora imenovati **izvor** popisa, da se šutljiv mehanizam razlikuje od krive konfiguracije |
| uređaj drži drugi proces | razred *zauzetost*; čeka se najviše `device_timeout`, pa kvar (`BR-03`) — ne čeka se neodređeno |
| nema prava nad uređajnim čvorom | razred *prava*, razlikovan od nepostojanja uređaja; poruka imenuje čvor, ne rješenje |
| uređaj odbija ili zaokružuje parametar | **nije kvar**: djelotvorna vrijednost se očitava i zapisuje (`BR-05`). Kvar je tek kad je razlika izvan deklarirane tolerancije |
| zadan goli redni indeks umjesto selektora | kvar konfiguracije pri gradnji, ne pri prvom čitanju |
| selektor pogađa **transceiver** | prihvaćeno kao svaki drugi uređaj; prijamni ugovor je isti. Odašiljanje traži izričit smjer i ovlast (`BR-12`, `NFRQ-SEC-07`) |
| tražen smjer koji uređaj ne prijavljuje | kvar konfiguracije pri uspostavi, uz popis smjerova koje uređaj nudi |
| tražen prijelaz prijam ↔ odašiljanje | dopušten samo kao izričita radnja iz mirovanja; nikad usporedno (poluduplex, `BR-12`) |
| EV05 — uređaj nestaje s magistrale | stanje prelazi u *prekinut*; prihvat je kvar, a ne uredan kraj (`BR-09`) |
| ponovni `connect` / `disconnect` nad istim stanjem | idempotentno ([`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) §4 t.2) |

## 7. Rezultat

Uređaj je zauzet, parametri primijenjeni i **očitani**, pristup imenovan. Svaki driver u workflowu
koristi **istu** konekciju, dakle isti uređaj i iste parametre — dvije obrade nad istim prihvatom
ne mogu se razići u pretpostavci o stopi uzorkovanja. Nedostupan ili zauzet uređaj zaustavlja
workflow prije prvog bloka (`BR-06`).

## 8. Kriteriji prihvaćanja

Nijedan nije zadovoljen — koda nema. Popis je **ulaz u izvedbu**, ne izvještaj.

1. Zadovoljen ugovor konekcije: `connect` / `disconnect` + stanje; nijedna operacija nad uzorcima
   nije konekcijina.
2. `ALLOWED` sadrži točno `device`, `sample_rate`, `center_freq`, `gain_mode`, `gain`, `antenna`,
   `ppm`, `device_timeout` uz naslijeđene ključeve — ništa skriveno (`NFRQ-ORG-07`).
3. Selektor razrješava točno jedan uređaj; nula i više od jednog daju **različite** razrede kvara
   (`BR-02`).
3a. Pojačanje se konfigurira kao karta imenovanih stupnjeva; uređaj s jednim stupnjem ne traži
   drukčiji oblik ni drugu klasu (`BR-04`).
3b. Sposobnosti su ispitane s uređaja i konfiguracija je provjerena protiv njih; nijedan raspon ni
   popis stupnjeva nije konstanta u kodu (`BR-11`).
3c. Odašiljanje se ne otključava logičkim ključem: traži prijavljenu sposobnost uređaja **i**
   razriješenu referencu ovlasti, a izostanak bilo čega od toga znači odbijanje (`NFRQ-SEC-07`
   k.1–k.3).
3d. Prijam i odašiljanje su **stanja koja se isključuju**; konekcija ne prelazi između njih kao
   posljedicu poziva drivera (`BR-12`).
4. Djelotvorni parametri očitani s uređaja i dostupni driveru kao svojstva (`BR-05`).
5. Zauzet uređaj daje razred *zauzetost* nakon `device_timeout`, ne blokira neodređeno (`BR-03`).
6. Tri razreda kvara — *dosežnost*, *zauzetost*, *prava* — razlikuju se u tipu, ne samo u poruci.
7. `connect` i `disconnect` idempotentni; `disconnect` oslobađa uređaj i kad prihvat pada.
8. Modul se uvozi i kad driverska knjižnica nije instalirana (test maskiranja, `NFRQ-SEC-03`).

## 9. Verifikacija

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2, 4 | `unittest` nad instancom s lažnim uređajem ([`DR-WFL-023`](../04-DR/DR-WFL-023-test-framework-is-stdlib-unittest.md)) | **nije izvedeno** — nema koda (2026-09-09) |
| 3, 3a, 3b | lažni mehanizam koji prijavljuje jedan i tri stupnja pojačanja te razmaknute raspone stopa; selektor s nula i s dva pogotka | nije izvedeno |
| 3c, 3d | lažni transceiver koji prijavljuje odašiljanje: sa i bez ovlasti, te pokušaj usporednog prijama i odašiljanja | nije izvedeno |
| 5, 6 | negativni slučajevi: uređaj zauzet drugim procesom, čvor bez prava | **djelomično izvedeno** — zauzetost potvrđena na stroju 2026-09-09 (`usb_claim_interface error -6` uz uspješno nabrajanje); ostalo nije |
| 7 | dvostruki `connect`/`disconnect`, te `disconnect` iz `except` grane | nije izvedeno |
| 8 | test maskiranja driverske knjižnice | nije izvedeno |

Uz svaku tvrdnju ide **mutacijska provjera** (`DR-WFL-023`): namjerna izmjena koja test mora
oboriti. Test koji prolazi i s pokvarenom izvedbom nije svjedočanstvo (D-05).

> **Slijepa pjega (D-11):** kriteriji 5, 6 i 7 traže **fizički uređaj** ili njegovu vjernu zamjenu.
> Lažni uređaj dokazuje ugovor, ne ponašanje USB sloja. Uz to je sve izmjereno 2026-09-09 mjereno
> na **jednom** modelu (RTL2832U + R820T); ponašanje uređaja s više stupnjeva pojačanja i širim
> formatom uzorka **nije** provjereno ni na jednom primjerku.

## 10. Nefunkcionalni zahtjevi

Registar i OSCAL obveza: [`HLRQ-16` §6](../01-HLRQ/HLRQ-16-sdr-capture.md#6-nefunkcionalni-zahtjevi-i-oscal).

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-SEC-01` | jedna konekcija po uređaju; stanje *povezan* je zauzeće fizičkog resursa, pa je i doseg fizički |
| `NFRQ-SEC-02` | parametri su konfiguracija u `ALLOWED`, ne nove javne metode |
| `NFRQ-SEC-03` | driverska knjižnica je odgođeni uvoz; binarna sistemska ovisnost ostaje nepokrivena rupa |
| `NFRQ-SEC-06` | u zapis idu serijski broj, frekvencija i stopa — **ne** uzorci ni izvedeni sadržaj |
| `NFRQ-ORG-02` | ime imenuje ulogu i pod-sustav, ne generičku imenicu; sámo ime nosi rječnik, ne ovaj zapis (`CLAUDE.md` §3.3). Zapis akronima čeka [`DR-WFL-004`](../04-DR/DR-WFL-004-acronym-identifier-casing.md) |
| OSCAL | prvi slučaj **podskupa** kontrola — `HLRQ-16` §6; deklaracija se ne upisuje bez sign-offa |

## 11. Otvoreno

1. **Tolerancija za `BR-05`.** Koliko smije biti razmaka između tražene i postignute frekvencije i
   stope prije nego što je to kvar. Bez broja kriterij 4 nije mjerljiv, a broj je `[M]` mjera pod
   poveljom `NFRQ-DEF-02`. Mjerenje pokazuje da tolerancija **mora** postojati: tablica pojačanja
   R820T ima korake do 3,1 dB, pa je traženo rijetko postignuto.
1b. **Gdje živi ovlast za odašiljanje** — na konekciji uz smjer, ili na driveru uz poziv
   (`FRQ-DRV-16.2` §11 t.3). Jedno mjesto, ne oba (D-12).
1a. **Gdje živi ispitivanje sposobnosti** — u konekciji pri `connect` (kako ovaj zapis pretpostavlja)
   ili u driveru pri prvom čitanju ([`DR-PRC-003`](../04-DR/DR-PRC-003-sdr-access-layer.md) §Cijena).
2. **Čeka li se zauzet uređaj uopće.** `device_timeout` pretpostavlja da čekanje ima smisla; za
   isključivi resurs možda nema — tada ključ nestaje, a `BR-03` postaje trenutan kvar.
3. **Tko vodi životni ciklus** — konekcija ili procesor ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
   §7 t.3). Ovaj zapis pretpostavlja **konekciju**, po ugovoru iz `HLRQ-13` §4; ako DR odluči
   drukčije, §5 i §6 se prepisuju.
