# FRQ-PIP-15.2 — Generički pipeline

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog. **Izmijenjeno 2026-09-09** ([`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md), prijedlog): ovaj sloj više ne emitira `INFO` |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `PIP`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericPipeline(Wattleflow, IPipeline, ABC)` — jedna transformacija nad jednom stavkom; uz njega `PipelineError` |
| **Sestrinski** | [`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md) (poziva `process`) · [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) (prima rezultat) · [`FRQ-DOC-15.8`](FRQ-DOC-15.8-document.md) (`facade`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/pipeline.py` |

## 1. Predmet

Pipeline je **najmanja jedinica transformacije**: jedna stavka ulazi, jedna transformacija se
izvodi. Sve što je oko toga — koliko stavki ima, odakle dolaze, kada se rezultat trajno sprema —
ne pripada mu.

Klasa razdvaja dvije metode koje se lako pomiješaju:

| metoda | tko je piše | što radi |
|---|---|---|
| `transform(processor, facade, **kwargs)` | **specijalizacija** — apstraktna | sam posao |
| `process(processor, facade, **kwargs)` | **generički sloj** — konkretna | okvir oko posla |

`process` je omotač koji radi četiri stvari koje specijalizacija ne smije ponavljati: provjeri
tipove ulaza, otvori audit zapis, pozove `transform`, i **svaku** iznimku omota u `PipelineError`
s uzrokom. Time je `BR-15-09` ispunjen na jednom mjestu za sve pipelinee.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi pipeline iz konfiguracije | `level`, `handler`, preset ključevi |
| **A2** | `GenericProcessor` — poziva `process` po stavci | `processor`, `facade` |
| **A3** | `GenericPipeline` — predmet ovog zahtjeva | transformacija |
| **A4** | `GenericBlackboard` — odredište rezultata | prima `write` iz `transform` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira pipeline → `__init__` |
| **EV02** | A2 poziva `process(processor, facade)` za jednu stavku |
| **EV03** | `transform` vrati rezultat |
| **EV04** | `transform` ili provjera ulaza padne |
| **EV05** | kraj životnog ciklusa → `__del__` |

## 4. Preduvjeti

1. `processor` je `IProcessor`, `facade` je `ITarget` — provjereno `assert`-om u `process`.
2. Preset je izgrađen; konfiguracijska imena se razrješavaju kroz `__getattr__`.
3. Specijalizacija je implementirala `transform` — inače je klasa apstraktna i ne instancira se.

## 5. Normalan tok

1. **EV01** — konstruktor prosljeđuje `level` i `handler` naviše zajedno s cijelim `**kwargs`,
   prijavljuje `Constructor` na `DEBUG`, pa gradi `PresetDecorator`.
2. **EV02** — `process` prijavljuje `Transform/Starting` na `DEBUG`.
3. Provjera ulaza: `processor` mora biti `IProcessor`, `facade` mora biti `ITarget`.
4. **Zapis na ulazu**, na `DEBUG`: `Transform/Started` s imenom izvora i identifikatorom
   dokumenta. Otvara se **prije** posla namjerno — mjesto zapisa i njegova polja odgovaraju redu
   kojim se posao odvija (`DR-WFL-021` t.4). Razina je `DEBUG`, ne `INFO`, jer obradu dokumenta
   posjeduje **procesor**, a pipeline je korak unutar te jedinice: `INFO` po pipelineu množio bi
   zapise brojem konfiguriranih pipelinea umjesto brojem dokumenata
   ([`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md)).
   Zatvaranje jedinice — i po dokumentu i po prolazu — pripada procesoru i workflowu.
5. Ime izvora izvodi `NameHelper.source_name(facade)` — `facade.filename` sveden na osnovno ime
   putanje. Metoda **nikad ne diže**: dokument bez imena datoteke daje `None`. Stoji u
   `concrete/helpers.py` jer je dijele pipeline i procesor — helper dvaju pod-paketa iste domene
   ide u domenski-interni dijeljeni modul (`NFRQ-ORG-01`).
6. **EV03** — `transform` se izvede; `Transform/Completed` na `DEBUG` s rezultatom.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `processor` nije `IProcessor` ili `facade` nije `ITarget` | `AssertionError` → `PipelineError(str(e))` | kvar konfiguracije, ne podatka |
| `transform` digne bilo što | `PipelineError(caller=self, error=…)` s `from e` | trag kaže **koji** je pipeline pao (`BR-15-09`) |
| bilo koji kvar | ovaj sloj piše samo `DEBUG` trag i **prosljeđuje** | `ERROR` piše pozivatelj koji zaustavlja propagaciju (`DR-WFL-018` t.2) |
| `facade` nema `filename` | `NameHelper.source_name` vraća `None` | zapis nastaje i bez imena izvora |
| `__init__` padne prije `_preset` | `__del__` posegne za `_preset`, `__getattr__` digne `AttributeError` | **izvorna iznimka se maskira** — vidi §11 t.1 |

## 7. Rezultat

Jedna stavka je transformirana, rezultat je predan dalje (tipično na platno), a audit tok nosi
otvarajući zapis te transformacije na mjestu koje odgovara redoslijedu izvođenja — na `DEBUG`
razini. Na `INFO` razini ovaj sloj **šuti**; stavku prijavljuje njezin vlasnik, procesor. Kvar je
omotan u `PipelineError` s očuvanim uzrokom i ne stiže pozivatelju kao anonimna iznimka.

## 8. Kriteriji prihvaćanja

1. `transform` je apstraktna; `process` je konkretna i jedina ulazna točka za pozivatelja. ✅
2. Svaka iznimka iz `transform` izlazi kao `PipelineError` s očuvanim `__cause__`. ✅
3. Otvarajući zapis nastaje **prije** posla, na `DEBUG` razini (`DR-WFL-028`). ✅
   - **3a.** Ovaj sloj ne emitira nijedan `INFO` zapis (`DR-WFL-028` t.4). ✅
4. Ovaj sloj ne piše `ERROR` — piše `DEBUG` trag i prosljeđuje (`DR-WFL-018` t.2). ✅
5. `NameHelper.source_name` nikad ne diže iznimku. ✅
6. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
7. `__del__` preživi neuspjelu konstrukciju. ❌ — vidi §11 t.1
8. Klasa deklarira `__slots__`. ❌ — vidi §11 t.2

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled dekoratora i potpisa | `@abstractmethod def transform`; `def process` |
| 2 | pregled obiju `except` grana | `raise PipelineError(...) from e` u obje |
| 3 | pregled redoslijeda u `process` (2026-09-09) | `self.debug(msg=Transform, step=Started, …)` prethodi `self.transform(...)` |
| 3a | `command grep -n 'self\.info(' concrete/pipeline.py` (2026-09-09) | nijedna pojava |
| 4 | `command grep -n 'self.error' concrete/pipeline.py` | jedina pojava je u `__del__`, ne u `process` |
| 5 | pregled `NameHelper.source_name` u `concrete/helpers.py` (2026-09-09) | `try/except Exception: return None` |
| 6 | pregled modula (2026-09-09) | `__all__ = ["GenericPipeline", "PipelineError"]`; uvozi `logging`, `typing` + `wattleflow.*` — `pathlib` je otišao s `_source_name` |
| 7 | pregled `__del__` | `if self._preset is not None` — bez zaštite `object.__getattribute__` |
| 8 | `command grep -n '__slots__' concrete/pipeline.py` | nema pojave |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` radno stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/pipeline.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Pipeline` je rezervirani primitiv; transformacija se ne seli u procesor ni u strategiju |
| `NFRQ-OBS-01` | **nijedan** `INFO` iz ovog sloja; `DEBUG` nosi ulaz, rezultat i kvar (`DR-WFL-028`) |
| `NFRQ-OBS-02` | polja zapisa na ulazu su `msg`, `step`, `source`, `document` — nepromijenjena izmjenom razine |
| `NFRQ-OBS-03` | volumen na `INFO` je **nula**; stavka je jedinica posla **procesora**, ne pipelinea (`DR-WFL-028` t.1) |
| `NFRQ-SEC-02` | `__all__` izlaže dva imena; imenovanje izvora je preseljeno u `NameHelper` |
| `NFRQ-SEC-03` | clean core tier; nijedan third-party uvoz |
| `NFRQ-ORG-05` | `NameHelper.source_name` je `@staticmethod` i doista ne dira nijedan član klase — izbor dekoratora izražava namjeru |
| `NFRQ-ORG-01` | imenovanje izvora dijele pipeline i procesor, pa stoji u `concrete/helpers.py`, ne duplicirano u oba |

## 11. Otvoreno

1. **`__del__` ne preživi neuspjelu konstrukciju.** `if self._preset is not None` poziva
   `__getattr__` kad `_preset` nije postavljen, a `__getattr__` radi
   `object.__getattribute__(self, "_preset")` bez zaštite → `AttributeError` iz destruktora
   **maskira pravu iznimku**. `GenericBlackboard` isti slučaj rješava ranim `return`-om
   (`FRQ-BBD-15.1` §5 t.7), `GenericProcessor` cijeli `__del__` drži u `try/except`. Tri sestre,
   tri različita rješenja istog problema — kandidat za `NFRQ-ORG-08`.
2. **Nema `__slots__`.** Klasa drži `_preset` u `__dict__`, za razliku od `GenericBlackboard` i
   `GenericProcessor` koji ga drže u slotu. Pipeline se instancira jednom po workflowu pa je
   trošak malen, ali `HLRQ-15` §4 t.5 propisuje slotove za klasu koja drži stanje.
3. ~~**Docstring `_source_name` opisuje kod koji ne postoji.**~~ **Riješeno 2026-09-09.** Metoda
   je preseljena u `NameHelper.source_name` (`concrete/helpers.py`) i docstring je prepisan: razlog
   za „nikad ne diže" sada glasi da se izvodi unutar audit zapisa, gdje bi iznimka maskirala
   događaj koji se prijavljuje. Poziv na nepostojeći `finally` blok je uklonjen.
4. **Zakomentiran `trace=traceback.format_exc()`** u drugoj `except` grani. Ili trag pripada
   iznimci (pa `NFRQ-SEC-06` mora reći što se iz njega redigira), ili ne pripada (pa se briše).
   Zakomentirana linija ne odlučuje ni jedno.
5. **`PipelineError` i `PipelineException` postoje usporedo.** Prvi je definiran ovdje
   (`AuditException`), drugi u `concrete/exception.py` i njega diže `GenericProcessor` kad
   pipeline padne. Dva razreda za istu obitelj kvara — razlika je stvarna (sloj koji ga diže),
   ali nigdje nije zapisana.
