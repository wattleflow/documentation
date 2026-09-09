# FRQ-OSCAL-14.13 — Komponentne baze pod OSCAL vratima

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Provedeno (2026-08-21) |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) (kategorija); izvedba u `wattleflow-workflow` traži `DR-WFL` — §11 t.1 |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `OSCALConnection`, `OSCALDriver`, `OSCALProcessor` |
| **Sestrinski** | [`14.11`](FRQ-OSCAL-14.11-policy.md) vrata i politika |
| **Izvedba** | `blackwattle`: `connections/oscal.py`, `drivers/oscal.py`, `processors/oscal.py` |

## 1. Predmet

Tri apstraktne baze koje nasljeđuju generičke primitive frameworka (`GenericConnection`,
`GenericDriver`, `GenericProcessor`) i **nose dekorater**. Komponenta koja ih naslijedi je pod
vratima; komponenta koja ih ne naslijedi nije. Izbor je izražen **tipom**, ne dosjetkom autora.

To rješava rupu koju je `BR-OSCAL-11` do sada samo imenovao: dekorater po klasi mora se **zapamtiti**,
a propust se ne vidi ničim. Baza se ne zaboravlja — vidi se u deklaraciji klase i prolazi kroz review.

| baza | nasljeđuje | dekorater | pravilo ili izbor |
|---|---|---|---|
| `OSCALConnection` | `GenericConnection` | `@oscal_connection(strict=False)` | **pravilo** — konekcija je granica prema vanjskom sustavu; svih 12 u distribuciji deklarira kontrole |
| `OSCALDriver` | `GenericDriver` | `@oscal_driver(strict=False)` | **izbor** — driver provodi kontrole samo kroz konekciju; lokalni driver ostaje na `GenericDriver` |
| `OSCALProcessor` | `GenericProcessor` | `@oscal_processor(strict=False)` | **izbor** — procesor koji sam drži pristupni put |

Baze deklariraju `OSCAL_CONTROLS = ()` — **obvezu, ne kontrole**. Prazna deklaracija znači da baza
sama nije subjekt provjere, a `controls_strategy="merge"` iz nje ne unira ništa, pa deklaracija
svake konkretne klase ostaje točna (`BR-OSCAL-04`).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Konkretna komponenta (konekcija, driver, procesor) | `OSCAL_CONTROLS` |
| **A2** | Baza — predmet ovog zahtjeva | dekorater i ugovor |
| **A3** | Dekorater `wattleflow.decorators.oscal` | provjera i čuvani FSM |
| **A4** | Pozivatelj / tvornica | `oscal_policy=` (danas: nitko — §11 t.3) |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 se konstruira → dekorater razrješava deklaraciju kroz MRO |
| **EV02** | provjera **prije** `__init__` bilo koje baze |
| **EV03** | prvi prijelaz FSM-a → čuvar provjerava i upisuje audit trag |

## 4. Preduvjeti

1. ~~`wattleflow-oscal` je instaliran.~~ **Otpalo 2026-08-22:** dekorater uvozi `wattleflow.oscal`
   iz istoga stabla, pa vanjskog preduvjeta nema.
2. Konkretna klasa deklarira `OSCAL_CONTROLS` i **ne ponavlja** dekorater (§6).

## 5. Normalan tok

1. EV01 — dekorater sabire deklaraciju kroz MRO (`merge`).
2. EV02 — **provjera prethodi konstrukciji**. To je bitno za konekcije: `GenericConnection.__init__`
   na kraju zove `ensure_created()`, dakle otvara vezu. Prije ispravke (§9) vrata su se postavljala
   *nakon* tog poziva, pa je neusklađena konekcija stigla otvoriti vezu i pala tek na sljedećem
   prijelazu. Sada pada prije nego išta otvori.
3. Prazna deklaracija → provjera se preskače; baza sama nikad nije subjekt.
4. EV03 — čuvani FSM provjerava još jednom pri prvom prijelazu i tada upisuje audit
   (`Validating` / `Validated` s `uuid`-om profila) — u konstruktoru zapisa još nema jer logger
   ne postoji.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| deklaracija izvan baselinea, politika predana | `OSCALPolicyError` **u konstruktoru**; ništa se ne otvara |
| bez politike, `strict=False` (danas) | provjera se preskače — vrata su inertna (§11 t.3) |
| komponenta ne deklarira ništa | prolazi; baza je bez učinka |
| konkretna klasa **ponovi** dekorater | dva sloja čuvara; vanjski popne `oscal_policy` pa unutarnji pod `strict=True` odbija ispravno konfiguriranu komponentu ([`14.11`](FRQ-OSCAL-14.11-policy.md) §11 t.7) |
| komponenta bez `_fsm` | provjera iz konstruktora je tada cijela vrata |

## 7. Rezultat

Vrata su svojstvo hijerarhije, ne discipline. Nova konekcija u distribuciji nasljeđuje
`OSCALConnection` i time je pod provjerom; driver koji ništa vanjsko ne dodiruje ostaje izvan nje,
i to se vidi iz njegove deklaracije.

## 8. Kriteriji prihvaćanja

1. Tri baze postoje, apstraktne su i nose dekorater svoje uloge. ✅
2. Svaka konkretna klasa koja deklarira `OSCAL_CONTROLS` nasljeđuje odgovarajuću bazu. ✅
3. Nijedna konkretna klasa ne ponavlja dekorater. ✅
4. Provjera prethodi konstrukciji — neusklađena konekcija ne otvara vezu. ✅
5. Baze deklariraju praznu obvezu i ne unose kontrole u potomke. ✅
6. Imena su u lijenom agregatu (`_EXPORTS`), pa se razrješavaju iz paketa. ✅
7. Konstrukcija koja padne ne proizvodi „Exception ignored in `__del__`". ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1, 2, 3 | uvoz svake deklarirajuće/dekorirane klase uz podmetnute third-party stubove (`verify_gates.py`) | **18 klasa, 0 grešaka**; 15 pod vratima **kroz bazu**, 3 baze nose vlastiti dekorater, `kontrola=0` |
| 4 | neusklađena konekcija nad stvarnom `OSCALConnection` (`base_e2e.py`) | `OSCALPolicyError` u konstruktoru, trag prazan — `create_connection` se nije izvršio |
| 4 | ista prije ispravke dekoratera (`order_probe.py`) | konstrukcija je **prošla**, trag `['ensure_created', 'OTVORENA VEZA']`, kvar tek na `connect()` |
| 5 | `OSCALConnection.OSCAL_CONTROLS` | `()`; potomci zadržavaju svoje četiri |
| 6 | `wattleflow.connections.OSCALConnection` i sestrinska imena | razrješavaju se; agregati 24 / 41 / 19 imena |
| 7 | ista konstrukcija koja padne | bez `AttributeError: … has no attribute '_logger'` na stderr |

**Trojka (D-10):** alat = `verify_gates.py`, `base_e2e.py`, `order_probe.py`; kriterij = §8 t.1–7;
platforma = Python 3.11.15, Linux (WSL2), radna stabla `processors` / `workflow` / `oscal`
2026-08-21.

> **Slijepa pjega (D-11).** Skripte nisu u repozitoriju (`CLAUDE.md` §4). Provjera uvoza koristi
> **stubove** umjesto stvarnih third-party knjižnica: dokazuje da se wattleflow kod uvozi i da su
> vrata postavljena, ne da knjižnice rade.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-ORG-08` | pravilo „komponenta pod vratima" postoji na jednom mjestu po ulozi, umjesto 14 puta |
| `NFRQ-ORG-01` | baza živi u paketu svoje vrste (`connections/`, `drivers/`, `processors/`), uz svoje potrošače |
| `NFRQ-SEC-02` | dekorater je na bazi; konkretne klase ne izlažu novu površinu |
| `NFRQ-SEC-03` | baze uvoze samo `wattleflow.*` — tier ostaje `stdlib ∪ wattleflow` |
| `NFRQ-ORG-04` | ne uvodi se novi primitiv: `OSCALConnection` je specijalizacija `Connection`, ne nova ontološka vrsta |

## 11. Otvoreno

1. **Dvije izmjene u `wattleflow-workflow` nemaju zapis odluke.** `decorators/oscal/policy.py`
   (provjera prije konstrukcije) i `concrete/connection.py` (`__del__` na polusagrađenom objektu)
   izvedene su 2026-08-21; obje diraju distribuciju kojoj `CLAUDE.md` §2.5 traži DR. Kandidat:
   `DR-WFL-014`.
2. **Kategorija vs distribucija.** [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
   t.2 veže kategoriju `OSCAL` uz distribuciju `wattleflow-oscal`, koja više ne postoji; sve živi u
   `blackwattle`. Oznaka je zadržana jer je **sposobnost** ista i roditelj je `HLRQ-14`;
   proširenje osi traži dopunu tog zapisa.
3. **Vrata su i dalje inertna.** `strict=False` na sve tri baze znači da bez predane politike
   provjere nema. Prednost novog oblika: prelazak na strogi režim je izmjena **na tri mjesta**,
   ne na četrnaest. Odluka *tko predaje politiku* stoji: [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.2.
4. **18 drivera i 17 procesora ostaje izvan vrata.** Ne zna se deklariraju li premalo ili doista
   ne provode kontrole; razrješava empirijska provjera po komponenti
   ([`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.5, `BR-OSCAL-12`).
