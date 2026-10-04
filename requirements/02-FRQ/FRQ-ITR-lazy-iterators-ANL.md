# Analiza: ponovna iteracija `LazyIterator` (`concrete/iterator.py`)

| | |
|---|---|
| **Datum** | 2026-10-04 |
| **Vrsta** | analiza (nije odluka) |
| **Povod** | [`FRQ-ITR`](FRQ-ITR-lazy-iterators.md) `DEF-ITR-01`: „nema ponovne iteracije” |
| **Trojka (D-10)** | alat: `unittest`, `timeit` i usporedba četiriju izvedbi · kriterij: [`FRQ-ITR`](FRQ-ITR-lazy-iterators.md) odjeljak 13 · platforma: CPython 3.12.14, Linux/WSL2, okruženje `workflow`, dokumentacija `v0.0.5`, kod `v0.0.1.23` |
| **Zahtjev** | [`FRQ-ITR`](FRQ-ITR-lazy-iterators.md) |

## 1. Sažetak

- **Zaključak:** ponašanje „nema ponovne iteracije” nije defekt nego svojstvo iteratora: `LazyIterator` je jednokratan (`iter(x) is x`), a novi prolaz daje agregat koji stvara **novi** iterator. Taj obrazac kod nas već postoji: `Catalog.create_iterator()` pri svakom pozivu vraća novi `ControlIterator` (provjereno na pravom `Catalog`-u).
- **Alternative** koje bi mijenjale `LazyIterator` lome protokol iteratora (`iter(x)` više nije `x`, ili `next(iter(x))` ne napreduje) i mijenjaju rezultat standardnih konstrukcija (`zip(x, x)`, `islice`, nastavak nakon `break`). `restart()` ne mijenja ništa od toga, ali ga nitko ne traži.
- **Preporuka:** kod ostaje; ugovor se zapisuje u dokumentu (jednokratan, novi prolaz iz agregata); ponašanje je sada zaključano testovima.
- **Trošak:** `LazyIterator` košta oko 70 ns po elementu, a običan iterator oko 12 ns, dakle oko 58 ns po elementu zbog `__next__` na razini Pythona.
- **Utjecaj na postojeći kod:** `LazyIterator` nasljeđuje jedna klasa (`ControlIterator`, `blackwattle/oscal/models.py`), a koristi je jedan poziv (`Catalog.create_iterator`); `LazyAsyncIterator` ne nasljeđuje nijedna.
- **Testovi:** 21 test ugovora i 4 testa troška u `workflow`, 5 u `blackwattle`; četiri mutacije uhvaćene.

## 2. Metoda i skripte

| što | gdje |
|---|---|
| testovi ugovora (sinkroni i asinkroni) | [`test_lazy_iterator.py`](../../../workflow/tests/test_lazy_iterator.py) (21 test) |
| test agregata s pravim `Catalog`-om | [`test_control_iterator.py`](../../../blackwattle/tests/oscal/test_control_iterator.py) (5 testova) |
| trošak svake metode (`__init__`, `__next__`, `__anext__`) | [`test_lazy_iterator_cost.py`](../../../workflow/tests/test_lazy_iterator_cost.py) (4 testa, oko 0,5 s) |
| mjerenje prijedloga (utrka dretvi, trošak, provjera tipa) | [`2026-10-04-iterators-analysis-proposal.py`](../06-ANALYSIS/2026-10-04-iterators-analysis-proposal.py) (nad kandidatskim razredima; kod u repozitoriju nije mijenjan) |
| usporedba izvedbi i trošak po elementu | [`2026-10-04-iterators-analysis-compare.py`](../06-ANALYSIS/2026-10-04-iterators-analysis-compare.py) |

Usporedba izvodi iste scenarije nad četiri izvedbe: **V0** sadašnja; **V1** `__iter__` vraća novi izvor pri svakom pozivu; **V2** `__iter__` premotava
izvor i vraća `self`; **V3** sadašnja uz izričiti `restart()`. Izvor ima 6 elemenata, a „izvora” je broj poziva `create_iterator()`. Trošak: 100 000 elemenata,
medijan pet ponavljanja po izvedbi, iznosi variraju nekoliko posto. Ponavljanje: `cd workflow && PYTHONPATH=src:../core/src python ../documentation/requirements/06-ANALYSIS/2026-10-04-iterators-analysis-compare.py`.

## 3. Ponašanje po scenarijima

| scenarij | V0 sada | V1 __iter__ nov izvor | V2 __iter__ premotava | V3 restart() |
|---|---|---|---|---|
| list(x) dva puta | [[1, 2, 3, 4, 5, 6], []] (izvora: 1) | [[1, 2, 3, 4, 5, 6], [1, 2, 3, 4, 5, 6]] (izvora: 2) | [[1, 2, 3, 4, 5, 6], [1, 2, 3, 4, 5, 6]] (izvora: 2) | [[1, 2, 3, 4, 5, 6], []] (izvora: 1) |
| for do 2. pa list(x) | [3, 4, 5, 6] (izvora: 1) | [1, 2, 3, 4, 5, 6] (izvora: 2) | [1, 2, 3, 4, 5, 6] (izvora: 2) | [3, 4, 5, 6] (izvora: 1) |
| 3 x next(iter(x)) | [1, 2, 3] (izvora: 1) | [1, 1, 1] (izvora: 3) | [1, 1, 1] (izvora: 3) | [1, 2, 3] (izvora: 1) |
| zip(x, x) | [(1, 2), (3, 4), (5, 6)] (izvora: 1) | [(1, 1), (2, 2), (3, 3), (4, 4), (5, 5), (6, 6)] (izvora: 2) | [(1, 2), (3, 4), (5, 6)] (izvora: 1) | [(1, 2), (3, 4), (5, 6)] (izvora: 1) |
| islice(x,2) dva puta | [[1, 2], [3, 4]] (izvora: 1) | [[1, 2], [1, 2]] (izvora: 2) | [[1, 2], [1, 2]] (izvora: 2) | [[1, 2], [3, 4]] (izvora: 1) |
| iter(x) is x | True (izvora: 0) | False (izvora: 1) | True (izvora: 0) | True (izvora: 0) |
| restart() pa list(x) | nema restart() (izvora: 0) | nema restart() (izvora: 0) | nema restart() (izvora: 0) | [1, 2, 3, 4, 5, 6] (izvora: 2) |

Agregat (obrazac `Catalog`): dva prolaza iz dva poziva `create_iterator()` daju `[[1, 2, 3, 4, 5, 6], [1, 2, 3, 4, 5, 6]] (izvora: 2)`.

Standardno ponašanje iteratora (V0, V3) daje `[1, 2]` zatim `[3, 4]` za dva `islice`, parove susjeda za `zip(x, x)` i napredak pri `next(iter(x))`.
V1 i V2 ponavljaju početak, pa bi kod koji se oslanja na te konstrukcije dobio drukčije rezultate bez greške.

## 4. Trošak

Medijan po elementu u nanosekundama (100 000 elemenata): V0 sada 69.8, V1 __iter__ nov izvor 10.5, V2 __iter__ premotava 68.3, V3 restart() 72.4 · običan iterator 12.2.

- V0 i V3 koriste isti put (`restart()` je samo dodatna metoda), razlika među njima je šum mjerenja.
- V1 je „brz” samo zato što `__iter__` vraća izvorni generator i zaobilazi `__next__`; time nestaje i sama klasa kao iterator.
- Trošak sadašnje izvedbe je oko 58 ns po elementu više od običnog iteratora (poziv `__next__` u Pythonu, provjera `is None`, `next`). To je cijena odgode i ne treba je mijenjati dok nitko ne iterira milijune elemenata.

## 5. Utjecaj na postojeći kod

| mjesto | utjecaj |
|---|---|
| `ControlIterator(LazyIterator[Control])` (`blackwattle/src/wattleflow/oscal/models.py`) | jedina podklasa; `create_iterator()` vraća `catalog.walk_controls()`; ostaje jednokratan |
| `Catalog.create_iterator()` | jedini izvor `ControlIterator`-a; svaki poziv daje novi iterator, pa su dva prolaza ispravna (provjereno: `['a', 'b', 'b1']` dvaput) |
| `LazyAsyncIterator` | nema podklasa ni poziva u `src`; ponaša se jednako kao sinkroni parnjak |
| ostali `IIterator` | `IIterator` ne propisuje politiku konstrukcije; ostale izvedbe nisu dirnute |

## 6. Utjecaj izmjena

| izmjena | posljedica | ocjena |
|---|---|---|
| V1 `__iter__` daje novi izvor | `iter(x) is x` postaje `False`; `zip(x, x)` daje `(1, 1), (2, 2)…`; `next(iter(x))` ne napreduje; `__next__` se zaobilazi | **ne**: lomi protokol |
| V2 `__iter__` premotava | `next(iter(x))` uvijek vraća prvi element; dva `islice` ponavljaju početak; nastavak nakon `break` kreće ispočetka | **ne**: tihi drukčiji rezultati |
| V3 `restart()` | ništa se ne mijenja; dodaje se jedna javna metoda bez korisnika | moguće, ali nepotrebno: agregat već daje novi prolaz |
| bez izmjene, ugovor zapisan i zaključan testovima | nema rizika | **preporuka** |

## 7. Testovi i mutacije

Ugovor koji testovi zaključavaju: izvor se ne gradi u konstruktoru; gradi se jednom pri prvom dohvatu; `iter(x) is x`; iscrpljen iterator ostaje iscrpljen; nastavak nakon `break`;
`zip(x, x)` i `islice` kao kod običnog iteratora; kraj izvora prolazi nepromijenjen (`StopIteration`, `StopAsyncIteration`); pad `create_iterator()` se širi, a sljedeći dohvat pokušava ponovno;
`__slots__ == ("_iterator",)`; agregat daje novi iterator po pozivu; isto za asinkrono.

Mutacije nad kopijom `iterator.py`: **`eager`** (gradi u konstruktoru) ruši 3 testa; **`fresh`** (V1) ruši 7; **`rewind`** (V2) ruši 4; **`every`** (gradi pri svakom dohvatu) uzrokuje beskonačnu petlju u `list(iterator)`, što je uhvaćeno prekidom. Izvorni kod je neizmijenjen.

## 8. Matrica uporabe

Statički (AST/`grep` nad `workflow/src` i `blackwattle/src`), stanje 2026-10-04:

| metoda | korisnici u `src` | testovi |
|---|---|---|
| `LazyIterator.__init__` | `ControlIterator.__init__` kroz `super().__init__()` (`oscal/models.py:377`) | `test_lazy_iterator.py`, `test_control_iterator.py` |
| `LazyIterator.__next__` | iterira se `ControlIterator`: `for control in catalog.create_iterator()` (`oscal/registry.py:57`) | oba testa |
| `LazyIterator.__iter__` (naslijeđen) | isti `for` | `test_lazy_iterator.py` (`iter(x) is x`) |
| `LazyAsyncIterator.__init__`, `__anext__`, `__aiter__` | nitko | `test_lazy_iterator.py` (`AsyncLazyIteratorTest`) |

Jedan `ISyncAggregate` (`Catalog.create_iterator`) i jedan potrošač (`registry.py`). Asinkrona klasa nema korisnika u `src`; dinamički pozivi nisu obuhvaćeni.

**Uporaba iteracije u cijelom frameworku** (AST nad 394 datoteke u `core`, `workflow` i `blackwattle`, 2026-10-04):

| mehanizam | gdje | tko ga koristi | broj |
|---|---|---|---|
| `IIterator`, `IAsyncIterator` (sučelja) | `core/behavioural.py` | `LazyIterator`, `LazyAsyncIterator`; podklasa `ControlIterator` | 2 sučelja, 3 razreda |
| `ISyncAggregate`, `IAsyncAggregate` (sučelja) | `core/behavioural.py` | `Catalog` (sinkroni); asinkroni nema implementacije | 1 implementacija |
| `LazyIterator` / `LazyAsyncIterator` | `concrete/iterator.py` | `ControlIterator`; asinkroni nitko | 1 podklasa, 1 potrošač (`oscal/registry.py:57`) |
| `IProcessor.create_generator()` | `core/transactional.py` | `GenericProcessor` drži `_generator` i napreduje ga s `next(self._generator)` (`processor.py:277`); 19 implementacija `create_generator` u `blackwattle/processors` (+2 `_iter_paths`) | **alternativni mehanizam**: običan generator, ne `IIterator` |
| generatori `search` u driverima | `blackwattle/drivers` | 12 (+ `_stream`, `open_target`) | običan generator |
| `walk_controls` | `blackwattle/oscal/models.py` | 3 generatora koji hrane `ControlIterator` | običan generator |
| generatori `@contextmanager` (`connect` 14, `reader`, `_tika_client`) | `connections`, `converters`, `strategies` | nisu iteratori elemenata | isključeno iz brojanja iteratora |
| vlastiti `__iter__` | `FileSourceScanner` (`workflow/helpers/files.py`), `MailMessage` (`blackwattle/converters/mail/parsers.py`) | ne nasljeđuju `IIterator` | 2 |
| izravni `iter()` i `next()` | `workflow` 10 poziva, `blackwattle` 10 | većinom `next(generatorski izraz)` za prvi pogodak (`monitor`, `exception`, `workflow`, `manager`) | 20 poziva |

Zaključak o uporabi: `IIterator` i `LazyIterator` u praksi gotovo nitko ne koristi (jedna podklasa, jedan potrošač). Framework iterira stavke običnim generatorima (`create_generator`, `search`, `walk_controls`).
Izmjena `LazyIterator`-a zato je niskog utjecaja, ali ponašanje koje bi se pritom mijenjalo mora ostati usklađeno s tim generatorima, koji su jednokratni kao i `LazyIterator`.

## 9. Opis funkcionalnosti metoda

Trajanje je medijan jednog poziva u nanosekundama ([`test_lazy_iterator_cost.py`](../../../workflow/tests/test_lazy_iterator_cost.py); CPython 3.12.14, Linux/WSL2; iznosi variraju nekoliko posto).

| metoda | svrha | što postiže | trajanje (ns) |
|---|---|---|---:|
| `LazyIterator.__init__` | odgođena gradnja izvora | poziva korijen (`Wattleflow`: audit, preset) i postavlja `_iterator = None`; izvor se ne gradi | 1635 |
| `LazyIterator.__next__`, prvi dohvat | izgradnja izvora na zahtjev | poziva `create_iterator()`, zadržava rezultat i vraća prvi element (iznos uključuje konstrukciju; sama gradnja je razlika prema `__init__`) | 1898 |
| `LazyIterator.__next__`, sljedeći dohvat | dohvat elementa | provjerava `_iterator is None` i delegira `next`; kraj izvora prolazi kao `StopIteration` | 119 |
| `LazyAsyncIterator.__init__` | isto za asinkroni izvor | kao sinkroni | 1538 |
| `LazyAsyncIterator.__anext__`, prvi dohvat | izgradnja asinkronog izvora na zahtjev | poziva `create_iterator()`, zadržava rezultat, `await` prvi element (uključuje konstrukciju) | 7393 |
| `LazyAsyncIterator.__anext__`, sljedeći dohvat | dohvat elementa | provjera i `await __anext__` zadržanog izvora; kraj prolazi kao `StopAsyncIteration` | 271 |

Pri iteraciji `for` petljom sinkroni trošak je oko 70 ns po elementu (§4), jer petlja zaobilazi poziv funkcije `next()`; izravan `next(x)` košta oko 120 ns.

## 10. Za razgovor

- Treba li ikada ponovna iteracija **na istoj instanci**? Ako da, jedino sigurno proširenje je izričiti `restart()` (V3); `__iter__` se ne smije dirati.
- Trošak od oko 58 ns po elementu: prihvaćen; optimizirati tek uz izmjereni pritisak.

## 11. Prijedlog rješenja uklopljenog u postojeće klase

**Stanje 2026-10-04:** `DEF-ITR-03` i `DEF-ITR-04` primijenjeni u `concrete/iterator.py` (`_build()` u oba razreda); `DEF-ITR-02` primijenjen kao `ThreadSafeLazyIterator` (potvrđeno: 8 dretvi, izvor se gradi jednom). Svi defekti zatvoreni. Retci tablice za 03 i 04 ostaju kao zapis o odluci.

Preporuka za `DEF-ITR-02`, `-03` i `-04`: koristiti mehanizme koje framework već ima, bez novih koncepata. Stanje se mjeri nad kandidatskim razredima
([skripta](../06-ANALYSIS/2026-10-04-iterators-analysis-proposal.py)); kod u repozitoriju nije mijenjan.

| defekt | rješenje | uzor u kodu | izmjereno |
|---|---|---|---|
| `DEF-ITR-04` izvor se ne provjerava | `Attribute.evaluate(self, source, Iterator)` odnosno `AsyncIterator` nakon `create_iterator()`, u oba razreda | isti poziv koristi framework 65 puta (`workflow.py`, 22 datoteke u `blackwattle`); `AttributeException` imenuje pozivatelja | asinkroni razred sa sinkronim izvorom diže sirovi `AttributeError`, kandidat `AttributeException` |
| `DEF-ITR-03` gradnja nije u auditu | zajednička privatna `_build()`: `self.debug(msg=Event.Create, step=Event.Started)`, zatim `Completed`, a pri padu `Failed` s `error=str(e)` i ponovno dizanje | `GenericConnection.ensure_created` bilježi `Event.Create` / `Event.Failed` na `DEBUG` (`NFRQ-OBS-01`); događaji već postoje u `Event` | samo prvi dohvat: ≈ 2347 ns uz gradnju naspram ≈ 297 ns sada; stalni dohvat ≈ 109 ns, nepromijenjen (123 ns) |
| `DEF-ITR-02` nije nit-siguran | **zasebna** klasa `ThreadSafeLazyIterator(LazyIterator)` s brojilom `_build_lock` i dvostrukom provjerom (`if self._iterator is None:` izvan brave, zatim unutar nje) | `ThreadSafeObservable` (`concrete/observable.py`): isti naziv prefiksa, `__slots__` s bravom; osnovni razred ostaje bez brave | 8 dretvi, 20 krugova: sadašnja izvedba gradi izvor **8 puta po krugu** (u svih 20 krugova), kandidat **1 put**; stalni dohvat ≈ 109 ns |

Dva detalja koja se moraju poštovati:

1. **Slot se ne smije zvati `_lock`.** `Audit` ima klasni `_lock`, a `Scheduler` u komentaru bilježi da slot istog imena zasjenjuje klasni i mora postojati prije poziva osnovnog konstruktora. Zato `_build_lock` (kao `_emit_lock` u `Orchestrator`), stvoren prije `super().__init__()`.
2. **Opseg sigurnosti je samo gradnja izvora.** Istodobni `next()` nad istim generatorom Python sam odbija (`ValueError: generator already executing`). `LazyAsyncIterator` brave ne treba: nema `await` između provjere `_iterator is None` i dodjele, pa korutine na istoj petlji ne mogu proći utrku; istodobni `__anext__` nad istim asinkronim generatorom Python odbija s `RuntimeError`.

Gdje bi to bilo: `workflow/src/wattleflow/concrete/iterator.py` (`_build()` u oba razreda, nova klasa i `__all__`), bez izmjena u `core`; testovi bi se dodali u `test_lazy_iterator.py` (utrka s `threading.Barrier`, provjera tipa, događaji preko `assertLogs`), a kriteriji u `FRQ-ITR`.

Utjecaj na postojeći kod: `ControlIterator` dobiva dva retka audita na `DEBUG` i provjeru tipa koju `walk_controls()` (generator) zadovoljava; ponašanje se ne mijenja.
Izvor koji nije iterator (npr. popis) i danas padne pri prvom `next()` s `TypeError`; provjera taj pad samo premješta ranije i imenuje pozivatelja.

**Preporuka:** primijeniti prva dva (zatvaraju `DEF-ITR-04` i `-03`, mali trošak, samo prvi dohvat). Treće (`ThreadSafeLazyIterator`) dodati tek kad se pojavi potrošač s više dretvi; danas postoji jedan (`registry.py`) i jednodretven je.
