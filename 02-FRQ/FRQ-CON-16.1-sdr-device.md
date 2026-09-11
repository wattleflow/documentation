# FRQ-CON-16.1 — Konekcija prema SDR uređaju na USB priključku

> **Oznaka je provizorna** koliko i razred `HLRQ` iznad nje (`CLAUDE.md` §3.6). Kategorija `CON`
> je u vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Djelomično provedeno 2026-09-11** — faza 1 (RTL2832U, prijam) u `blackwattle` `connections/sdr/`; ovjereno lažnom obitelji, bez uređaja (§9). Ranije: prijedlog, 2026-09-09 |
| **Odluka** | **Nijedan DR nije otvoren.** Klasa po pristupnom mehanizmu, selektor umjesto skalara i profil modela provjeren ispitivanjem prijedlog su [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §4a; opseg obitelji otvoren (§7 t.1) |
| **Podloga** | [raznolikost SDR uređaja](../06-ANALYSIS/2026-09-09-sdr-vendor-variability.md) (2026-09-09) |
| **Dijagrami** | [klase](FRQ-CON-16.1-class.puml) · [stanje uređaja](FRQ-CON-16.1-device-state.puml) · [sastavljanje](../01-HLRQ/HLRQ-16-assembly-sequence.puml) (sekvencijski) · [dekompozicija](../01-HLRQ/HLRQ-16-sdr-decomposition.puml) — pogledi, ne izvor istine (D-13) |
| **Kaskada** | P-11 (granica oko promjenjive odluke) · P-14 (specifikacija znanja, ne dnevnik) · P-08 (proza nosi uloge, rječnik imena) |
| **Norme** | `NFRQ-ORG-07` (`ALLOWED`) · `NFRQ-ORG-08` (obvezujuće preko `BR-13`) · `NFRQ-SEC-07` (smjer i ovlast) · `NFRQ-SEC-01`, `NFRQ-SEC-06` |
| **Nadređeni zahtjev** | [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) — narativ, `BR-01…BR-10`, svojstva uređaja (§2) |
| **Predmet** | Konekcija prema uređaju — isključivo zauzeće prijamnika, ispitivanje sposobnosti i primjena parametara; `connect` / `disconnect`. **Jedna klasa po mehanizmu, ne po proizvođaču** |
| **Sestrinski** | [`FRQ-DRV-16.2`](FRQ-DRV-16.2-iq-stream.md) (dohvat uzoraka) · [`FRQ-PRC-16.3`](FRQ-PRC-16.3-sdr-capture.md) (vođenje prolaza) |
| **Izvedba** | — (predložena putanja: `connections/sdr.py`) |

## 1. Predmet

Prijamnik je **jedan pod-sustav** kojem konekcija daje pristup. Za razliku od mrežnih konekcija,
zauzeće je isključivo i stanje *povezan* znači da uređaj **nitko drugi ne može dobiti**
([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) §2). Konekcija drži uređaj i njegove parametre;
uzorci su driverovi (`FRQ-DRV-16.2`).

Klasa je **po pristupnom mehanizmu**, ne po proizvođaču ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
§4a): razlika među modelima iscrpljuje se u vrijednostima koje nosi **profil modela** u konfiguraciji,
a konekcija ga provjerava ispitivanjem (`BR-11`) — ne u konstantama u kodu.

### Što mora biti u konfiguraciji

Konfiguracija ima **dvije razine**, jer se mijenjaju iz različitih razloga: profil kad se promijeni
model, instanca kad se promijeni zadatak.

| razina | ključ | što nosi | pravilo |
|---|---|---|---|
| **profil modela** — jedan po modelu, dijeli se | `model` | identitet kako ga mehanizam prepoznaje (vrsta tunera, oznaka ploče, zapis proizvođača) | `BR-11` |
| | `family` | obitelj, tj. knjižnica domaćina | `BR-13` |
| | `requires` | najmanja inačica mehanizma koja model podržava | `BR-14` |
| | `freq_ranges` | frekvencijski doseg kao **lista** raspona | `BR-11` |
| | `sample_rates` | dopuštene stope kao lista raspona, uz najveću stopu bez gubitaka | `BR-07`, `BR-11` |
| | `gain_stages` | imenovani stupnjevi i njihovi rasponi ili tablice | `BR-04` |
| | `formats` | formati uzorka s punom skalom i uz koju stopu ili način vrijede | `BR-13` |
| | `directions`, `duplex` | prijam / odašiljanje; poluduplex ili puni | `BR-12` |
| | `ports`, `features` | ulazi; bias-tee, HF put, vanjski takt | `BR-11` |
| **instanca** — jedna po jedinici i zadatku | `device` | selektor, skup ključ-vrijednost koji razrješava točno jedan uređaj | `BR-02` |
| | `profile` | referenca na profil modela | `BR-11` |
| | `sample_rate`, `center_freq` | tražena točka i širina prihvata | `BR-04` |
| | `gain_mode`, `gain` | način (`auto`/`manual`) i karta stupnjeva **imenovanih u profilu** | `BR-04` |
| | `ppm`, `antenna`, `bias_tee` | korekcija takta, ulaz, napajanje antene | `BR-04` |
| | `direction`, `authorisation` | smjer; kod odašiljanja i referenca ovlasti | `BR-12`, `NFRQ-SEC-07` |
| | `device_timeout` | koliko se čeka zauzet uređaj | `BR-03` |

Tri pravila drže razine razdvojenima:

1. Instanca **ne nadjačava** profil: vrijednost izvan profila je kvar konfiguracije.
2. Profil i ispitano **moraju se slagati** gdje mehanizam nešto zna reći; neslaganje je kvar
   uspostave, ne tiho odabrana strana (`BR-11`).
3. Instanca ne nosi sposobnosti, profil ne nosi zadatak.

Oblik — vrijednosti su ilustracija, a profil od zapisa traži izvor za svaku stavku
([analiza tržišta](../06-ANALYSIS/2026-09-11-sdr-device-market.md)):

```yaml
profile:                                  # jedan zapis po modelu
  model:       {tuner: R820T}
  family:      rtlsdr
  freq_ranges: [[24.0e6, 1766.0e6]]       # R820T2 prema proizvođaču
  gain_stages: {TUNER: [0.0, 0.9, 1.4, …, 49.6]}   # 29 vrijednosti, izmjereno 2026-09-09
  formats:     [{format: u8_iq}]          # 8 bit
  directions:  [rx]

instance:                                 # jedan zapis po jedinici i zadatku
  device:      {family: rtlsdr, serial: "82895453"}
  profile:     r820t
  sample_rate: 2.4e6
  center_freq: 137.9125e6
  gain_mode:   manual
  gain:        {TUNER: 28.0}
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
4. A4 zauzima uređaj, prepoznaje model i učitava njegov **profil**; provjerava da ugrađena inačica
   mehanizma podržava taj model (`BR-14`).
5. A4 **ispituje** ono što mehanizam zna reći (npr. vrstu tunera i tablicu pojačanja) i provjerava da
   se slaže s profilom; zatim provjerava instancu protiv profila. Neslaganje je kvar, s obje strane u
   poruci (`BR-11`).
6. A4 primjenjuje parametre redom: stopa uzorkovanja → središnja frekvencija → pojačanje → `ppm`.
7. A4 **čita natrag djelotvorne vrijednosti** i objavljuje ih u zapis (`BR-05`); razlika između
   tražene i postignute vrijednosti je podatak o prihvatu, ne šum. Izmjereno: R820T prihvaća 29
   diskretnih vrijednosti pojačanja, pa tražena vrijednost redovito **nije** postignuta.
8. A4 prelazi u stanje *povezan* u traženom **smjeru**; prijam i odašiljanje se isključuju, pa je
   prijelaz između njih izričita radnja, ne posljedica poziva (`BR-12`,
   [dijagram stanja](FRQ-CON-16.1-device-state.puml)).
9. A3 vraća konekciju, driver je smije koristiti (EV03).
10. EV04 — `disconnect` oslobađa uređaj i vraća ga na raspolaganje drugom procesu.

**Normalan tok je postignut kad je uređaj zauzet, profil potvrđen i djelotvorni parametri
očitani.** Nijedan uzorak u ovom toku nije pročitan — prvo čitanje radi driver.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| konekcija nije registrirana ili joj je konfiguracija nevaljana | workflow **ne započinje** (`BR-01`, `BR-06`) |
| selektor ne pogađa nijedan uređaj | razred *dosežnost*, sa selektorom i popisom prisutnih uređaja u poruci |
| selektor pogađa više uređaja | razred *dvoznačnost* — nikad se ne uzima prvi (`BR-02`) |
| konfiguracija imenuje stupanj ili ulaz koji uređaj nema | kvar konfiguracije **pri uspostavi**, s popisom onoga što uređaj nudi (`BR-11`) |
| ispitano se ne slaže s profilom | kvar uspostave; poruka imenuje obje strane — profil i ono što je mehanizam prijavio — i nikad ne bira tiho (`BR-11`) |
| mehanizam ne podržava deklarirani model (npr. RTL-SDR V4 sa standardnim driverom) | **glasan** kvar uspostave prije prvog bloka (`BR-14`) — rad na krivoj frekvenciji nije dopušten ishod |
| frekvencija ili plan ugađanja izvan raspona profila | kvar konfiguracije **pri učitavanju**; workflow ne započinje (`BR-15`) |
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

Popis je bio ulaz u izvedbu i ostaje mjerilo; stanje 2026-09-11 vodi §9.

1. Zadovoljen ugovor konekcije: `connect` / `disconnect` + stanje; nijedna operacija nad uzorcima
   nije konekcijina.
2. `ALLOWED` sadrži točno `device`, `sample_rate`, `center_freq`, `gain_mode`, `gain`, `antenna`,
   `ppm`, `device_timeout` uz naslijeđene ključeve — ništa skriveno (`NFRQ-ORG-07`).
3. Selektor razrješava točno jedan uređaj; nula i više od jednog daju **različite** razrede kvara
   (`BR-02`).
3a. Pojačanje se konfigurira kao karta imenovanih stupnjeva; uređaj s jednim stupnjem ne traži
   drukčiji oblik ni drugu klasu (`BR-04`).
3b. Konfiguracija ima dvije razine; profil je potvrđen ispitivanjem gdje mehanizam to omogućuje, a
   instanca je provjerena protiv profila; nijedan raspon ni
   popis stupnjeva nije konstanta u kodu (`BR-11`).
3c. Odašiljanje se ne otključava logičkim ključem: traži prijavljenu sposobnost uređaja **i**
   razriješenu referencu ovlasti, a izostanak bilo čega od toga znači odbijanje (`NFRQ-SEC-07`
   k.1–k.3).
3d. Prijam i odašiljanje su **stanja koja se isključuju**; konekcija ne prelazi između njih kao
   posljedicu poziva drivera (`BR-12`).
3e. Mehanizam koji ne podržava deklarirani model odbijen je prije prvog bloka (`BR-14`).
3f. Frekvencija ili plan ugađanja izvan raspona profila odbijeni su pri učitavanju konfiguracije
   (`BR-15`).
4. Djelotvorni parametri očitani s uređaja i dostupni driveru kao svojstva (`BR-05`).
5. Zauzet uređaj daje razred *zauzetost* nakon `device_timeout`, ne blokira neodređeno (`BR-03`).
6. Tri razreda kvara — *dosežnost*, *zauzetost*, *prava* — razlikuju se u tipu, ne samo u poruci.
7. `connect` i `disconnect` idempotentni; `disconnect` oslobađa uređaj i kad prihvat pada.
8. Modul se uvozi i kad driverska knjižnica nije instalirana (test maskiranja, `NFRQ-SEC-03`).

## 9. Verifikacija

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 4 | `unittest` nad lažnom obitelji ([`DR-WFL-023`](../04-DR/DR-WFL-023-test-framework-is-stdlib-unittest.md)) | ✅ djelotvorne vrijednosti očitane s uređaja; konekcija nema operaciju nad uzorcima — sirove bajtove vuče driver kroz obiteljski helper |
| 2 | pregled deklaracije ključeva | ⚠️ **razilaženje:** deklaracija nosi i ključeve iz §1 (`profile`, `bias_tee`, `direction`, `authorisation`) te `tuning_plan` (odluka 2026-09-11); kriterij 2 ih ne navodi i treba ga uskladiti |
| 3, 3a, 3b | selektor s nula i s dva pogotka; pojačanje kao skalar; model i tablica pojačanja protiv profila | ✅ uz mutaciju m3; uređaj s više stupnjeva **nije** ispitan — lažna obitelj ima jedan |
| 3c, 3d | smjer odašiljanja | ✅ odbijen tipom kvara konfiguracije jer nijedna obitelj ne odašilje; ovlast i isključivost smjerova čekaju fazu 2 |
| 3e, 3f | nepodržan model; prenizak `requires`; frekvencija i plan izvan raspona pri učitavanju | ✅ odbijena konfiguracija ne zauzima uređaj |
| 5, 6 | zauzeto do i nakon `device_timeout`; bez prava; bez pogotka | ✅ na lažnoj obitelji; zauzetost je 2026-09-09 potvrđena i na stroju (`usb_claim_interface error -6`) |
| 7 | dvostruko zatvaranje; oslobađanje nakon kvara provjere | ✅ uz mutaciju m2 |
| 8 | test maskiranja u podprocesu | ✅ uz mutaciju m8 — eager uvoz knjižnice obara test |

> **Trojka (D-10):** `unittest`, Python 3.11.15, `blackwattle/tests/sdr/` — 28 testova · kriterij §8 ·
> WSL2 6.18.33.2, 2026-09-11. Mutacijske provjere m1–m8 (`DR-WFL-023`) sve obaraju testove.

Uz svaku tvrdnju ide **mutacijska provjera** (`DR-WFL-023`): namjerna izmjena koja test mora
oboriti. Test koji prolazi i s pokvarenom izvedbom nije svjedočanstvo (D-05).

> **Slijepa pjega (D-11):** obiteljski helper za RTL2832U **nije izveden ni jednom** — `pyrtlsdr` nije bio
> instaliran u okruženju testova, a uređaj nije bio prikopčan; ovjeren je čitanjem koda, ne izvođenjem.
> Kriteriji 5, 6 i 7 traže **fizički uređaj** ili njegovu vjernu zamjenu.
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
1a. **Gdje žive profili i gdje se provjeravaju** — profili u paketu ili u korisničkoj konfiguraciji;
   provjera u konekciji pri `connect` (kako ovaj zapis pretpostavlja)
   ili u driveru pri prvom čitanju.
2. **Čeka li se zauzet uređaj uopće.** `device_timeout` pretpostavlja da čekanje ima smisla; za
   isključivi resurs možda nema — tada ključ nestaje, a `BR-03` postaje trenutan kvar.
3. **Tko vodi životni ciklus** — konekcija ili procesor ([`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md)
   §7 t.3). Ovaj zapis pretpostavlja **konekciju**, po ugovoru iz `HLRQ-13` §4; ako DR odluči
   drukčije, §5 i §6 se prepisuju.
