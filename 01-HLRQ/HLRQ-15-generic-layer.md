# HLRQ-15 — Cjevovodna obrada kroz zamjenjive primitive (generički sloj)

> **Razred `HLRQ` je provizoran.** Nije u vokabularu registra (`CLAUDE.md` §3.6); uvođenje traži
> DR (D-12). Kategorije djece su u vokabularu od [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md).

| | |
|---|---|
| **Status** | Provedeno u kodu, **nezapisano do sada** (2026-08-27) — obrnuto inženjerstvo zatečenog sloja |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorije po ulozi; os je ontološki primitiv |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `wattleflow-workflow` — clean core tier (`CLAUDE.md` §7.1) |
| **Predmet** | `src/wattleflow/concrete/` — 22 modula, radna osnova frameworka (`CLAUDE.md` §4 t.2) |
| **Djeca** | [`FRQ-BBD-15.1`](../02-FRQ/FRQ-BBD-15.1-blackboard.md) · [`FRQ-PIP-15.2`](../02-FRQ/FRQ-PIP-15.2-pipeline.md) · [`FRQ-PRC-15.3`](../02-FRQ/FRQ-PRC-15.3-processor.md) · [`FRQ-STR-15.4`](../02-FRQ/FRQ-STR-15.4-strategy.md) · `FRQ-CON-15.5` · `FRQ-DRV-15.6` · `FRQ-REP-15.7` · `FRQ-DOC-15.8` · `FRQ-WFL-15.9` · `FRQ-MEM-15.10` · `FRQ-PTN-15.11…15.21` (u izradi) |
| **Sljedivost** | `NFRQ-ORG-04` (ontologija) · `NFRQ-SEC-01` (blast radius) · `NFRQ-SEC-03` (lokalnost distribucije) · `NFRQ-OBS-01/02/03` (audit zapis) · `DR-COR-002`, `DR-WFL-009`, `DR-WFL-018`, `DR-WFL-021` |

## 1. Narativ

Podatkovni posao se u praksi piše kao skripta: dohvat, transformacija i pohrana spleteni u jedan
tok. Takav tok radi dok se ne promijeni jedan njegov kraj — drugi izvor, druga baza, drugi format
— a tada se mijenja **cijeli** jer nijedan njegov dio nema samostalno sučelje.

`core/` na to odgovara skupom apstraktnih ugovora izvedenih iz dizajn patterna. Ali ugovor sam
ne pokreće ništa: između sučelja i konkretnog posla nedostaje sloj koji ugovor **ispunjava** —
koji zna kako se objekt gradi, kako se prijavljuje u audit, kako se ruši, kako se njegovo stanje
sprema i vraća. Bez tog sloja svaka bi specijalizacija te odgovornosti izvodila iznova, i to
različito.

**Zašto.** Sloj `concrete/` postoji da bi zamjena bilo kojeg primitiva bila **izmjena
konfiguracije, ne koda**. Cijena za to je da generička klasa preuzme sve što je zajedničko —
identitet, audit, životni ciklus, automat stanja, granice kvara — a specijalizaciji ostavi
isključivo ono što je za nju specifično. Mjera uspjeha je koliko malo specijalizacija mora
napisati, a ne koliko generička klasa može.

**Zatečeno.** Sloj postoji i radi; ovaj zapis ga ne uvodi nego **zapisuje**. Do 2026-08-27
`concrete/` nije imao nijedan zahtjev u registru — kod je bio zreliji od svojeg zapisa
(`CLAUDE.md`, zaglavlje). Sve što slijedi izvedeno je iz koda, a razilaženje koda i namjere
prijavljeno je kao nalaz, ne izglađeno u tekstu (D-11).

## 2. Mjesto u dekompoziciji

```
core/          apstraktni ugovori (IWattleflow, IBlackboard, IProcessor, …)   ← autoritativno
   ↑
concrete/      generičke implementacije                                       ← OVAJ ZAHTJEV
   ↑
specijalizacije (connections/, drivers/, pipelines/, strategies/, …)          ← blackwattle
```

Ovisnost je **jednosmjerna** (`CLAUDE.md` §7.2): `concrete/` ne smije uvoziti iz specijalizacija.
Ta jednosmjernost je ono što `concrete/` drži u clean core tieru — sloj koji bi posegnuo za
third-party ovisnošću preselio bi cijelu distribuciju (`NFRQ-SEC-03`).

## 3. Ontologija funkcionalnosti

Deset rezerviranih primitiva (`CLAUDE.md` §1) i njihova uloga u toku posla:

| primitiv | odgovornost | generička klasa |
|---|---|---|
| **Workflow** | gradi i drži cjelinu; vlasnik konfiguracije | `GenericWorkflow` + `WorkflowFactory` |
| **Processor** | vodi prolaz nad skupom stavki; vlasnik ciklusa | `GenericProcessor` |
| **Pipeline** | jedna transformacija nad jednom stavkom | `GenericPipeline` |
| **Blackboard** | dijeljeni radni prostor između pipelinea i spremišta | `GenericBlackboard` |
| **Repository** | trajno spremište stavki | `GenericRepository`, `RepositoryWithDriver` |
| **Driver** | operacije nad vanjskim sustavom | `GenericDriver`, `LazyDriverProxy` |
| **Connection** | pristup vanjskom sustavu | `GenericConnection` |
| **Strategy** | zamjenjivi algoritam jedne operacije | `Strategy` + četiri obitelji |
| **Document** | jedinica podatka koja putuje tokom | `Document`, `DocumentAdapter`, `DocumentFacade` |
| **Memento** | snimka stanja za nastavak | `GenericMemento` |

Uz njih, sloj nosi i **pattern-infrastrukturu** koja nije primitiv: korijenska baza, orkestrator,
scheduler, manageri, observable, automat stanja, iterator, serijalizacija, singleton, hijerarhija
iznimaka, `Attribute`/`NameHelper`. Ona ima jednu oznaku (`PTN`), namjerno
([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) t.3).

## 4. Opseg

**U opsegu:** sve u `src/wattleflow/concrete/`.

**Izvan opsega:** `core/` (autoritativno, serija `DR-COR`), specijalizacije u
`blackwattle` (vlastite sposobnosti), te `helpers/`, `decorators/`, `enums/`,
`constants/` — dijeljena imena bez `__init__.py` (`DR-WFL-017`), koja ne pripadaju ovom sloju.

### Zajednički ugovor generičke klase

Svaka generička klasa ovog sloja:

1. **Nasljeđuje `Wattleflow` prvo**, ispred svojeg pattern sučelja, i nikad ne imenuje `Audit`
   sama. Redoslijed baza je **ograničenje, ne stil** — MRO se inače ne linearizira
   (`TODO.md` §`__slots__` i MRO).
2. **Prosljeđuje cijeli `**kwargs` naviše nepromijenjen.** Razdvajanje logging ključeva od
   ostatka događa se na jednom mjestu, u `Wattleflow.__init__`; podklasa koja to ponovi
   duplicira podjelu i razilazi se čim se doda novi ključ.
3. **Prijavljuje na ulazu, ne na izlazu.** Audit tok se čita odozgo nadolje redom kojim se posao
   odvija (`DR-WFL-021`); zatvaranje jedinice pripada sloju koji jedinicu posjeduje — procesoru i
   workflowu.
4. **Ne diže iznimku iz destruktora.** `__del__` prijavljuje kvar i nastavlja; iznimka u
   destruktoru maskira onu koja se stvarno dogodila.
5. **Deklarira `__slots__`** gdje drži stanje, i **`__all__`** u modulu (`NFRQ-SEC-02` t.2).

## 5. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-15-01** | Zamjena primitiva je izmjena konfiguracije, ne koda. Klasa se razrješava po imenu iz konfiguracije (`WorkflowFactory.resolve`). |
| **BR-15-02** | Konfiguracija koja se ne razriješi zaustavlja workflow **prije** obrade, ne tijekom nje. |
| **BR-15-03** | Svaki objekt frameworka nosi identitet izveden iz vlastitog tipa; identitet se ne postavlja izvana i ne mijenja nakon konstrukcije (`DR-COR-002`). |
| **BR-15-04** | Auditabilnost nije opcija. Nasljeđuje se na jednom deklariranom mjestu, pa je svaki potomak nosi po ugovoru, ne kao nuspojavu druge baze (`DR-WFL-009`). |
| **BR-15-05** | Prijelaz stanja koji automat ne dopušta ne izvodi se. Stanje se mijenja isključivo primjenom akcije nad tablicom prijelaza. |
| **BR-15-06** | Kvar u jednom prolazu ne ostavlja objekt u stanju iz kojeg se ne može ni nastaviti ni čisto završiti — `FAILED` je stanje iz kojeg vodi oporavak (`LOAD`) i završetak (`CLEAN`/`STORE`). |
| **BR-15-07** | Sloj ne posjeduje granicu prema vanjskom sustavu. Mrežu, disk i baze dodiruju driver i konekcija; generička klasa iznad njih ne poznaje protokol. |
| **BR-15-08** | Tajna se ne zapisuje. Vrijednost kredencijala nikad ne ulazi u audit zapis; zapisuje se samo *je li* autentikacija postignuta (`NFRQ-SEC-06`). |
| **BR-15-09** | Iznimka jednog sloja ne prolazi kroz drugi nepromijenjena — svaki sloj je omata u vlastiti razred, s uzrokom (`raise … from e`), da trag kaže **gdje** je puklo. |

## 6. Nefunkcionalni zahtjevi

| NFR | posljedica za ovu sposobnost |
|---|---|
| [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) | deset primitiva je zatvoren popis; *cross-cutting* sposobnost je pozivljivi helper, ne novi primitiv |
| [`NFRQ-SEC-01`](../03-NFRQ/NFRQ-SEC-01-blast-radius-EN.md) | jedan primitiv = jedna odgovornost; kompromitacija drivera ne doseže blackboard |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface-EN.md) | `__all__` u svakom modulu; `PresetDecorator` je jedina ulazna površina za konfiguraciju |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) | import closure sloja je `stdlib ∪ wattleflow`; nijedan modul ne smije referirati third-party |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality-EN.md) | `BR-15-08`; redakcija je odgovornost `Audit` obitelji, ne pojedine klase |
| [`NFRQ-OBS-01/02/03`](../03-NFRQ/NFRQ-OBS-01-audit-levels-EN.md) | razina, imena polja i volumen po jedinici posla; mjeri se lintom |
| [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers-EN.md) | metoda koja referira vlastitu klasu je `@classmethod` |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication-EN.md) | zajedničko ponašanje živi u generičkoj klasi, ne prepisano po specijalizacijama |

**OSCAL:** ovaj sloj **ne nosi** OSCAL dekoratere. Sloj usklađenosti živi u
`blackwattle` (`DR-WFL-015`), a `concrete/` je clean core — nijedna OSCAL referenca ni
ovisnost (`CLAUDE.md` §6.1).

## 7. Otvoreno

1. **Broj sposobnosti `15` je prvi slobodan redni broj**, kao i `14` u `DR-WFL-013` t.3 — nije
   izveden ni iz čega u kodu.
2. **Automat stanja nije jednako usvojen.** Konekcija, driver i procesor grade automat u
   **generičkoj** klasi; blackboard ga izvozi kao tablicu i prepušta **specijalizaciji**. Je li
   ta asimetrija namjerna, nije zapisano — posljedica je da blackboard bez automata prolazi bez
   ijedne provjere prijelaza (vidi `FRQ-BBD-15.1` §11).
3. **`Wattleflow` je jedini `__slots__ = ()` korijen, ali nisu sve podklase pod `__slots__`.**
   `GenericPipeline` i `Strategy` obitelj ih ne deklariraju, pa nose `__dict__` — mjerljiva
   razlika u memoriji po stavci koju nijedan zahtjev ne opravdava.
4. **Test framework nije odabran** (`CLAUDE.md` §4), pa je verifikacija svakog djeteta ovog
   zahtjeva **pregled i ručno izvođenje**, ne automatiziran test. Deklarirana slijepa pjega
   (D-11) zajednička cijeloj skupini.
5. **Status skupine je „provedeno u kodu, nezapisano".** Status se mijenja kroz DR, ne prešutno
   (D-03); djeca ovog zahtjeva pišu se s tim statusom dok ih DR ne prevede u *prihvaćen*.
