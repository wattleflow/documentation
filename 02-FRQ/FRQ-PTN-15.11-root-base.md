# FRQ-PTN-15.11 — Korijenska baza `Wattleflow`

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) t.3 — kategorija `PTN` |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — `BR-15-03`, `BR-15-04`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `Wattleflow(Audit, IWattleflow)` — kanonski korijen svakog objekta frameworka |
| **Sestrinski** | svi zapisi `FRQ-*-15.*` — svaki od njih nasljeđuje ovu klasu prvu |
| **Izvedba** | `workflow/src/wattleflow/concrete/base.py` (69 linija) |

## 1. Predmet

Klasa ima **jednu naredbu u tijelu** i najveći domet u sloju. Njezino tijelo je
`super().__init__(**kwargs)`; sve ostalo je ono što njezino postojanje **jamči**:

1. **Identitet** se izvodi iz konkretnog tipa pri pristupu, pa je nepromjenjiv: nijedan put u kodu
   ne može objekt preimenovati nakon konstrukcije. Time audit zapisi temeljeni na `__str__`
   ostaju otporni na krivotvorenje (`DR-COR-002`, `BR-15-03`).
2. **Auditabilnost nije opcija.** `Audit` se nasljeđuje **ovdje**, na jednom deklariranom mjestu,
   pa je svaki potomak nosi **po ugovoru** — a ne tako što je usput pokupi kao nuspojavu neke
   druge baze (`DR-WFL-009`, `BR-15-04`).
3. **Podjela `**kwargs` je na jednom mjestu.** `__init__` otvara kooperativni lanac i jedini
   razdvaja logging ključeve od ostalih: logging ide `Audit`-u, ostatak staje ovdje. Podklasa
   prosljeđuje **cijeli** `**kwargs` nepromijenjen i ostatak zadržava za sebe.

Treća točka je razlog zašto klasa uopće postoji kao zaseban sloj iznad `Audit`. Podklasa koja bi
sama izdvajala logging ključeve duplicira podjelu — i razilazi se **istog trena** kad se doda novi
logging ključ, jer njezina kopija popisa ostaje stara.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Bilo koja generička klasa sloja | `**kwargs` iz tvornice |
| **A2** | `Audit` — vlasnik identiteta i zapisa (`DR-WFL-009`) | logging ključevi |
| **A3** | `PresetDecorator` — u podklasi, nad ostatkom ključeva | konfiguracijski ključevi |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | podklasa se konstruira → `super().__init__(**kwargs)` |
| **EV02** | bilo koji poziv `debug` / `info` / `warning` / `error` |
| **EV03** | objekt ulazi u zapis → `__str__` / `__repr__` |

## 4. Preduvjeti

1. Podklasa navodi `Wattleflow` **prvi** u popisu baza, ispred svojeg pattern sučelja.
2. Podklasa **ne imenuje** `Audit` sama.
3. Podklasa ne izdvaja logging ključeve iz `**kwargs`.

## 5. Normalan tok

1. **EV01** — podklasa poziva `super().__init__(**kwargs)`; lanac vodi kroz `Wattleflow` u `Audit`,
   koji uzme logging ključeve.
2. Podklasa zatim gradi vlastito stanje iz preostalih ključeva, tipično kroz `PresetDecorator`.
3. **EV02/EV03** — identitet i zapis dolaze iz `Audit`: `name` (ime konkretnog tipa), `__str__`
   (ime), `__repr__` (`TypeName()`).

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| podklasa navede `Wattleflow` iza `Generic[…]` | MRO se ne linearizira kad se u miks doda `IOriginator` | konstrukcija pada pri definiciji klase (`FRQ-BBD-15.1` §8 k.1) |
| podklasa sama imenuje `Audit` | audit stiže dvaput u MRO | prekršaj `DR-WFL-009`; nije mjereno lintom |
| podklasa sama pop-a logging ključ | podjela je duplicirana | razilazi se pri sljedećem novom ključu |

## 7. Rezultat

Svaki objekt frameworka ima ime koje se ne može promijeniti i zapis koji se ne može izostaviti, a
konfiguracijski ključevi stižu tamo gdje pripadaju bez ijedne kopije popisa u podklasama.

## 8. Kriteriji prihvaćanja

1. Klasa nasljeđuje `Audit` i `IWattleflow`, tim redom. ✅
2. `__slots__ = ()` — korijen ne dodaje stanje. ✅
3. Tijelo `__init__` je isključivo `super().__init__(**kwargs)`. ✅
4. Modul deklarira `__all__`; import closure je `wattleflow.*`. ✅
5. Docstring imenuje ugovor koji klasa jamči. ✅
6. Nijedna podklasa u sloju ne imenuje `Audit` niti izdvaja logging ključeve. ✅ (mjereno)
7. Nijedna podklasa u sloju ne ponavlja slotove baze. ❌ — §11 t.1

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled | `class Wattleflow(Audit, IWattleflow)` |
| 2 | pregled | `__slots__ = ()` |
| 3 | pregled | jedna naredba |
| 4 | pregled modula | `__all__ = ["Wattleflow"]`; uvozi `wattleflow.core` i `wattleflow.helpers.audit` |
| 5 | pregled docstringa | ugovor `name` / `__str__` / `__repr__` naveden izrijekom |
| 6 | `command grep -rn 'Audit' concrete/*.py` | jedina pojava izvan `base.py` je `AuditException` (drugi pojam) |
| 7 | usporedba `__slots__` po modulima | `GenericWorkflow` ponavlja `_level` i `_handler` iz `Audit.__slots__` |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` radno stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/` (22 modula) + `helpers/audit.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-SEC-06` | redakcija je odgovornost `Audit` obitelji; podklasa je ne može zaobići jer je ne nasljeđuje zaobilazno |
| `NFRQ-OBS-01/02/03` | jedno mjesto nasljeđivanja znači da lint mjeri zapis nad **svim** objektima, ne nad uzorkom |
| `NFRQ-SEC-03` | korijen uvozi samo `wattleflow.core` i `wattleflow.helpers.audit` — clean core |
| `NFRQ-ORG-08` | podjela `**kwargs` postoji jednom; to je i cijela svrha klase |

## 11. Otvoreno

1. **Ponovljeni slotovi u podklasi.** `GenericWorkflow.__slots__` sadrži `_level` i `_handler`,
   koje `Audit.__slots__` već ima (`FRQ-WFL-15.9` §11 t.3). Ponovljena deklaracija stvara drugi
   descriptor i troši dodatni prostor po instanci. Ništa to ne mjeri.
2. **Pravilo „nasljeđuj `Wattleflow` prvi" nije provedivo.** Ono je ograničenje MRO-a
   (`TODO.md` §`__slots__` i MRO), a jedini signal danas je pad pri definiciji klase — i to samo
   ako se u miks doda `IOriginator`. Klasa koja pravilo prekrši bez tog miksa prolazi tiho.
   Kandidat za AST pravilo u `wem_lint`.
3. **`Wattleflow` nema `__slots__` disciplinu prema dolje.** Korijen je `()`, ali `GenericPipeline`,
   `Strategy` obitelj, `ThreadSafeObservable`, `LazyIterator` i `Orchestrator` slotove ne
   deklariraju, pa nose `__dict__`. `HLRQ-15` §4 t.5 to traži; nijedno pravilo ne mjeri.
