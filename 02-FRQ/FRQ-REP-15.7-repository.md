# FRQ-REP-15.7 — Generičko spremište i varijanta s driverom

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `REP`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericRepository(Wattleflow, IRepository, ABC)` i `RepositoryWithDriver(GenericRepository)` |
| **Sestrinski** | [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) (jedini pozivatelj `write`) · [`FRQ-STR-15.4`](FRQ-STR-15.4-strategy.md) (čita i piše) · [`FRQ-DRV-15.6`](FRQ-DRV-15.6-driver.md) (kanal prema vanjskom sustavu) |
| **Izvedba** | `workflow/src/wattleflow/concrete/repository.py` (394 linije) |

## 1. Predmet

Spremište je **odredište** stavke: ono zna gdje se piše i odakle se čita, ali **ne zna kako**. Kako
je posao strategije (`FRQ-STR-15.4`), a kroz što — drivera (`FRQ-DRV-15.6`). Spremište je dakle
mjesto na kojem se te tri stvari sastaju, i jedino koje broji koliko je zapisa prošlo.

Dvije klase, jedna razlika:

| klasa | ima driver | čemu služi |
|---|---|---|
| `GenericRepository` | ne | odredište kojem je dovoljna strategija (memorija, lokalni izlaz) |
| `RepositoryWithDriver` | da (`ALLOWED = ["driver"]`) | odredište iza vanjskog sustava; driver ide **strategiji u `kwargs`** |

Ta zadnja stavka je važnija nego što izgleda: strategija ne dohvaća driver, nego ga **prima**.
`RepositoryWithDriver.write` predaje `driver=self.driver`, i to je jedini kanal kojim write-
strategija u processorsu dolazi do vanjskog sustava — otud i ~15 mjesta koja provjeravaju
„Driver not in kwargs — strategy requires RepositoryWithDriver".

Spremište pri pozivu strategije predaje **sebe** kao `caller`, ne pozivatelja iznad sebe. Time
pozivni kontekst blackboarda ne curi u sloj strategije.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi spremište iz konfiguracije | `strategy_write`, `strategy_read`, `driver` |
| **A2** | `GenericBlackboard` — jedini pozivatelj `write` | `caller`, `facade` |
| **A3** | `StrategyWrite` / `StrategyRead` | operacija |
| **A4** | `GenericDriver` — samo u varijanti s driverom | vanjski sustav |
| **A5** | `GenericRepository` — predmet ovog zahtjeva | brojač i uvezivanje |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira spremište → `__init__` |
| **EV02** | A2 piše pri flushu → `write(caller, facade)` |
| **EV03** | pozivatelj čita → `read(identifier)` |
| **EV04** | brojanje se resetira → `clear()` |
| **EV05** | strategija ili driver padne |

## 4. Preduvjeti

1. `strategy_write` je `StrategyWrite` — `assert` prije `super().__init__`.
2. `strategy_read`, ako je zadan, je `StrategyRead`; ako nije zadan, čitanje je onemogućeno.
3. Za `RepositoryWithDriver`: `driver` je zadan — instanca `GenericDriver` ili putanja klase koju
   razrješava `ClassLoader`.
4. `caller` je `IBlackboard`, `facade` je `ITarget` — provjereno u `write`.

## 5. Normalan tok

1. **EV01** — strategije se provjere, preset preuzme konfiguraciju, brojač krene od nule.
   `RepositoryWithDriver` uz to razriješi driver: instancu prihvaća kakva jest, string tumači kao
   putanju klase i instancira ga kroz `ClassLoader`.
2. **EV02** — `write` prijavljuje `DEBUG` s pozivateljem i brojačem, pa provjeri tipove.
3. **Zapis na ulazu**, na `INFO`: ime write-strategije, identifikator dokumenta i dosadašnji broj
   zapisa — prije nego strategija krene, da audit tok slijedi pozivni red (`DR-WFL-021`).
4. Strategija se poziva s `caller=self`, `facade=`, `repository=self` i — u varijanti s driverom —
   `driver=self.driver`.
5. Brojač se poveća; rezultat strategije se vraća pozivatelju.
6. **EV03** — `read` proslijedi na read-strategiju s `caller=self` i `identifier=`.
7. **EV04** — `clear()` vraća brojač na nulu.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `strategy_write` nije `StrategyWrite` | `AssertionError` prije konstrukcije | spremište bez ispravne strategije ne nastaje |
| `driver` izostavljen u varijanti s driverom | `ValueError` iz konstruktora | vidi §11 t.4 |
| čitanje bez read-strategije | `warning` + `None` | odsutnost čitanja nije kvar nego konfiguracija |
| `write` strategija padne | `RepositoryException(caller=self, error=…, facade=…) from e` | uzrok očuvan (`BR-15-09`) |
| kvar u varijanti s driverom | poruka nosi **strategiju, driver, dokument, pozivatelja i korijenski tip** | dijagnostika ne traži čitanje traga |
| `read` strategija padne u `GenericRepository` | `RepositoryException` | omotano |
| `read` strategija padne u `RepositoryWithDriver` | **iznimka izlazi neomotana** | vidi §11 t.1 |
| `clear()` u `GenericRepository` | resetira brojač; spremište ostaje upotrebljivo | — |
| `clear()` u `RepositoryWithDriver` | resetira brojač, čisti driver i **poništava obje strategije** | spremište postaje neupotrebljivo — vidi §11 t.2 |

## 7. Rezultat

Stavka je trajno zapisana kroz strategiju koja zna format i driver koji zna protokol, a spremište
zna samo koliko ih je prošlo. Zamjena odredišta je izmjena konfiguracije (`BR-15-01`): ni platno
ni pipeline ne znaju je li iza spremišta datoteka, baza ili broker.

## 8. Kriteriji prihvaćanja

1. Strategije se provjeravaju prije konstrukcije. ✅
2. Spremište predaje **sebe** kao `caller`; kontekst iznad ne curi u strategiju. ✅
3. Driver stiže strategiji kroz `kwargs`, iz spremišta — strategija ga ne dohvaća sama. ✅
4. Otvarajući `INFO` nastaje prije strategije (`DR-WFL-021`). ✅
5. Poruka kvara u varijanti s driverom imenuje strategiju, driver, dokument i pozivatelja. ✅
6. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
7. `read` je omotan u `RepositoryException` u **obje** klase (`BR-15-09`). ❌ — §11 t.1
8. `clear()` znači isto u obje klase. ❌ — §11 t.2
9. `hash()` radi na svakoj podklasi `GenericRepository`. ❌ — §11 t.3
10. Komentari su na UK engleskom (`CLAUDE.md` §2.4). ❌ — §11 t.6

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `__init__` | dva `assert`-a prije `super().__init__` |
| 2 | pregled poziva strategije | `caller=self` u sva četiri poziva |
| 3 | pregled `RepositoryWithDriver.write` | `driver=self.driver` u `kwargs` strategije |
| 4 | pregled redoslijeda | `self.info(...)` prethodi `self._strategy_write.write(...)` |
| 5 | pregled `except` grane | poruka nosi `strategy`, `driver`, `document`, `caller`, `root` |
| 6 | pregled modula | `__all__` = 2 imena; uvozi samo `abc`, `typing` + `wattleflow.*` |
| 7 | usporedba obiju `read` metoda | roditelj: `try/except → RepositoryException`; dijete: **bez `try`** |
| 8 | usporedba obiju `clear` metoda | roditelj: brojač + `INFO`; dijete: brojač + driver + strategije `= None`, `DEBUG` |
| 9 | pregled `GenericRepository.__hash__` | referira `self._driver`, kojeg roditeljski `__slots__` nema |
| 10 | pregled komentara | „Strategy.execute **asertira** IRepository" ×3 |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` i `processors` radna stabla 2026-08-27, CPython 3.11 (Linux/WSL2).
**Mjereno stablo:** `concrete/repository.py` + `processors/src/wattleflow/strategies/documents/`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Repository` je rezervirani primitiv; ono je odredište, ne platno (`FRQ-BBD-15.1`) |
| `NFRQ-SEC-01` | spremište poznaje jedan driver; kompromitacija jednog odredišta ne doseže druga |
| `NFRQ-SEC-02` | `ALLOWED = ["driver"]` je cijela konfiguracijska površina varijante s driverom |
| `NFRQ-SEC-03` | clean core tier; `ClassLoader` razrješava specijalizaciju po imenu, bez uvoza third-partyja u ovaj modul |
| `NFRQ-OBS-01` | jedan `INFO` po zapisu; `clear()` je `INFO` u roditelju i `DEBUG` u djetetu — nedosljedno (§11 t.2) |
| `NFRQ-OBS-02` | otvarajući zapis nosi `strategy`, `document`, `written` |
| `NFRQ-ORG-08` | dvije `write` metode dijele ~40 linija gotovo doslovno — otvoreni DRY klaster (§11 t.5) |

## 11. Otvoreno

1. **`RepositoryWithDriver.read` ne omata kvar.** Roditeljski `read` hvata iznimku i diže
   `RepositoryException` s očuvanim uzrokom; nasljedna verzija poziva strategiju bez `try` uopće.
   Kvar čitanja iz spremišta s driverom stiže pozivatelju kao izvorna iznimka strategije ili
   drivera, pa trag ne kaže koje je spremište pucalo. `BR-15-09` je prekršen u djetetu, ne u
   roditelju.
2. **`clear()` znači dvije različite stvari.** U roditelju resetira brojač; u djetetu uz to čisti
   driver i **postavlja obje strategije na `None`**, čime spremište postaje neupotrebljivo — svaki
   sljedeći `write` pada. Ista metoda, isto ime, nespojiva semantika; uz to različita razina
   zapisa (`INFO` vs `DEBUG`), što `NFRQ-OBS-01` mjeri. Ako je dječja verzija zapravo `close()`,
   treba je tako i zvati.
3. **`GenericRepository.__hash__` referira `self._driver`, koji ta klasa nema.** Atribut pripada
   djetetu; roditeljski `__slots__` ga ne sadrži. Podklasa `GenericRepository` bez drivera na
   `hash()` (a time i na `__eq__`) propada kroz `__getattr__` u preset. Danas takva podklasa ne
   postoji u trima stablima — kvar je latentan, ne aktivan.
4. **Nedostajući driver diže goli `ValueError`.** Svaki drugi kvar ovog modula izlazi kao
   `RepositoryException` (`BR-15-09`); ovdje ne. Isti razred problema kao `RuntimeError` u
   konekciji (`FRQ-CON-15.5` §11 t.3).
5. **Dvije `write` metode su gotovo doslovni duplikat.** Razlikuju se u dvije stvari: `driver=` u
   pozivu strategije i bogatija poruka kvara. Ostalih ~35 linija je isto. Kandidat: dijete
   nadjačava samo dvije kuke (`_strategy_kwargs()` i `_failure_reason()`), a tok ostaje u
   roditelju (`NFRQ-ORG-08`).
6. **Hrvatski u komentarima koda**, tri mjesta: „Strategy.execute **asertira** IRepository" i
   inačice. Uz `CLAUDE.md` §2.4 (UK engleski) to je i anglicizam. Isti nalaz nosi
   `processors/strategies/documents/avro.py:102` („Driver **stize** kroz kwargs…").
7. **Komentar opisuje zapis koji ne postoji.** Obje `write` metode nose komentar „The repository
   … reports its OWN completion. One record, at completion only" — a zatvarajućeg `INFO` zapisa
   nema; postoji samo otvarajući, koji drugi komentar par redaka iznad ispravno opisuje. Dva
   komentara u istoj metodi tvrde suprotno.
8. **`GenericRepository.__repr__` ima pogrešno imenovanu varijablu:** `level = self.name or
   "UNKNOWN"` — ime kaže razina, vrijednost je ime, pa se ime ispisuje dvaput. Dječja verzija
   koristi `self.levelname`, kako i treba.
