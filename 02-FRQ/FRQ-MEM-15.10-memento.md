# FRQ-MEM-15.10 — Snimka stanja (memento)

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `MEM`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericMemento(Wattleflow, IMemento)` — nepromjenjiva snimka stanja |
| **Sestrinski** | [`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md) (jedini proizvođač i potrošač) · `FRQ-PTN-15.13` *(nenapisan)* (stanje koje snimka nosi) |
| **Izvedba** | `workflow/src/wattleflow/concrete/memento.py` (75 linija) |

## 1. Predmet

Memento je **nepromjenjiv spremnik** koji nosi ono što je potrebno za nastavak prekinutog posla.
Jedini je primitiv ovog sloja bez apstraktnih metoda i bez specijalizacija — jer i nema što
specijalizirati: sadržaj snimke određuje onaj tko je snima (`IOriginator`), ne snimka.

Tri načina pristupa istom sadržaju, namjerno:

| pristup | čemu služi |
|---|---|
| `memento.cycle` (atribut) | čitljivost na mjestu poziva |
| `get_state()` | kanonski ključ `"state"`, koji `IMemento` traži |
| `to_dict()` | kad se snimka predaje dalje kao podatak |

**Nepromjenjivost je plitka, i to je zapisano.** `_data` je `MappingProxyType` nad kopijom
rječnika, pa se sam skup ključeva ne može mijenjati; vrijednosti se čuvaju **po referenci**. Za
integritet promjenjive vrijednosti odgovoran je vlasnik snimke, ne snimka. Ta je granica u
docstringu izrečena, ne prešućena (D-11).

**Payload se ne krati.** Konstruktor prosljeđuje cijeli `**payload` naviše, a `Wattleflow` čita
logging ključeve s **vlastite kopije** — pa snimka zadrži svaki ključ koji je pozivatelj predao,
uključujući onaj koji se slučajno zove `level` ili `handler`.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `IOriginator` (danas: `GenericProcessor`) — snima i vraća | `cycle`, `state` |
| **A2** | `GenericMemento` — predmet ovog zahtjeva | payload |
| **A3** | Pozivatelj koji čuva snimku | `to_dict()` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 snima → `save_state()` → `GenericMemento(cycle=…, state=…)` |
| **EV02** | A1 vraća → `restore_state(memento)` → `get_state()` + atributi |
| **EV03** | A3 čita snimku kao podatak → `to_dict()` / `in` |

## 4. Preduvjeti

1. Payload je predan kao **imenovani** argument; pozicijskih nema.
2. Za `get_state()` payload nosi ključ `state`; inače je rezultat `None`.
3. Vrijednosti koje moraju preživjeti nepromijenjene su ili nepromjenjive same, ili ih vlasnik
   kopira prije snimanja.

## 5. Normalan tok

1. **EV01** — konstruktor kopira payload u rječnik pa ga zamota u `MappingProxyType`.
2. **EV02** — pristup atributom ide kroz `__getattr__` na `_data`; ključ kojeg nema daje običan
   `AttributeError` s imenom ključa.
3. `get_state()` vraća `_data.get("state")` — **bez** iznimke ako ključa nema.
4. `to_dict()` vraća novi rječnik; pozivatelj ga smije mijenjati bez posljedica za snimku.
5. `__repr__` ispisuje sva polja s vrijednostima (`k=v!r`).

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| pristup nepostojećem ključu | `AttributeError(key)` | ključ se tretira kao odsutan, ne kao interni kvar |
| `get_state()` bez ključa `state` | `None` | odsutnost stanja nije iznimka |
| pokušaj izmjene `_data` | `TypeError` iz `MappingProxyType` | skup ključeva je zaključan |
| promjena vrijednosti kroz vanjsku referencu | **snimka se mijenja s njom** | plitka kopija; odgovornost vlasnika |
| payload nosi `level` / `handler` | ključ ostaje u snimci **i** stiže loggeru | dvostruka namjena ključa je prihvaćena |

## 7. Rezultat

Prekinuti posao ima čime nastaviti: snimka nosi stanje automata i napredak, a njezin skup ključeva
se od trenutka nastanka ne mijenja. Nijedan potrošač snimke ne mora znati koji ju je originator
napravio.

## 8. Kriteriji prihvaćanja

1. Skup ključeva je nepromjenjiv nakon konstrukcije. ✅
2. Plitkost kopije je **deklarirana**, ne prešućena (D-11). ✅
3. Payload se ne krati — logging ključevi ostaju i u snimci. ✅
4. Odsutan ključ daje `AttributeError`, odsutno `state` daje `None`. ✅
5. `to_dict()` vraća kopiju koju pozivatelj smije mijenjati. ✅
6. `__slots__` je deklariran; modul deklarira `__all__`. ✅
7. Import closure je `stdlib ∪ wattleflow`. ✅
8. Snimka se može trajno pohraniti. ❌ — §11 t.1
9. `__repr__` ne otkriva povjerljive vrijednosti. ❌ — §11 t.2

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `__init__` | `MappingProxyType(dict(payload))` |
| 2 | pregled docstringa | „shallow copy … values are stored by reference. Owners are responsible…" |
| 3 | pregled `__init__` + komentar | cijeli `**payload` ide i naviše i u `_data` |
| 4 | pregled `__getattr__` / `get_state` | `raise AttributeError(key)` odnosno `.get("state")` |
| 5 | pregled `to_dict` | `return dict(self._data)` |
| 6 | pregled modula | `__slots__ = ("_data",)`; `__all__ = ["GenericMemento"]` |
| 7 | pregled uvoza | `types`, `typing`, `collections.abc` + `wattleflow.*` |
| 8 | pregled docstringa | `TODO: persistence layer — pluggable read/write strategies` |
| 9 | pregled `__repr__` | `", ".join(f"{k}={v!r}" …)` nad **svim** poljima |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda; kriterij — §8 gore; platforma —
`workflow` radno stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/memento.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Memento` je rezervirani primitiv; snimka ne postaje spremište (`FRQ-REP-15.7`) |
| `NFRQ-SEC-02` | jedno javno ime u `__all__`; skup ključeva zaključan proxyjem |
| `NFRQ-SEC-03` | clean core tier; nijedan third-party uvoz |
| `NFRQ-SEC-06` | `__repr__` ispisuje sve vrijednosti — vidi §11 t.2 |
| `NFRQ-OBS-01` | snimka ne prijavljuje ništa; prijavljuje originator |

## 11. Otvoreno

1. **Trajna pohrana snimke ne postoji.** Docstring nosi `TODO` za perzistencijski sloj sa
   zamjenjivim read/write strategijama (analogno `PresetDecorator`). Bez njega nastavak radi samo
   unutar **istog procesa**: `GenericProcessor.save_state()` vrati objekt koji nitko ne zapisuje,
   pa pad procesa odnosi i snimku. Cijela `restore_state` grana procesora
   (`FRQ-PRC-15.3` §5, §11 t.1) time je za sada bez stvarnog scenarija oporavka. `TODO` u
   docstringu nije zahtjev — kandidat je za `FRQ-MEM-*` nastavak.
2. **`__repr__` ispisuje sve vrijednosti bez redakcije.** Snimka prima proizvoljan payload; ako
   originator u nju stavi konfiguracijsku vrijednost s tajnom, `repr(memento)` je otkriva. Svaki
   drugi `__repr__` ovog sloja ispisuje ime i stanje, ne sadržaj. `BR-15-08` i `NFRQ-SEC-06` traže
   suprotno.
3. **Klasa nije `ABC` i nema apstraktnih metoda**, za razliku od svih ostalih generičkih klasa
   sloja. To je ispravno — snimka nema što specijalizirati — ali odstupa od obrasca dovoljno da
   zaslužuje biti zapisano, a ne otkriveno čitanjem.
4. **Ključ `state` je kanonski po dogovoru, ne po ugovoru.** `get_state()` ga traži imenom, a
   ništa ne provjerava je li predan ni kojeg je tipa. `GenericProcessor.restore_state` na
   `saved_state` odmah radi `(saved_state, ProcessorAction.LOAD) not in TRANSITIONS`, pa snimka s
   krivim tipom pada tamo, s porukom o automatu umjesto o snimci.
