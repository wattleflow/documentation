# FRQ-DOC-15.8 — Dokument, adapter i fasada

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog. **Dopunjeno 2026-09-11** pravilom o sadržaju i metapodacima (odluka autora, bez DR-a — §11) |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `DOC`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `Document(Wattleflow, IAdaptee, Generic[Content], ABC)`, `DocumentAdapter(Wattleflow, IAdapter, Generic[Adaptee])`, `DocumentFacade(Wattleflow, ITarget, Generic[Adaptee], ABC)` |
| **Sestrinski** | [`FRQ-PIP-15.2`](FRQ-PIP-15.2-pipeline.md) (prima `facade`) · [`FRQ-BBD-15.1`](FRQ-BBD-15.1-blackboard.md) (drži ih na platnu) · [`FRQ-STR-15.4`](FRQ-STR-15.4-strategy.md) (stvara i piše) |
| **Izvedba** | `workflow/src/wattleflow/concrete/document.py` (254 linije) |

## 1. Predmet

Dokument je **jedinica podatka koja putuje tokom**. Tri klase, tri različita razloga:

| klasa | pattern | zašto postoji |
|---|---|---|
| `Document` | Adaptee | nosi **sadržaj, identitet i povijest izmjena** |
| `DocumentAdapter` | Adapter | prevodi dokument u ono što tok očekuje |
| `DocumentFacade` | Facade | ono što pipeline i strategije **stvarno drže** u rukama |

Pipeline nikad ne dobiva `Document` nego `DocumentFacade`. Razlog je delegacija: fasada
prosljeđuje svako nepoznato **javno** ime dokumentu iza sebe, pa strategija koja treba `filename`
ili `identifier` piše `facade.filename` bez obzira na to kojeg je tipa dokument. Nepoznato ime
koje dokument nema daje `AttributeError` s imenom **fasade**, ne adaptera — trag pokazuje na sloj
na kojem je poziv nastao.

**Povijest izmjena je zaštićena.** Ključevi `last_change_key` i `last_change_time` su rezervirani
(`_AUDIT_KEYS`): pozivatelj ih **ne smije** postaviti, jer bi time falsificirao povijest izmjene
dokumenta. Piše ih isključivo `update_metadata`/`update_content`, iznutra.

**Tip sadržaja se zaključava.** Prvi dodijeljeni sadržaj određuje `_expected_type`; svaka sljedeća
izmjena mora biti istog tipa. Dokument koji je počeo kao tekst ne postaje binaran usput.

**Sadržaj nosi podatke, metapodaci ih opisuju** (odluka autora 2026-09-11). Podaci — tekst, zapisi,
tablica — žive u sadržaju dokumenta. Metapodaci nose ono što podatke opisuje: izvor, ime datoteke,
list, shemu, broj redaka, trag obrade. Za strukturirane podatke koristi se dokument kojem je sadržaj
tablica (DataFrame), **gdje god je to moguće**; ime datoteke taj dokument već čita i piše kroz
metapodatke. Podaci u metapodacima su skriven kanal: vrsta dokumenta ne govori što nosi, a pristup
ide preko dogovorenog ključa.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `StrategyCreate` — stvara dokument | sadržaj |
| **A2** | `GenericPipeline` — transformira | `facade` |
| **A3** | `StrategyWrite` — čita sadržaj i metapodatke pri pisanju | `facade` |
| **A4** | `GenericBlackboard` — drži fasade na platnu | ključ (identifikator ili digest) |
| **A5** | `Document` / `DocumentFacade` — predmet ovog zahtjeva | sadržaj i povijest |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 gradi dokument → `__init__` → `created_at` + prvi sadržaj |
| **EV02** | fasada se gradi nad dokumentom → `DocumentFacade.__init__` |
| **EV03** | A2 mijenja sadržaj → `update_content` |
| **EV04** | A2/A3 mijenja metapodatak → `update_metadata` |
| **EV05** | A3 čita kroz fasadu → `facade.<ime>` → delegacija |
| **EV06** | platno se prazni → `clean()` |
| **EV07** | kraj životnog ciklusa → `__del__` |

## 4. Preduvjeti

1. Specijalizacija dokumenta je implementirala `size`.
2. Objekt predan adapteru i fasadi je `IAdaptee` — inače `TypeError`.
3. Ključ metapodatka nije prazan i nije rezerviran.

## 5. Normalan tok

1. **EV01** — identitet je `uuid4()`, dodijeljen pri konstrukciji i **nepromjenjiv** izvana
   (`BR-15-03`). Metapodaci kreću praznim rječnikom, pa se upisuje `created_at` iz `Now.utc()`,
   pa prvi sadržaj.
2. **EV02** — fasada gradi adapter **s istom konfiguracijom** koju je sama dobila: adapter je
   njezin implementacijski detalj i ne smije padati na zadane postavke zapisa.
3. **EV03** — prvi sadržaj postavlja `_expected_type`; svaki sljedeći se provjerava prema njemu.
   Uspješna izmjena upisuje `last_change_key="content"` i vrijeme.
4. **EV04** — `update_metadata` odbija prazan ključ i rezervirani ključ, pa upiše vrijednost i
   osvježi `last_change_key` / `last_change_time`.
5. **EV05** — `facade.<ime>`: `_`-imena se odbijaju odmah; javno ime se traži na dokumentu iza
   adaptera i prosljeđuje ako postoji.
6. **EV06** — `clean()` oslobađa sadržaj i briše metapodatke.
7. Metapodaci se izvana vide **samo za čitanje** (`MappingProxyType`), kao i platno blackboarda.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| sadržaj drugog tipa nego prvi | `TypeError` s oba imena tipa | dokument ne mijenja prirodu usput |
| `update_content(None)` | sadržaj se briše, povijest se upisuje | brisanje je legitimna izmjena |
| čitanje `content` kad je `None` | **`ValueError`** | vidi §11 t.2 |
| prazan ključ metapodatka | `ValueError` | ključ bez imena nije metapodatak |
| rezervirani ključ (`last_change_*`) | `ValueError` s objašnjenjem | povijest se ne može falsificirati |
| objekt koji nije `IAdaptee` | `TypeError` iz adaptera odnosno fasade | tok ne prima tuđi tip |
| `request()` vrati `None` | `ValueError` s imenom klase | prazna fasada ne ide dalje |
| pristup `_`-imenu kroz fasadu | `AttributeError` s imenom **fasade** | delegacija ne otvara privatnu površinu |
| nepoznato javno ime | `AttributeError` s imenom **fasade** | trag pokazuje na sloj poziva |
| iznimka u `__del__` | `try/except Exception: pass` | destruktor ne diže (`HLRQ-15` §4 t.4) |

## 7. Rezultat

Stavka putuje tokom kao jedan objekt sa stabilnim identitetom, tipski zaključanim sadržajem i
poviješću izmjena koju pozivatelj ne može krivotvoriti. Pipeline i strategije rade s fasadom, pa
promjena tipa dokumenta ne dira nijedan od njih.

## 8. Kriteriji prihvaćanja

1. Identitet nastaje pri konstrukciji i ne postavlja se izvana (`BR-15-03`). ✅
2. Rezervirani ključevi povijesti nisu dostupni pozivatelju. ✅
3. Tip sadržaja se zaključava na prvoj dodjeli. ✅
4. Metapodaci se izvana vide samo za čitanje. ✅
5. Fasada delegira samo javna imena i griješi vlastitim imenom. ✅
6. Adapter se gradi s konfiguracijom fasade, ne sa zadanom. ✅
7. `Wattleflow` prethodi `Generic[Content]` u popisu baza (MRO). ✅
8. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
9. Dokument se može staviti u `set` ili koristiti kao ključ rječnika. ❌ — §11 t.1
10. `content` ne diže iznimku u stanju koje je klasa sama proglasila legalnim. ❌ — §11 t.2
11. Podaci su u sadržaju, a metapodaci ih samo opisuju. ❌ — put zapisa za RSS i XML drži zapise u
    metapodacima (§11)

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `__init__` | `self._identifier: str = str(uuid4())`; nema settera |
| 2 | pregled `update_metadata` | `if key in _AUDIT_KEYS: raise ValueError(...)` |
| 3 | pregled `update_content` | `_expected_type` postavljen jednom, potom `isinstance` provjera |
| 4 | pregled `metadata` propertyja | `MappingProxyType(self._metadata)` |
| 5 | pregled `DocumentFacade.__getattr__` | `_`-grana prva; obje poruke nose `self.__class__.__name__` |
| 6 | pregled `DocumentFacade.__init__` | `DocumentAdapter(adaptee, **kwargs)` |
| 7 | pregled popisa baza | `class Document(Wattleflow, IAdaptee, Generic[Content], ABC)` |
| 8 | pregled modula | `__all__` = 3 imena; uvozi `uuid`, `datetime`, `types`, `typing`, `abc` + `wattleflow.*` |
| 9 | `command grep -n 'def __hash__' concrete/document.py` | **nema** — a `__eq__` postoji, pa Python postavlja `__hash__ = None` |
| 10 | pregled `content` propertyja i `update_content` | `update_content(None)` je dopušten, `content` na `None` diže `ValueError` |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda, `command grep`, i provjera pravila
`__eq__`/`__hash__` izvođenjem (`python3 -c`); kriterij — §8 gore; platforma — `workflow` radno
stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:** `concrete/document.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Document` je rezervirani primitiv; adapter i fasada su patterni oko njega, ne novi primitivi |
| `NFRQ-SEC-02` | fasada odbija delegirati `_`-imena; metapodaci su read-only pogled |
| `NFRQ-SEC-03` | clean core tier; nijedan third-party uvoz — formati (PDF, DOCX, Avro…) žive u processorsu |
| `NFRQ-SEC-06` | povijest izmjena je zaštićena od pozivatelja — `AU-3` integritet zapisa počinje ovdje |
| `NFRQ-OBS-01` | konstrukcija, izmjena i čišćenje su `DEBUG`; dokument nije jedinica posla |
| `NFRQ-ORG-05` | `utc_time_stamp` ne dira nijedan član — trebao bi biti `@staticmethod` (§11 t.3) |

## 11. Otvoreno

1. **Dokument je nehashabilan.** Klasa definira `__eq__` bez `__hash__`, pa ga Python postavlja na
   `None`: `hash(document)` diže `TypeError: unhashable type`. Posljedica: dokument se ne može
   staviti u `set` ni koristiti kao ključ rječnika. Danas to ne puca jer platna ključuju po
   `identifier` odnosno digestu (stringovi), ali svaka dedup-logika koja posegne za skupom pada.
   Ispravak je jedna linija (`__hash__ = lambda self: hash((type(self), self._identifier))`) i
   slaže se s postojećim `__eq__`.
2. **`content` diže iznimku u legalnom stanju.** `update_content(None)` je izričito dopušten put
   (postavlja sadržaj na `None` i upisuje povijest), a `content` property nad tim stanjem diže
   `ValueError("Content value is missing or uninitialised.")`. Ili je `None` legalan sadržaj — pa
   ga property mora vratiti — ili nije, pa ga `update_content` ne smije primiti.
3. **`utc_time_stamp` je instance-metoda koja ne dira `self`.** Po `NFRQ-ORG-05` t.2 to je
   `@staticmethod`; izbor dekoratora izražava namjeru. Uz to `TODO.md` bilježi da
   `strategies/documents/json.py:268` koristi `Now.utc()` izravno dok sve ostale write-strategije
   idu kroz `document.utc_time_stamp()` — dvije rute do iste vrijednosti.
4. **Zaključavanje tipa ne uhvati dokument koji je počeo prazan.** Ako je prvi sadržaj `None`,
   `_expected_type` ostaje `None` i brava se nikad ne aktivira — takav dokument prihvaća bilo koji
   tip zauvijek.
5. **Fasada provjerava tip nakon `super().__init__`, adapter prije.** `DocumentAdapter` odbija
   ne-`IAdaptee` prije nego pozove roditelja; `DocumentFacade` prvo zove roditelja pa provjeri.
   Razlika je bezopasna danas, ali dvije klase u istom modulu rade isto na dva načina.
6. **`DocumentFacade.__getattr__` gradi adaptee pri svakom promašaju** (`self._adapter.request()`).
   Za dokument to je jeftin `return self`, ali ugovor `IAdapter` to ne jamči za drugu izvedbu.
7. **Zapisi u metapodacima.** Put zapisa za RSS i XML (`HLRQ-17`, `FRQ-PIP-17.1` korak 6,
   `DR-PRC-004` t.5) predaje zapise strategiji zapisa preko metapodataka dokumenta — protivno pravilu iz
   §1 i kriteriju 11. Pravilo je unijeto odlukom autora 2026-09-11 bez DR-a; usklađivanje traži DR
   (D-03). Analiza `2026-09-11-dataframe-working-form.md`.
