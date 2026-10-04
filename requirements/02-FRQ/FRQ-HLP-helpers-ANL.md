# Analiza: `Attribute` i `NameHelper` (`concrete/helpers.py`)

| | |
|---|---|
| **Datum** | 2026-10-04 |
| **Vrsta** | analiza (nije odluka) |
| **Povod** | zatvaranje defekata [`FRQ-HLP`](FRQ-HLP-helpers.md); usporedba s predloženom izvedbom `helpers.py` |
| **Trojka (D-10)** | alat: `unittest`, `timeit`, `ast` (statičko brojanje) · kriterij: [`FRQ-HLP`](FRQ-HLP-helpers.md) odjeljak 13 · platforma: CPython 3.12.14, Linux/WSL2, okruženje `workflow`, dokumentacija `v0.0.5`, kod `v0.0.1.23` |
| **Zahtjev** | [`FRQ-HLP`](FRQ-HLP-helpers.md) |

## 1. Sažetak

- **Ispravci:** `convert` vraća pretvorenu vrijednost; ispravljeno je deset točaka iz popisa (`mandatory`, `get`, `optional`, `exists`, `evaluate`, `allowed`, `print_all`, `print_prop`), svaka s testom koji je prije ispravka padao (§3).
- **Usporedba s predloženom izvedbom:** na kandidatu prolazi 55 od 56 tada postojećih testova ponašanja; razlike su razvrstane na ono što je preuzeto i ono što je ostalo za razgovor (§4, §5).
- **Preuzeto** (dokazivo sigurnije ili bolje): zajednički `_resolve`, `_extra` (sudar imena u `kwargs`), `NameHelper._members` (objekti sa `__slots__`), `NameHelper.owner`, imena u `Attribute` kao omotači `NameHelper`, rani izlaz u `evaluate`, `optional` vraća vrijednost i primjenjuje zadanu `0`. Uz to: klasa se provjerava **prije** instanciranja.
- **Odluke (2026-10-04, §9):** tri ograničenja ostaju kakva jesu, 14 metoda bez korisnika se zadržava, a `@classmethod` je primijenjen samo na privatni `_resolve`; preimenovanje parametra `cls` ostaje otvoreno (`DEF-HLP-09`).
- **Trošak i uporaba:** 26 javnih metoda, medijan po pozivu između 73 i 5650 ns, daleko ispod proračuna od 200 000 ns (§6); 14 od 26 metoda nema korisnika u `src`, 7 se koristi samo unutar `helpers.py` (§7).
- **Testovi:** `workflow/tests` 239/239; `blackwattle` skup nepromijenjen (4 neovisna pada u `tests/metrics`, 34 greške okoline bez `PIL`, `docx`, `numpy`, `pyspark`).

## 2. Metoda i skripte

| što | gdje |
|---|---|
| testovi ponašanja, po metodi | [`test_attribute_helpers.py`](../../../workflow/tests/test_attribute_helpers.py) (62 testa) |
| testovi `convert` | [`test_attribute_convert.py`](../../../workflow/tests/test_attribute_convert.py) (10 testova) |
| struktura (`NFRQ-ORG-05`) | [`test_helpers_structure.py`](../../../workflow/tests/test_helpers_structure.py) (4 testa) |
| mjerenje troška svih javnih metoda | [`test_helpers_cost.py`](../../../workflow/tests/test_helpers_cost.py) (7 testova) |
| usporedba ponašanja i troška na jednom stablu | [`2026-10-04-helpers-analysis-compare.py`](../06-ANALYSIS/2026-10-04-helpers-analysis-compare.py) |
| statičko brojanje uporabe | [`2026-10-04-helpers-analysis-usage.py`](../06-ANALYSIS/2026-10-04-helpers-analysis-usage.py) |

Trošak: za svaku metodu jedan reprezentativan poziv; probom od 200 poziva procjenjuje se koliko ponavljanja stane u oko 20 ms;
pet ponavljanja (`timeit`); **medijan** troška jednog poziva. Ispis `print_all` i `print_prop` odbacuje se. Ponavljanje s ispisom
tablice: `cd workflow && WF_TIMING=1 PYTHONPATH=src:../core/src python -m unittest tests.test_helpers_cost -v`. Ulazi su mali, pa
iznosi pokazuju cijenu same metode; kod `get` uključuju izradu ulaznog rječnika, a kod `load_from_class` instanciranje jednostavne klase.
Iznosi variraju nekoliko posto među pokretanjima. Uporaba je brojana statički (AST), pa se ne vide dinamički pozivi (`getattr(Attribute, …)`)
ni ponovni izvoz.

## 3. Ispravci iz popisa

| metoda | ispravak | testovi |
|---|---|---|
| `convert` | vraća pretvorenu vrijednost (član `Enum` po imenu ili vrijednosti, ili `cls(value)`), ne dira pozivateljev rječnik | `test_attribute_convert` (prije ispravka pada 5) |
| `mandatory` | učitana instanca sprema se na pozivatelja; pozivatelj ide u učitavanje; `{name!r}` | `MandatoryTest` |
| `get` | bez tipa vraća objekt; učitana klasa provjerava se prema tipu; neobavezna vrijednost pogrešnog tipa dize iznimku, ne tihi `None` | `GetTest` |
| `optional` | ključ s `None` je odsutan | `OptionalTest` |
| `exists` | odsutan je samo `None`; obitelj `IWattleflow` provjerava se prije čitanja | `ExistsTest` |
| `evaluate` | nema prečaca za samu klasu | `EvaluateTest` |
| `allowed` | prazan popis zabranjuje svaki ključ; klasa `list` umjesto popisa dize `AttributeException` | `AllowedTest` |
| `print_all`, `print_prop` | obična petlja, ne vraćaju popis `None`-ova | `PrintTest` |

Prije ispravaka padalo je 21 od 47 tadašnjih testova. Statički prolaz: 65 poziva `evaluate`, nijedan ne predaje samu klasu.

## 4. Usporedba s predloženom izvedbom

Ista skripta ([`compare`](../06-ANALYSIS/2026-10-04-helpers-analysis-compare.py)) izvedena na tri stabla: izvorna (moja) verzija, predložena verzija i spojena verzija.
Razlike u ponašanju (`built` je broj konstruiranih instanci):

| scenarij | izvorna | predložena | spojena |
|---|---|---|---|
| mandatory: path -> NON-IWattleflow type (Loadable) | AttributeException; built=0 | True; built=1 | AttributeException; built=0 |
| mandatory: path of wrong class for Path | AttributeException; built=0 | AttributeException; built=1 | AttributeException; built=0 |
| get: cls=None, string value | 'abc'; built=0 | AttributeException; built=0 | 'abc'; built=0 |
| get: value of right type sets attribute? | False; built=0 | True; built=0 | False; built=0 |
| optional: default 0 (falsy) | False; built=0 | True; built=0 | True; built=0 |
| optional: returns | None; built=0 | 'd'; built=0 | 'd'; built=0 |
| print_all: slotted object | AttributeError; built=0 | None; built=0 | None; built=0 |
| list_vars: slotted object | TypeError; built=0 | list; built=0 | list; built=0 |
| name(object whose __name__ is '') | ''; built=0 | '<unknown>'; built=0 | '<unknown>'; built=0 |

Stvarni defekt izvorne verzije koji predložena rješava: `convert(…, error="boom")` diže `TypeError` (sudar imena s argumentom iznimke)
umjesto `AttributeException`; `_extra` to uklanja.

## 5. Što je preuzeto, što je ostalo

| dio predložene izvedbe | odluka | razlog |
|---|---|---|
| `_extra`, `_RESERVED`, `_PRIMITIVES` | preuzeto | uklanja `TypeError` pri sudaru imena |
| `_resolve` (jedno razrješavanje za `mandatory`, `get`, `optional`) | preuzeto, sa zaštitama | tri duplicirana puta postaju jedan |
| `evaluate`: rani izlaz, `NameHelper.owner` | preuzeto | 4× brže (375 → 89 ns); poruka se gradi samo pri grešci |
| `NameHelper._members`, `print_*`, `list_vars` | preuzeto | rade i na objektima sa `__slots__` (izvorna verzija pada) |
| `NameHelper.owner` | preuzeto | nikad ne diže, za poruke o greškama |
| `Attribute.name`, `class_name`, `type_name`, `find_object_by_name` kao omotači `NameHelper` | preuzeto | jedan izvor imena (`DEF-HLP-04`) |
| `optional` vraća vrijednost i primjenjuje zadanu `0` | preuzeto | izvorna ga je ignorirala |
| `load_from_class` provjerava klasu **prije** instanciranja | dodano uz preuzimanje | predložena instancira pa provjerava (izmjereno `built=1` za pogrešnu klasu) |
| učitavanje klase za svaki neprimitivni tip u `mandatory` | **ostavljeno** | širi uvoz modula iz niza (`NFRQ-SEC-02`); §9 t.1 |
| `cls=None` učitava klasu iz niza (vezano uz `object`) | **ostavljeno** | instancirala bi se bilo koja klasa; §9 t.3 |
| `get` uvijek sprema atribut na pozivatelja | **ostavljeno** | mijenja ugovor; §9 t.2 |
| 14 od 26 metoda nema korisnika u `src`; `@staticmethod` naspram `@classmethod` (`DEF-HLP-03`) | **ostavljeno** | mijenja oblik API-ja, ne ponašanje; §9 t.4 i t.5 |

## 6. Matrica troška

Medijan po pozivu u nanosekundama: izvorna verzija, predložena i spojena (spojena je ona u repozitoriju).

| metoda | izvorna | predložena | spojena |
|---|---:|---:|---:|
| `Attribute.allowed` | 786 | 401 | 448 |
| `Attribute.class_name` | 87 | 125 | 126 |
| `Attribute.convert` | 272 | 274 | 281 |
| `Attribute.evaluate` | 375 | 89 | 88 |
| `Attribute.exists` | 694 | 306 | 313 |
| `Attribute.find_name_by_variable` | 172 | 157 | 161 |
| `Attribute.find_object_by_name` | 69 | 106 | 105 |
| `Attribute.get` | 144 | 208 | 192 |
| `Attribute.get_attr` | 423 | 430 | 399 |
| `Attribute.load_from_class` | 4548 | 4247 | 4570 |
| `Attribute.mandatory` | 645 | 314 | 508 |
| `Attribute.name` | 71 | 106 | 108 |
| `Attribute.optional` | 631 | 319 | 318 |
| `Attribute.type_name` | 66 | 97 | 105 |
| `NameHelper.cls_name` | 88 | 89 | 91 |
| `NameHelper.list_dir` | 4472 | 5846 | 5650 |
| `NameHelper.list_vars` | 260 | 791 | 643 |
| `NameHelper.name` | 112 | 120 | 111 |
| `NameHelper.nc` | 126 | 134 | 131 |
| `NameHelper.nt` | 106 | 115 | 108 |
| `NameHelper.obj_name` | 66 | 70 | 73 |
| `NameHelper.owner` | — | 143 | 92 |
| `NameHelper.print_all` | 553 | 1253 | 991 |
| `NameHelper.print_prop` | 712 | 1561 | 1149 |
| `NameHelper.source_name` | 115 | 83 | 86 |
| `NameHelper.typ_name` | 81 | 68 | 79 |

Brže su metode koje su ranije radile suvišan posao (`evaluate`, `mandatory`, `optional`, `exists`, `allowed`). Sporije su `list_vars`
i `print_*`, jer sada čitaju i slotove, te omotači imena (jedan sloj poziva više). Sve je daleko ispod proračuna od 200 000 ns.

## 7. Matrica uporabe

Stanje na dan 2026-10-04 (kod `v0.0.1.23`); pozivi izvan `helpers.py`; stupac „unutar" broji pozive iste datoteke (uključujući `cls.` i `self.`);
„testovi" su pozivi u `workflow/tests` i `blackwattle/tests` bez testa troška.

| metoda | `workflow` pozivi | unutar `helpers.py` | `blackwattle` pozivi (datoteke) | testovi | stanje |
|---|---:|---:|---:|---:|---|
| `Attribute.allowed` | 0 | 0 | 0 (0) | 6 | ne rabi se nigdje u `src` |
| `Attribute.class_name` | 0 | 1 | 0 (0) | 1 | rabi se samo unutar `helpers.py` |
| `Attribute.convert` | 0 | 0 | 0 (0) | 13 | ne rabi se nigdje u `src` |
| `Attribute.evaluate` | 4 | 2 | 58 (22) | 5 | rabi se u `workflow` |
| `Attribute.exists` | 0 | 0 | 0 (0) | 5 | ne rabi se nigdje u `src` |
| `Attribute.find_name_by_variable` | 0 | 1 | 0 (0) | 0 | rabi se samo unutar `helpers.py` |
| `Attribute.find_object_by_name` | 0 | 0 | 0 (0) | 0 | ne rabi se nigdje u `src` |
| `Attribute.get` | 0 | 0 | 0 (0) | 17 | ne rabi se nigdje u `src` |
| `Attribute.get_attr` | 0 | 0 | 0 (0) | 0 | ne rabi se nigdje u `src` |
| `Attribute.load_from_class` | 0 | 1 | 0 (0) | 0 | rabi se samo unutar `helpers.py` |
| `Attribute.mandatory` | 0 | 0 | 44 (21) | 11 | rabi se samo u `blackwattle` |
| `Attribute.name` | 0 | 0 | 0 (0) | 1 | ne rabi se nigdje u `src` |
| `Attribute.optional` | 0 | 0 | 0 (0) | 10 | ne rabi se nigdje u `src` |
| `Attribute.type_name` | 0 | 0 | 0 (0) | 1 | ne rabi se nigdje u `src` |
| `NameHelper.cls_name` | 0 | 2 | 0 (0) | 1 | rabi se samo unutar `helpers.py` |
| `NameHelper.list_dir` | 0 | 0 | 0 (0) | 0 | ne rabi se nigdje u `src` |
| `NameHelper.list_vars` | 0 | 0 | 0 (0) | 1 | ne rabi se nigdje u `src` |
| `NameHelper.name` | 0 | 0 | 0 (0) | 0 | ne rabi se nigdje u `src` |
| `NameHelper.nc` | 1 | 1 | 0 (0) | 0 | rabi se u `workflow` |
| `NameHelper.nt` | 1 | 1 | 0 (0) | 0 | rabi se u `workflow` |
| `NameHelper.obj_name` | 0 | 3 | 0 (0) | 1 | rabi se samo unutar `helpers.py` |
| `NameHelper.owner` | 0 | 2 | 0 (0) | 3 | rabi se samo unutar `helpers.py` |
| `NameHelper.print_all` | 0 | 0 | 0 (0) | 3 | ne rabi se nigdje u `src` |
| `NameHelper.print_prop` | 0 | 0 | 0 (0) | 3 | ne rabi se nigdje u `src` |
| `NameHelper.source_name` | 2 | 0 | 0 (0) | 0 | rabi se u `workflow` |
| `NameHelper.typ_name` | 0 | 2 | 0 (0) | 1 | rabi se samo unutar `helpers.py` |

Bez ijednog korisnika u `src`: 14 od 26 (`Attribute.allowed`, `Attribute.convert`, `Attribute.exists`, `Attribute.find_object_by_name`, `Attribute.get`, `Attribute.get_attr`, `Attribute.name`, `Attribute.optional`, `Attribute.type_name`, `NameHelper.list_dir`, `NameHelper.list_vars`, `NameHelper.name`, `NameHelper.print_all`, `NameHelper.print_prop`). Samo unutar `helpers.py`: 7 (`Attribute.class_name`, `Attribute.find_name_by_variable`, `Attribute.load_from_class`, `NameHelper.cls_name`, `NameHelper.obj_name`, `NameHelper.owner`, `NameHelper.typ_name`).

## 8. Opis funkcionalnosti metoda

Svrha i učinak opisani su prema kodu u `helpers.py` (`v0.0.1.23`); trajanje je medijan jednog poziva (§2, §6).

| metoda | svrha | što postiže | trajanje (ns) | napomena |
|---|---|---|---:|---|
| `Attribute.name` | ime klase, funkcije ili modula za poruke | vraća `NameHelper.obj_name` ili „<unknown>"; ne diže iznimku | 108 |  |
| `Attribute.class_name` | ime klase instance za poruke | vraća `NameHelper.cls_name` ili „<None>"; ne diže iznimku | 126 |  |
| `Attribute.type_name` | ime tipa objekta, uvijek niz | vraća `NameHelper.typ_name` | 105 |  |
| `Attribute.find_name_by_variable` | ime klase objekta bez okidanja preopterećenog `__getattr__` (fasada, preset) | čita `__class__` kroz `object.__getattribute__` i vraća njezino ime; koristi ga `evaluate` | 161 |  |
| `Attribute.find_object_by_name` | ime objekta s zadanom vrijednošću | vraća `NameHelper.obj_name` ili „Unknown" | 105 |  |
| `Attribute.allowed` | ograda ulazne površine: smiju li se ključevi u `kwargs` | `None` daje `False`; popis mora biti `list`; ključ izvan popisa diže `AttributeException`; prazan popis zabranjuje svaki ključ; inače `True` | 448 |  |
| `Attribute.evaluate` | provjera tipa objekta | ako je očekivani tip zadan, a `target` nije njegova instanca, diže `AttributeException` s imenom pozivatelja, nađenim i očekivanim tipom; uspjeh izlazi odmah, bez gradnje poruke | 88 |  |
| `Attribute.exists` | provjera da pozivatelj ima atribut tražena tipa | pozivatelj mora biti iz obitelji `IWattleflow` (provjera prije čitanja); atribut ne smije biti `None` i mora biti tipa `cls`, inače `AttributeException` | 313 |  |
| `Attribute.get` | izvlačenje vrijednosti iz rječnika konfiguracije | uklanja ključ iz zadanog rječnika; vraća vrijednost tipa `cls` (ili bilo koju kad je `cls` `None`); naziv klase učitava (podtip `cls`, provjera prije instanciranja) i sprema na pozivatelja; neobavezan odsutan ključ daje `None`, obvezan `None` ili pogrešan tip diže iznimku | 192 | rječnik se stvara u svakom pozivu mjerenja, pa je to dio iznosa |
| `Attribute.get_attr` | čitanje atributa neovisno o `__getattr__` | vraća vrijednost iz `__dict__` ili iz slota po MRO-u; neinicijaliziran slot diže `AttributeError`, nepostojeće ime `AttributeException` | 399 |  |
| `Attribute.convert` | pretvorba konfiguracijske vrijednosti u traženi tip | vraća `kwargs[name]` pretvoren u `cls` (član `Enum` po imenu ili vrijednosti, ili `cls(value)`); ne mijenja pozivateljev rječnik; nepretvorivo ili odsutno diže `AttributeException` | 281 |  |
| `Attribute.mandatory` | obvezan konfiguracijski ključ postaje atribut pozivatelja | instancu tipa `cls` sprema kao atribut pozivatelja; za `IWattleflow` tip naziv klase učitava (podtip, provjera prije instanciranja) i sprema; vraća `True`; odsutan ključ ili pogrešan tip diže `AttributeException` | 508 |  |
| `Attribute.optional` | neobavezan konfiguracijski ključ postaje atribut pozivatelja | odsutan ključ ili `None` daje zadanu vrijednost ili ništa; vrijednost tipa `cls` sprema na pozivatelja, naziv klase učitava; vraća pohranjenu vrijednost (ili `None`) | 318 |  |
| `Attribute.load_from_class` | instanciranje klase iz njezine staze | razrješava klasu iz niza `obj`, provjerava da je podtip `cls` **prije** instanciranja, zatim instancira kroz `ClassLoader`; `TypeError` ako `obj` nije niz, `ModuleNotFoundError` ako modula nema, `ValueError` ako staza ili instanciranje padnu, `AttributeException` za pogrešan tip | 4570 | iznos uključuje uvoz i instanciranje jednostavne klase |
| `NameHelper.obj_name` | ime klase, funkcije ili modula | vraća `__name__` ili `None` | 73 |  |
| `NameHelper.cls_name` | ime klase instance | vraća `__class__.__name__` ili `None` | 91 |  |
| `NameHelper.typ_name` | ime tipa objekta, uvijek niz | vraća `type(o).__name__` | 79 |  |
| `NameHelper.name` | alias za `obj_name` | vraća isto što i `obj_name` | 111 |  |
| `NameHelper.nc` | kratica za `cls_name` | vraća `cls_name(o)` | 131 | koristi ga `concrete/exception.py` za ime pozivatelja |
| `NameHelper.nt` | kratica za `typ_name` | vraća `typ_name(o)` | 108 |  |
| `NameHelper.owner` | prikazno ime objekta za poruke o greškama | vraća atribut `name`, inače ime klase; nikad ne diže iznimku | 92 |  |
| `NameHelper.source_name` | čitljivo ime izvora za audit zapise | vraća zadnji dio staze iz atributa `filename` ili `None`; nikad ne diže iznimku, jer radi unutar audit zapisa | 86 | koriste ga `pipeline.py` i `processor.py` |
| `NameHelper.list_vars` | popis varijabli objekta | vraća imena instancijskih atributa (`__dict__` i slotovi) bez onih koja počinju i završavaju s `_` | 643 | cijena raste s brojem atributa |
| `NameHelper.list_dir` | popis članova objekta | vraća imena iz `dir(o)` (i metode) bez onih koja počinju i završavaju s `_` | 5650 | najskuplji uz `load_from_class`: `dir` skuplja članove cijelog MRO-a |
| `NameHelper.print_all` | ispis svih atributa za pregled | na standardni izlaz ispisuje „ključ: vrijednost" za svaki instancijski atribut (`__dict__` i slotovi); ne vraća ništa; ispis ide mimo audita | 991 | cijena raste s brojem atributa; mjerenje odbacuje izlaz |
| `NameHelper.print_prop` | ispis javnih i zaštićenih atributa | kao `print_all`, bez imena koja počinju i završavaju s `_` | 1149 | cijena raste s brojem atributa |

## 9. Odluke i ostalo

Odluke od 2026-10-04 (A1–A5, prihvaćene preporuke):

1. **`mandatory` učitava klasu iz niza samo za `IWattleflow` tipove: ostaje.** Predložena izvedba učitava za svaki neprimitivni tip. Širi uvoz modula iz niza (`NFRQ-SEC-02`); zaštita „podtip prije instanciranja" već postoji, pa je preostali rizik sam uvoz.
2. **`get` sprema na pozivatelja samo vrijednost koju je učitao: ostaje.** `get` nema pozivatelja u `src`; vratiti se kad ga netko treba.
3. **`get` bez tipa vraća niz kakav jest: ostaje.** Učitavanje bi instanciralo bilo koju klasu.
4. **14 od 26 metoda nema korisnika u `src`, 7 se koristi samo unutar `helpers.py`: zadržavaju se** dok se ne zna koriste li ih vanjski korisnici; tada ukloniti.
5. **`@staticmethod` naspram `@classmethod` (`NFRQ-ORG-05`, `DEF-HLP-03`): provedeno djelomično.** Privatni `_resolve` je `@classmethod`. Javne metode `exists`, `get`, `load_from_class`, `mandatory` i `optional` vežu domenski tip na parametar `cls`, pa po izuzetku c.1 ostaju `@staticmethod` (odgođeno, ne prekršaj). Moja preporuka da se sve prebaci bila je preoptimistična: preimenovanje `cls` u `expected` uz pozicijske parametre prošlo je testove i poziva u `src`, ali slomilo bi 14 poziva s ključnim riječima (`caller=`, `name=`, `cls=`) u 4 datoteke primjera (`blackwattle/examples/workflows`) i vjerojatno korisničke skripte, pa je vraćeno. Strukturni test (`test_helpers_structure.py`) sada čuva da nijedna druga metoda ne imenuje vlastitu klasu. Preimenovanje je otvoreno kao `DEF-HLP-09`.

Informativno:

6. **Cijena `list_vars` i `print_*` porasla je 2–3×** zbog čitanja slotova; apsolutno oko 1 µs. Prihvaćeno radi ispravnosti na objektima sa `__slots__`; ako se ikada pozivaju po stavci, vrijedi ponovno mjeriti.
7. **Docstring `NameHelper` još spominje „ORG-01 cycle"**; zasad se ignorira (komentar u kodu).
