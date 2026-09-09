# FRQ-MET-01 — Prikupljanje i usmjeravanje metrike

> **Oznaka je provizorna**, kao `AUD` i `MAIL`: `MET` je sposobnost, ne ontološki primitiv, i
> nema nadređeni HLRQ. Broj ne referira roditelja. Ulazak u vokabular ide kroz DR (D-12).

| | |
|---|---|
| **Status** | **Prijedlog. Nije provedeno.** Izvedivost mjerena prototipom izvan `src/` (§10) |
| **Norme** | [`NFRQ-OBS-04`](../03-NFRQ/NFRQ-OBS-04-metric-admissibility-EN.md) · [`NFRQ-DEF-02`](../03-NFRQ/NFRQ-DEF-02-measurement-charter-EN.md) · [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume-EN.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) |
| **Sestrinski** | [`FRQ-PRC-15.22`](FRQ-PRC-15.22-document-flow.md) — tok iz kojeg mjerenje nastaje |
| **Predmet** | Kolektor koji iz postojećih audit parova izvodi mjere, ispravlja ih, i usmjerava u nula ili više odredišta |
| **Standard** | Prometheus exposition format 0.0.4; PromQL kao jezik upita |
| **Dijagrami** | Građa: [dekompozicija](FRQ-MET-01-metric-structure.puml) · Integracija: [priključak](FRQ-MET-01-metric-attachment.puml). Pogledi s deklariranim gledištem, ne izvor istine (D-13) |

## 1. Opseg

Sustav mora moći odgovoriti **koliko je posla obavljeno, koliko je trajalo i koliko ga je
propalo**, po sloju i po vrsti predmeta — bez da ijedna komponenta zna za monitoring.

Temelj već postoji i **nije ga potrebno graditi**: svaka metoda koja piše
`step=Started` i `step=Completed` time omeđuje trajanje, jer svaki zapis nosi `record.created`.
Završni zapis uz to redovito nosi i količinu (`size`, `chars`, `pages`). Mjerenje je dakle
**izvedeno iz zatečenog**, ne dodano u vruću putanju.

**Izvan opsega:** prikaz (Grafana ploče), pravila uzbune (`NFRQ-SEC-04`), i sama razina
logiranja — `DEBUG` je dijagnostička kategorija koja se u produkciji ne koristi, pa metrika o njoj
ne smije ovisiti (§6, `EV05`).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | komponenta koja radi posao (blackboard, repozitorij, procesor, driver, strategija) | par `Started`/`Completed`\|`Failed` |
| **A2** | **kolektor** — `logging.Handler` koji upari, **ispravi** i akumulira | zapisi |
| **A3** | **metrički repozitorij** — jedan po odredištu, po uzoru na `RepositoryWithDriver` | strategija upisa + driver |
| **A4** | `DriverPrometheus` / `DriverGrafana` — prijenos | exposition tekst / anotacija |
| **A5** | `Workflow` — vlasnik prolaza; on zatvara mjerenje i traži izvještaj | granica prolaza |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 emitira `step=Started` → kolektor otvara raspon |
| **EV02** | A1 emitira `step=Completed` → raspon se zatvara ishodom *completed* |
| **EV03** | A1 emitira `step=Failed` → raspon se zatvara ishodom **failed** |
| **EV04** | raspon nadživi `ttl` → **istekne**, broji se kao neizmjeren |
| **EV05** | razina komponente potisne zapis → raspon **ne nastaje** |
| **EV06** | A5 zatvara prolaz → kolektor slaže uzorke i predaje ih repozitorijima |
| **EV07** | A3 nema konfiguriranog drivera → mjere ostaju u memoriji i odbacuju se |

## 4. Preduvjeti

1. Metoda koja otvori raspon zatvara ga **točno jednom** (`NFRQ-OBS-04` k.6). Danas ne vrijedi —
   §12 t.1.
2. Kolektor je pretplaćen prije prvog posla; `Audit.subscribe_handler()` je postojeći priključak.
3. Odredište je neobavezno: bez repozitorija sustav i dalje mjeri, samo nikamo ne šalje.

## 5. Normalan tok

1. A1 radi svoj posao i piše par, kao i danas. **Ne mjeri, ne broji, ne zna za A2.**
2. A2 upari zapise po `(logger, funcName, event)` — `record.funcName` postoji, pa se raspon veže
   uz metodu, ne samo uz klasu.
3. A2 **ispravlja**, i to je razlog zašto je ispravljanje na jednom mjestu:
   - `Failed` zatvara raspon — neuspjeh je ishod, ne nestanak;
   - zaostali raspon ističe umjesto da visi;
   - količina se čita iz završnog zapisa, kojim god poznatim ključem bila izražena;
   - protok se **izvodi** iz količine i trajanja, nikad ne prima kao podatak.
4. Na granici prolaza (EV06) A2 slaže uzorke: brojači, histogrami, stanja (§8).
5. A5 predaje uzorke **svakom** registriranom metričkom repozitoriju — *N* odredišta = *N*
   strategija = *N* drivera, isti obrazac koji `FRQ-PRC-15.22` opisuje za dokumente.
6. A3 oblikuje uzorak za svoje odredište i predaje ga driveru; driver ga otprema.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| par ostaje otvoren | istek nakon `ttl`, uvećava `wf_open_spans` | gap je **objavljen**, ne prešućen (`NFRQ-OBS-04` k.8) |
| dva `Started` u istoj metodi | drugi se odbacuje, uvećava brojač nalaza | mjerenje ne laže; kod je nalaz (§12 t.1) |
| zapis potisnut razinom | raspon ne nastaje | zato metrika ide **vlastitim kanalom**, ne dijeli razinu s dijagnostikom |
| odredište nedostupno | `WARNING`, uzorci se odbacuju | prolaz ne pada zbog monitoringa |
| jedno odredište padne | ostala se i dalje obilaze | za razliku od dokumentnog flusha (`FRQ-PRC-15.22` §11 t.2) |

## 7. Dekompozicija

Građu nosi [dijagram razreda](FRQ-MET-01-metric-structure.puml), a priključak
[drugi](FRQ-MET-01-metric-attachment.puml). Podjela u jednoj tablici:

| razred | uloga | distribucija |
|---|---|---|
| `MetricSpan` | jedna izmjerena operacija; `depth` čini isključivo vrijeme izračunljivim | clean core |
| `MetricSample` | jedna objavljena točka niza; `buckets` razdvaja razdiobu od broja | clean core |
| `MetricKind` | `COUNTER` · `GAUGE` · `HISTOGRAM` | clean core |
| `MetricSet` | **vlasnik izvođenja** — brojači, kante, protok; ništa ne stiže već agregirano | clean core |
| `MetricCollector` | `logging.Handler`; jedino mjesto koje **ispravlja** | clean core |
| `MetricSink` | apstraktno odredište | clean core |
| `MetricReporter` | drži *N* odredišta nad **jednim** skupom uzoraka | clean core |
| `PrometheusSink` | slaže exposition **kao tekst** — strukturirani put drivera ne emitira `# TYPE` | processors |
| `GrafanaSink` | anotacije: početak i kraj prolaza, kvarovi | processors |

**Što je ponovno upotrijebljeno:** `Audit.subscribe_handler()` kao postojeći priključak; oblik
razlijevanja iz `RepositoryWithDriver` (*N* odredišta nad jednim predmetom) — **kao oblik**, ne
kao nasljeđivanje; i pravilo da prijenos radi driver.

**Što namjerno nije:** `IRepository` (ugovor mu je `read(identifier) -> ITarget`; mjera nema
identitet za dohvat, pa bi ime posudilo riječ bez mehanizma), `StrategyWrite` (`execute` asertira
`ITarget`, a uzorci nisu facade), i `IObserver` na kolektoru (prima **zapise**, ne domenske
događaje; `IEvent` nema izvedbu u stablu, pa bi takva tipizacija bila aspiracija).

**Granica distribucije (§7.1):** sve lijevo od odredišta uvozi samo stdlib i `wattleflow`, pa
ostaje u clean coreu. Dva odredišta imenuju Prometheus i Grafana drivere, pa žive u
`blackwattle` — deklariranoj iznimci (§7.4).

## 8. Rezultat — objavljeni skup

```
wf_documents_total{workflow,processor,outcome}         brojač; outcome=completed|failed
wf_bytes_total{workflow,driver,direction}              brojač; direction=read|write
wf_operations_total{workflow,component,method,outcome} brojač
wf_operation_seconds{component,method,scope}           histogram; scope=inclusive|exclusive
wf_document_bytes{processor,kind}                      histogram
wf_open_spans{component}                               stanje — zdravlje mjerenja
wf_run_info{workflow,version}                          stanje; uvijek 1
```

Jedinice: `seconds`, `bytes`, `documents`. Skala omjerna. Oznake su **racionalne podgrupe**
(`NFRQ-OBS-04` k.3) i zatvoren su skup; `filename`, identifikator dokumenta i putanja **nikad**
nisu oznake (k.4).

## 9. Kriteriji prihvaćanja

1. Nijedna komponenta ne uvozi kolektor niti ga imenuje. — *pregled uvoza*
2. Kolektor zatvara raspon i na `Failed`. ❌ *ovisi o §12 t.1*
3. Zaostali raspon ističe i broji se; ne visi. — *provjerivo testom*
4. Protok se izvodi, nigdje se ne pohranjuje kao ulaz. — *pregled*
5. Trajanja se objavljuju kao razdioba (`_count`/`_sum`/kante), ne kao vrijednost. — *pregled objavljenog skupa*
6. Svaki brojač grešaka ima brojač pokušaja. — *pregled*
7. `scope` je deklariran na svakoj mjeri trajanja. ❌ *odluka, §12 t.2*
8. Kardinalnost je ograničena i deklarirana; nijedna oznaka nije identifikator. — *pregled*
9. Metrika radi kad je `DEBUG` isključen. ❌ *ovisi o odluci o kanalu, §12 t.3*
10. `scikit`-a i Prometheus klijenta nema u clean core distribuciji. — `wem_lint` `clean_core_imports`

## 10. Verifikacija — izmjereno prototipom

Prototip kolektora (scratchpad, izvan `src/`), četiri dokumenta od kojih jedan namjerno neuspješan:

| komponenta | metoda | događaj | n | ok | fail | ukupno ms | izvedeni protok |
|---|---|---|---|---|---|---|---|
| `DriverPdf` | `extract` | Read | 3 | 3 | 0 | 11,00 | 96 325 j/s |
| `CreatePdfDocument` | `execute` | Create | 4 | 4 | 0 | 7,73 | — |
| `WritePdfArchive` | `execute` | Write | 3 | 3 | 0 | 4,90 | — |
| `DriverPdf` | `copy` | Copy | 3 | 3 | 0 | 2,62 | 3 174 058 j/s |

Trajanje i količina izvedeni su **bez ijedne izmjene koda**. Ali:

- **neuspjeh je nevidljiv** — `fail=0` iako je jedan dokument pao; par je ostao otvoren jer put
  kvara ne piše `Failed`;
- ugniježđenost mjeri isti rad četiri puta: `flush` 26,62 → `repository.write` 26,51 →
  `strategy` 26,32 → **`driver.copy` 25,79 ms**;
- `WritePdfArchive` ne nosi količinu u završnom zapisu, pa mu protok nije izvediv.

**Trojka reproducibilnosti (D-10):** alat — `logging.Handler` nad `record.created`/`funcName`,
uparivanje po `(logger, funcName, event)`; kriterij — §9 gore; platforma — CPython 3.11.15,
Linux/WSL2, 2026-09-07. **Mjereno stablo:** prototip u scratchpadu, nijedna produkcijska datoteka.
**Slijepa pjega:** jedan dokument po prolazu, jedan proces, bez dretvi — uparivanje po `funcName`
nije provjereno pri istodobnosti ni rekurziji.

## 11. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-OBS-04` | razdioba umjesto točke; oznake kao podgrupe; ograničena kardinalnost; deklariran `scope` |
| `NFRQ-DEF-02` | svaka mjera je dijagnostička; promocija u vrata traži DR |
| `NFRQ-OBS-03` | kolektor ne emitira po stavci; izvještaj nosi ishodne brojače na granici prolaza |
| `NFRQ-SEC-03` | Prometheus/Grafana driveri pripadaju `blackwattle` (§7.4 iznimka) |
| `NFRQ-ORG-01` | kolektor je sposobnost dviju domena → dijeljeni helper, ne domenski primitiv |

## 12. Otvoreno

1. **Parovi nisu potpuni, i to sustavno.** Statički: `SmallBlackboard.write` ima 2 `Started` i **0**
   `Completed`; `flush` 2/1; `GenericProcessor.start` 1/0; `DriverLocalStorage.search` i `copy`
   2/1. Uz to **14 `raise` izlaza** u `drivers/local_storage.py` (10) i `drivers/pdf.py` (4) nema
   nikakav zapis, pa neuspjeh ne zatvara par. Izmjena je mehanička — jedan zapis po izlazu — ali
   je preduvjet za k.2 i k.3.
2. **`scope`: uključivo, isključivo ili oboje.** Bez odluke mjera nema operativnu definiciju
   (`NFRQ-OBS-04` k.6), a zbroj po slojevima je četverostruko precijenjen.
3. **Jedan kanal ili dva.** Metrika na istom loggeru kao dijagnostika gasi se s `DEBUG`-om
   (izmjereno: `WARNING` → kolektor primi 0 zapisa), a držanje `DEBUG`-a košta **+32%** po
   dokumentu. Vlastiti kanal to razdvaja, po cijenu druge točke emisije.
4. **Guranje ili struganje.** Pushgateway drži zadnju vrijednost po grupi i namijenjen je poslovima
   koji završe; dugotrajni workflow traži `/metrics` koji Prometheus struže. Mijenja driver, ne
   mjere. Uz to treba provjeriti odbija li Pushgateway uzorke s vremenskom oznakom — driver nudi
   `timestamp_ms`, a to bi tada vratilo grešku.
5. **`# TYPE` i `# HELP`.** Strukturirani put `DriverPrometheus`-a emitira samo
   `ime{oznake} vrijednost`; tip i opis prolaze jedino kroz sirovi string. Histogram je izraziv,
   ali ga mora složiti pozivatelj.
6. **Granice kanti** nisu izvedive prije nego postoji povijest procesa (`NFRQ-OBS-04` §6 t.2).
