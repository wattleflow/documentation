# FRQ-PTN-18.1 — Manager resursa workflowa

> **Oznaka je provizorna** koliko i razred `HLRQ` iznad nje (`CLAUDE.md` §3.6).

| | |
|---|---|
| **Status** | **Prijedlog — nije provedeno** (2026-09-11) |
| **Odluka** | Nijedan DR nije otvoren. Odluke autora 2026-09-11: vidi [`HLRQ-18`](../01-HLRQ/HLRQ-18-resource-awareness.md) |
| **Nadređeni zahtjev** | [`HLRQ-18`](../01-HLRQ/HLRQ-18-resource-awareness.md) — `BR-01…BR-08`, resursi i signali (§2) |
| **Predmet** | Manager koji tvornica workflowa gradi uz managere konekcija, drivera i procesora: popis radne memorije, pohrane, CPU-a i GPU-a pri pokretanju, praćenje na granicama jedinice posla, pragovi, upozorenje |
| **Dijagram** | [praćenje resursa](FRQ-PTN-18.1-resource-sequence.puml) (sekvencijski) — pogled, ne izvor istine (D-13) |
| **Kaskada** | P-09 · P-10 · D-09 · D-11 |
| **Norme** | `NFRQ-OBS-01` · `NFRQ-OBS-03` · `NFRQ-OBS-04` · `NFRQ-DEF-02` · `NFRQ-SEC-02` · `NFRQ-SEC-03` · `NFRQ-ORG-02` (kvalificirana uloga „manager" je dopuštena) |

## 1. Predmet

Za svaki resurs manager drži **granicu i njezin izvor**, **osnovicu**, **vrh** i **pragove**. Osnovna
izvedba (radna memorija, pohrana, CPU) živi u temeljnom sloju i koristi samo standardnu knjižnicu;
mjerenje GPU-a kroz vanjske knjižnice je proširenje u `blackwattle`, uključeno odgođenim uvozom.

**Izvori** — lokalno provjereno 2026-09-11 (WSL2, Python 3.11.15):

| resurs | izvor | lokalno | napomena |
|---|---|---|---|
| RAM | cgroup `memory.max`, `memory.current`, `memory.events` | ne postoje — nije kontejner | tvrda granica; dosezanjem se poziva OOM ([cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html)) |
| RAM | `sysconf` | 15,6 GiB, slobodno 1,7 GiB | u kontejneru **precjenjuje** raspoloživo |
| RAM | procfs `VmRSS`, `VmHWM` | radi | Linux |
| RAM | `resource`, najveći RSS | radi | **oprez**: u procesu-djetetu može pokazati vrh roditelja |
| CPU | cgroup `cpu.max` („$MAX $PERIOD"; `max` = bez granice) | ne postoji | kvota kontejnera |
| CPU | cgroup `cpu.stat` (`nr_throttled`, `throttled_usec`) | postoji | **prigušivanje je vidljivo usko grlo** |
| CPU | `cpuset.cpus.effective`; afinitet procesa | `0-3`; 4 | CPU-i dodijeljeni procesu |
| CPU | broj CPU-a domaćina; `getloadavg`; `resource` (CPU vrijeme) | 4; 0,22 / 0,21 / 0,26; radi | broj CPU-a domaćina u kontejneru zavarava |
| CPU | broj CPU-a dostupnih procesu | ne postoji u 3.11 | od Pythona 3.13 ([What's New 3.13](https://docs.python.org/3/whatsnew/3.13.html)) |
| pohrana | `shutil.disk_usage` po putanji zapisa | 1007 GiB, slobodno 876 GiB | svaka platforma |
| pohrana | `statvfs`, slobodni inodi | 66 177 995 | Unix; disk se puni i inodima |
| GPU | standardna knjižnica | — | ne vidi GPU |
| GPU | [`nvidia-ml-py`](https://pypi.org/project/nvidia-ml-py/) (NVIDIA, BSD) | nije instaliran | `blackwattle` |
| GPU | [`amdsmi`](https://pypi.org/project/amdsmi/) (AMD) | nije instaliran | `blackwattle`; licenca nije provjerena |
| GPU | `torch` — već ovisnost jezičnih modela | 2.12.0; CUDA nedostupna, 0 uređaja | `blackwattle`; vidi samo ono što vidi torch |

## 2. Akteri

| oznaka | tko / što | unosi u proces |
|---|---|---|
| **A1** | konfiguracija workflowa | pragove po resursu; putanje pohrane koje se prate (iz konfiguracije drivera za pisanje) |
| **A2** | tvornica workflowa | izgradnju managera uz ostale managere |
| **A3** | procesor | granice jedinice posla — dovršen ciklus, flush, kraj prolaza |
| **A4** | korisnik ili operater | prima upozorenje i sažetak |

Platforma nije akter: ništa ne pokreće, samo je izvor mjere.

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | A2 gradi workflow → manager utvrđuje granice |
| **EV02** | A3 počinje prolaz → osnovice |
| **EV03** | A3 dovršava ciklus ili flush → uzorci |
| **EV04** | prag prijeđen ili signal uskog grla vidljiv |
| **EV05** | A3 završava prolaz → sažetak |

## 4. Preduvjeti

1. Pragovi su udjeli granice između 0 i 1, po resursu.
2. Putanje pohrane koje se prate poznate su iz konfiguracije.
3. Proširenje za GPU je instalirano ili se GPU vodi kao nemjeren (`BR-08`).

## 5. Normalan tok

1. **EV01** — za svaki resurs manager utvrđuje granicu iz najužeg izvora i bilježi izvor (`BR-01`).
2. **EV02** — bilježi osnovice: RSS, CPU vrijeme, zauzeće pohrane, memoriju GPU-a ako je proširenje tu.
3. **EV03** — na granici jedinice posla uzima uzorke: udio memorije; prigušivanje CPU-a od prošlog
   uzorka i CPU vrijeme jedinice; slobodan prostor i inode na putanjama zapisa; GPU ako je proširenje tu.
4. **EV04** — prelazak praga ili vidljiv signal uskog grla daje `WARNING` odmah, jednom po prijelazu
   (`BR-04`).
5. **EV05** — jednom po prolazu piše sažetak po resursu: granica i izvor, osnovica, vrh, udio,
   prijelazi pragova. Resursi su u sažetku odvojeni (`BR-05`).

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| granica se ne da utvrditi | resurs se bilježi kao **nepoznate granice**; relativni pragovi se ne primjenjuju i to se prijavljuje jednom |
| proširenje za GPU nije instalirano | GPU je **nemjeren** — nikad slobodan (`BR-08`) |
| platforma bez procfs ili cgroupa | slijepa pjega se deklarira; pohrana se i dalje mjeri |
| prijeđen prag ili vidljivo prigušivanje CPU-a | `WARNING` odmah, s resursom, izvorom granice i mjestom u toku; ponovno tek nakon povratka ispod praga |
| udio memorije ne pada nakon flusha | nalaz o neotpuštenim referencama (`BR-06`, `HLRQ-18` §7 t.4) |
| mjerenje ne uspije | `WARNING`, workflow nastavlja (`BR-07`) |

## 7. Rezultat

Svaki prolaz zna granicu i izvor svakog resursa, ima vrh i udio po jedinici posla i imenovana
upozorenja čim usko grlo postane vidljivo — prije pada, prepunjenog diska ili neobjašnjenog usporenja.

## 8. Kriteriji prihvaćanja

Nijedan nije zadovoljen — koda nema.

1. Granica i izvor zabilježeni su za svaki resurs pri izgradnji; kontejner ima prednost pred domaćinom.
2. Nepoznata granica prijavljuje se kao nepoznata; nijedna vrijednost se ne izmišlja.
3. Nemjeren resurs prijavljuje se kao nemjeren, nikad kao slobodan.
4. Uzorci se uzimaju samo na granicama jedinice posla; broj zapisa razmjeran je broju jedinica.
5. Upozorenje se piše čim je prag prijeđen ili signal uskog grla vidljiv, i jednom po prijelazu.
6. Nijedan prag ne zaustavlja workflow.
7. Resursi su zasebne mjere; ocjene nema.
8. Temeljni dio uvozi samo standardnu knjižnicu; proširenje za GPU uvozi se odgođeno (test maskiranja).
9. Kvar mjerenja ne ruši workflow.
10. Najveći RSS iz `resource` ne koristi se kao vrh procesa-djeteta bez provjere nasljeđivanja.

## 9. Verifikacija

| kriterij | metoda | rezultat |
|---|---|---|
| 1–3 | `unittest` s lažnim izvorima: datoteke cgroupa (`memory.max`, `cpu.max`, `cpu.stat`) u privremenom direktoriju, lažni procfs, lažno zauzeće diska | nije izvedeno |
| 4–6, 9 | lažni procesor koji emitira N ciklusa; uzorci iznad i ispod praga; rastuće prigušivanje | nije izvedeno |
| 8 | test maskiranja proširenja za GPU | nije izvedeno |
| prihvat | primjer za Excel: list po list prema svi odjednom — udio mora slijediti izmjereni odnos iz analize | nije izvedeno |

Uz svaku tvrdnju ide mutacijska provjera (`DR-WFL-023`).

## 10. Nefunkcionalni zahtjevi

Registar: [`HLRQ-18` §6](../01-HLRQ/HLRQ-18-resource-awareness.md#6-nefunkcionalni-zahtjevi).

## 11. Otvoreno

1. ~~Gdje živi kod.~~ **Odlučeno 2026-09-11 (autor):** osnovna izvedba u temeljnom sloju, GPU s vanjskim
   knjižnicama u `blackwattle`.
2. **GPU preko vanjskog alata** kao pod-procesa — `HLRQ-18` §7 t.7.
3. **Promatranje ili poziv** — sluša li manager procesor kroz postojeći mehanizam promatrača, ili ga
   procesor izravno zove na granici jedinice.
4. **Odnos s `FRQ-MET-01`** i s modulom metrika u helperima.
5. **Kanal upozorenja** izvan zapisa (`HLRQ-18` §4, kandidat).
