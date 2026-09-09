# FRQ-PRC-15.3 — Generički procesor

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog. **Izmijenjeno 2026-09-09** ([`DR-WFL-028`](../04-DR/DR-WFL-028-per-document-confirmation-belongs-to-the-processor.md), prijedlog): procesor piše i granicu **po dokumentu** |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `PRC`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericProcessor(Wattleflow, IProcessor, IOriginator, ABC)` — vlasnik prolaza nad skupom stavki; uz njega `ProcessorState`, `ProcessorAction`, `TRANSITIONS` |
| **Sestrinski** | [`FRQ-PIP-15.2`](FRQ-PIP-15.2-pipeline.md) (poziva se po stavci) · [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) (flush na granici ciklusa) · [`FRQ-MEM-15.10`](FRQ-MEM-15.10-memento.md) (snimka stanja) |
| **Izvedba** | `workflow/src/wattleflow/concrete/processor.py` |

## 1. Predmet

Procesor je **vlasnik prolaza**: on zna koliko stavki ima, dovodi ih jednu po jednu, provlači
svaku kroz sve pipelinee i odlučuje kada se platno prazni. On je i **jedina** klasa ovog sloja
koja je istodobno `IProcessor` i `IOriginator` — dakle jedina koja svoje stanje zna spremiti u
snimku i iz nje se nastaviti.

| član | uloga |
|---|---|
| `_generator` | izvor stavki; gradi ga apstraktni `create_generator()` |
| `_pipelines` | redoslijed transformacija kroz koje svaka stavka prolazi |
| `_blackboard` | platno na koje pipelinei pišu |
| `_cycle` | broj dovršenih stavki — **jedina** vrijednost koja ulazi u snimku uz stanje |
| `_fsm` | automat stanja; ovdje je u **generičkoj** klasi, ne u specijalizaciji |
| `_flush_per_cycle` | prazni li se platno nakon svake stavke ili tek na kraju |

Specijalizacija piše **jednu** metodu: `create_generator()`. Sve ostalo — petlja, automat, audit,
granice kvara, snimka — nasljeđuje.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `WorkflowFactory` — gradi procesor iz konfiguracije | `blackboard`, `pipelines`, `flush_per_cycle` |
| **A2** | `GenericWorkflow` / `Orchestrator` — pokreće prolaz | `Operation.Start` |
| **A3** | `GenericProcessor` — predmet ovog zahtjeva | ciklus i stanje |
| **A4** | `GenericPipeline` — prima svaku stavku | `process(processor, facade)` |
| **A5** | `GenericBlackboard` — prima `flush` | granica ciklusa |
| **A6** | `GenericMemento` — nosi snimku | `cycle` + stanje automata |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instancira procesor → `__init__` |
| **EV02** | pipeline ili blackboard se pridružuje → `register_pipeline` / `register_blackboard` |
| **EV03** | A2 traži pokretanje → `operation(Operation.Start)` → `start()` |
| **EV04** | generator daje sljedeću stavku → `NEXT_ITEM` |
| **EV05** | stavka prošla sve pipelinee → `CYCLE_COMPLETED` |
| **EV06** | generator iscrpljen → `RECORDS_PROCESSED` |
| **EV07** | kvar u prolazu → `FAIL` |
| **EV08** | nastavak iz snimke → `restore_state(memento)` → `LOAD` |
| **EV09** | kraj životnog ciklusa → `__del__` |

## 4. Preduvjeti

1. Blackboard je registriran — inače `start()` odbija raditi (`BR-15-02`).
2. Najmanje jedan pipeline je registriran — isto.
3. Specijalizacija je implementirala `create_generator()`.
4. Za nastavak: snimka nosi stanje iz kojeg je `LOAD` dopušten (`IDLE` ili `FAILED`).

## 5. Normalan tok

1. **EV01** — konstruktor izdvaja `blackboard`, `pipelines`, `flush_per_cycle` iz `**kwargs`, pa
   ostatak prosljeđuje naviše. Zastarjeli `defer_flush` se prihvaća i **preslikava** u
   `flush_per_cycle = not defer_flush`, uz `warning` koji imenuje zamjenu.
2. Automat se gradi ovdje, u generičkoj klasi: `StateMachine(TRANSITIONS, ProcessorState.IDLE,
   name="ProcessorFSM")`.
3. **EV03** — `operation(Operation.Start)` je jedina podržana operacija; svaka druga daje
   `warning` i `False` umjesto iznimke.
4. `start()` prvo provjerava blackboard pa pipelinee. Tek **nakon** provjera piše otvarajući
   `INFO` — zapis tako imenuje ono što će stvarno raditi, a ne ono što je zatraženo. Konfiguracija
   (tipovi blackboarda, pipelinea i spremišta) ide u **otvarajući** zapis, jer je operateru
   korisna dok još može djelovati; imena idu kao spojeni string, ne lista, jer audit renderer
   kolekciju na `INFO` sažima u `<list: N>` i sakrio bi upravo ta imena.
5. Generator se gradi ako ne postoji; `START` prevodi automat u `RUNNING`.
6. **EV04–EV05** — za svaku stavku: `NEXT_ITEM`, pa redom svi pipelinei, pa `_cycle += 1`,
   `CYCLE_COMPLETED`, i — ako je `flush_per_cycle` — `blackboard.flush(caller=self,
   **self.write_context)`. Ključeve koje `write_context` objavljuje čitaju write strategije;
   generička implementacija vraća prazan rječnik, pa procesor koji ništa ne deklarira ne mijenja
   nijedan poziv (vidi §11 t.6).
7. **Granica dokumenta** — tek **iza** flusha procesor piše `INFO` s `msg=Event.Processed`, bez
   `step`-a, s poljima `cycle`, `source` i `document`. Dokument je jedinica posla, a procesor
   njezin vlasnik: on dovodi stavku, provlači je kroz **sve** pipelinee i određuje granicu flusha
   (`DR-WFL-028` t.1–t.3). Zapis stoji iza flusha da uz `flush_per_cycle=True` potvrdi i
   transformaciju i pohranu; uz `flush_per_cycle=False` potvrđuje samo transformaciju — deklarirano
   ograničenje, vidi §11 t.7.
8. **EV06** — `RECORDS_PROCESSED` prevodi automat u `COMPLETED`.
9. Zatvarajući `INFO` nosi **samo ishod** (`cycles`), pod `msg=Event.Completed`. Otvarajući i
   zatvarajući zapis razlikuju se **imenom zapisa**, ne poljem `step` — inače operater mora
   čitati polje da bi znao koji od dva zapisa gleda.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| nema blackboarda ili pipelinea | `ProcessorException` prije otvarajućeg zapisa | prolaz ne počinje (`BR-15-02`) |
| pipeline padne nad stavkom | `PipelineException(caller=self, …) from e` | prolaz staje; uzrok očuvan |
| `PipelineException` u vanjskom `except` | **ponovno diže se nepromijenjen** | ne omata se dvaput |
| bilo koji drugi kvar | `FAIL` (ako ga automat dopušta) → `ProcessorException(...) from e` | stanje je `FAILED`, dakle nastavljivo (`BR-15-06`) |
| nepodržana operacija | `warning` + `False` | pogrešna konfiguracija ne ruši prolaz |
| snimka iz stanja iz kojeg `LOAD` nije dopušten | `ProcessorException` **prije** izmjene stanja | automat se ne kvari polovičnim vraćanjem |
| snimka traži više ciklusa nego skup ima | `StopIteration` → `ProcessorException("dataset shorter than saved cycle")` | nastavak nad promijenjenim skupom pada glasno |
| iznimka u `__del__` | `error(...)`, pa `finally: gc.collect()` | destruktor ne diže (`HLRQ-15` §4 t.4) |

## 7. Rezultat

Skup stavki je prošao kroz sve pipelinee, platno je ispražnjeno na granici koju određuje
`flush_per_cycle`, a audit tok nosi **`2 + N`** `INFO` zapisa: otvarajući s konfiguracijom, po
jedan za svaki od *N* dovršenih dokumenata, i zatvarajući s brojem ciklusa. Broj konfiguriranih
pipelinea u toj formuli ne sudjeluje. Prekinuti prolaz ostaje u stanju `FAILED` iz kojeg vodi
nastavak preko snimke; dokument na kojem je stao **nema** svoj zapis, pa se neuspjeh čita po
odsutnosti (`DR-WFL-021` t.6).

## 8. Kriteriji prihvaćanja

1. `create_generator()` je jedina apstraktna metoda. ✅
2. Automat se gradi u generičkoj klasi i primjenjuje na svaki prijelaz. ✅
3. `start()` provjerava preduvjete **prije** otvarajućeg zapisa. ✅
4. `2 + N` `INFO` zapisa po prolazu (`N` = dovršeni dokumenti); sva tri oblika razlikuju se
   `msg`-om, nijedan ne nosi `step` (`NFRQ-OBS-03` k.1, `DR-WFL-028` t.2). ✅
5. `restore_state` provjerava dopuštenost `LOAD`-a prije nego dira stanje. ✅
6. `PipelineException` se ne omata dvaput. ✅
7. Zastarjeli `defer_flush` radi i imenuje zamjenu. ✅
8. `__del__` ne diže iznimku ni u jednoj grani. ✅
9. Snimka nosi dovoljno za nastavak nad **istim** skupom. ⚠ — vidi §11 t.1

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | `command grep -n '@abstractmethod' concrete/processor.py` | jedna pojava — `create_generator` |
| 2 | pregled `__init__` i `start` | `StateMachine(TRANSITIONS, …)`; `apply` na `START`, `NEXT_ITEM`, `CYCLE_COMPLETED`, `RECORDS_PROCESSED`, `FAIL` |
| 3 | pregled redoslijeda u `start` | dvije `raise` grane prethode `self.info` |
| 4 | `command grep -n 'self\.info(' concrete/processor.py` (2026-09-09) | tri poziva: `msg=Event.Start` (317), `msg=Event.Processed` u petlji (350), `msg=Event.Completed` (391); nijedan ne nosi `step` |
| 5 | pregled `restore_state` | `if (saved_state, ProcessorAction.LOAD) not in TRANSITIONS: raise` prije `self._fsm.state = …` |
| 6 | pregled `except` grana | `except PipelineException: raise` prethodi općem `except Exception` |
| 7 | pregled `__init__` | preslikavanje + `self.warning(deprecated="defer_flush", use="flush_per_cycle")` |
| 8 | pregled `__del__` | vanjski `try/except`, unutarnji oko `blackboard.clean()`, `finally: gc.collect()` |
| 9 | pregled `save_state` | snimka nosi `cycle` i `state`; nastavak premotava generator `next()`-om |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` radno stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/processor.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Processor` je rezervirani primitiv; on posjeduje ciklus, a pipeline transformaciju |
| `NFRQ-OBS-01` | tri `INFO` točke: dvije granice prolaza i jedna granica dokumenta; sve ostalo `DEBUG`, osim deprecation `warning` |
| `NFRQ-OBS-02` | otvarajući zapis nosi `blackboard`, `pipelines`, `repositories`; po-dokumentu `cycle`, `source`, `document`; zatvarajući samo `cycles` |
| `NFRQ-OBS-03` | volumen je `2 + N` — procesor posjeduje **dvije ugniježđene** jedinice, prolaz i dokument (`DR-WFL-028`) |
| `NFRQ-SEC-01` | procesor ne poznaje spremišta osim kroz platno; `_repository_names` je samo za zapis i nikad ne diže |
| `NFRQ-SEC-03` | clean core tier; uvozi samo `gc`, `abc`, `enum`, `typing`, `collections.abc` + `wattleflow.*` |
| `NFRQ-ORG-08` | petlja, automat i granice kvara žive ovdje, ne prepisane po specijalizacijama |

## 11. Otvoreno

1. **Nastavak pretpostavlja nepromijenjen i determinističan skup.** `restore_state` premotava
   generator pozivom `next()` točno `cycle` puta — dakle vjeruje da isti skup daje iste stavke u
   istom redoslijedu. Za datotečni izvor s promijenjenim sadržajem ili za izvor bez zajamčenog
   redoslijeda nastavak tiho obrađuje **druge** stavke. Snimka nosi broj, ne identitet posljednje
   obrađene stavke. Kandidat: u snimku dodati identifikator, pa premotavanje provjeriti umjesto
   pretpostaviti.
2. **Premotavanje je linearno.** Nastavak od ciklusa *n* izvodi *n* poziva generatora prije
   prvog korisnog posla; za velike skupove to je cijena koja raste s napretkom. Nije mjereno —
   uočeno.
3. **`flush_per_cycle` je zadano `True`, a `GenericBlackboard` postavlja `defer_flush=True`**
   (dakle „ne prazni po ciklusu"). Dvije generičke klase istog toka nose **suprotan** zadani
   izbor, a blackboard uz to pali deprecation upozorenje u procesoru pri svakoj konstrukciji
   (`FRQ-BBD-15.1` §11 t.5). Uskladiti — jedna od dvije zadane vrijednosti je pogrešna.
4. **`Operation` podržava samo `Start`.** Zaustavljanje, pauza i nastavak nisu operacije nego
   posljedice iznimke, iako automat ima stanja koja bi ih podnijela. Nije kvar — nezapisano
   ograničenje.
5. **`gc.collect()` u `__del__`.** Prisilno sakupljanje pri svakom uništenju procesora je
   mjerljiv trošak koji nijedan zahtjev ne traži. Ako postoji ciklička referenca koja to
   opravdava, treba je imenovati; ako ne postoji, poziv se briše.
6. **`write_context` nema nijednog implementatora.** Svojstvo postoji u `GenericProcessor`,
   `FRQ-PRC-15.22` EV08 ga opisuje kao put kojim procesor objavljuje konfiguraciju write
   strategijama, a `blackboard.flush(**self.write_context)` ga doista prosljeđuje. Provjereno
   2026-09-09 (`command grep -rn 'write_context'` nad `workflow` i `blackwattle`): **nijedna
   podklasa ga ne nadjačava**, pa je proslijeđeni rječnik uvijek prazan. Mehanizam je time
   dokumentiran i pozvan, ali nikad izvršen — dakle nedokazan (D-05). Uočeno preko konfiguracije
   koja je `skip_*` ključeve stavila na procesor očekujući upravo taj put (`workflow/TODO.md`).
7. **Granica dokumenta ne razlikuje „obrađen" od „pohranjen".** Uz `flush_per_cycle=False` zapis
   `Processed` nastaje dok je dokument još na platnu, pa tvrdi manje nego što mu ime sugerira.
   Rascjep na dva zapisa udvostručuje volumen; vezanje uz flush gubi vezu s pojedinim dokumentom.
   Deklarirano u `DR-WFL-028` §Cijena, neodlučeno.
