<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-HLP — Pomoćnici za atribute i imena

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) — zajednički ugovor generičke klase (§4), `BR-PTN-05` (iznimka se omata s uzrokom) |
| **Predmet** | `Attribute` (provjera, pretvorba i dohvat atributa i konfiguracijskih ključeva) i `NameHelper` (imena objekata za zapise) — klase bez stanja |
| **Sestrinski** | [`FRQ-PTN`](FRQ-PTN-root-base.md) (`Wattleflow.__init__` i presetovi ih koriste) · [`FRQ-BBD`](FRQ-BBD-blackboard.md) (`PresetDecorator`) |
| **Izvedba** | `workflow/src/wattleflow/concrete/helpers.py` |
| **Dijagrami** | inline (02, 03, 06, 08, 09) — pogledi izvedeni iz koda, ne izvor istine (D-13) |

# Index
- [01. Interface](#01-interface)
- [02. Class Diagrams](#02-class-diagrams)
- [03. Context Diagram](#03-context-diagram)
- [04. User Diagram](#04-user-diagram)
- [05. Events](#05-events)
- [06. Use Case Diagrams](#06-use-case-diagrams)
- [07. Constraints and Preconditions](#07-constraints-and-preconditions)
- [08. Sequence Diagrams](#08-sequence-diagrams)
- [09. Flow Chart Diagrams](#09-flow-chart-diagrams)
- [10. State Machine](#10-state-machine)
- [11. Non-Functional Requirements](#11-non-functional-requirements)
- [12. Results](#12-results)
- [13. Acceptance Criteria](#13-acceptance-criteria)
- [14. Verification](#14-verification)
- [15. Open issues](#15-open-issues)
- [16. References](#16-references)
- [17. Change history](#17-change-history)

## 01. Interface

Dvije klase s metodama na razini klase. **`Attribute`** provjerava i čita konfiguraciju koju generičke klase primaju kroz
`**kwargs`: tip, obveznost, dopušteni ključevi, učitavanje klase iz niza. Pogreške izlaze kao `AttributeException`. **`NameHelper`**
daje imena za zapise i **nikad ne smije dići iznimku** (radi unutar audit zapisa, gdje bi iznimka prikrila događaj koji se prijavljuje).
Nijedna klasa ne drži stanje i ne nasljeđuje `Wattleflow`.

| metoda `Attribute` | što radi |
|---|---|
| `evaluate(caller, target, type)` | `AttributeException` ako `target` nije instanca `type`; bez tipa ništa ne provjerava |
| `allowed(caller, allowed, **kw)` | `AttributeException` ako `kw` sadrži ključ izvan `allowed` (prazan popis zabranjuje svaki ključ); `False` ako popis nije zadan |
| `mandatory(caller, name, cls, **kw)` | obvezan ključ; instancu `cls` postavlja kao atribut pozivatelja; za `IWattleflow` tip učitava klasu iz niza |
| `get(caller, name, kwargs, cls, mandatory)` | uzima (briše) ključ iz rječnika; bez tipa vraća objekt kakav jest; niz učitava kao klasu i sprema na pozivatelja |
| `optional(caller, name, cls, default, **kw)` | kao `mandatory`, uz zadanu vrijednost; ključ s `None` je odsutan; vraća pohranjenu vrijednost |
| `convert(caller, name, cls, **kw)` | vraća vrijednost pretvorenu u `cls` ili člana `Enum`; ne dira pozivateljev rječnik |
| `exists(caller, name, cls)` | pozivatelj iz obitelji `IWattleflow` ima atribut koji nije `None` i ima tip |
| `load_from_class(name, obj, cls, ...)` | razrješava klasu iz niza, provjerava podtip prije instanciranja, zatim `ClassLoader(obj).instance` |
| `get_attr(caller, name)` | atribut iz `__dict__` ili iz slotova po MRO-u |
| `name`, `class_name`, `type_name`, `find_*` | imena bez iznimke (omotači `NameHelper`); zadane „<unknown>", „<None>", „Unknown" |

| metoda `NameHelper` | što radi |
|---|---|
| `obj_name`, `cls_name`, `typ_name` (`name`, `nc`, `nt`) | `__name__`, `__class__.__name__`, `type().__name__` |
| `owner(o)` | prikazno ime za poruke o greškama (`name`, inače ime klase); nikad ne diže |
| `source_name(o)` | naziv datoteke iz atributa `filename`; `None` bez iznimke |
| `list_vars`, `list_dir`, `print_all`, `print_prop` | pregled i ispis; `list_vars` i `print_*` obuhvaćaju `__dict__` i slotove; ispis ide na standardni izlaz |

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Attribute, NameHelper

top to bottom direction

  class Attribute <<utility>> {
    {static} +evaluate(caller, target, expected_type)
    {static} +allowed(caller, allowed, **kw) : bool
    {static} +mandatory(caller, name, cls, **kw) : bool
    {static} +get(caller, name, kwargs, cls, mandatory)
    {static} +optional(caller, name, cls, default, **kw)
    {static} +convert(caller, name, cls, **kw)
    {static} +exists(caller, name, cls)
    {static} +load_from_class(name, obj, cls, caller, **kw)
    {static} +get_attr(caller, name)
    {static} +name(o) : str
    {static} +class_name(o) : str
    {static} +type_name(o) : str
    {static} +find_name_by_variable(obj)
    {static} +find_object_by_name(obj)
    {static} -_resolve(caller, name, value, expected, params, load) : object
    {static} -_class_at(path) : object
  }
  class NameHelper <<utility>> {
    {static} +obj_name(o)
    {static} +cls_name(o)
    {static} +typ_name(o) : str
    {static} +owner(o) : str
    {static} +source_name(o) : str | None
    {static} +list_vars(o)
    {static} +list_dir(o)
    {static} +print_all(o) : None
    {static} +print_prop(o) : None
    {static} +name(o)
    {static} +nc(o)
    {static} +nt(o) : str
    {static} -_members(o) : dict
  }
  class AttributeException
  interface IWattleflow
  class ClassLoader
Attribute .right.> AttributeException
Attribute .right.> NameHelper : names, owner
Attribute .right.> ClassLoader : imported at call time
Attribute .up.> IWattleflow : exists, mandatory
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Attribute, NameHelper

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Attribute, NameHelper", "Validates configuration and resolves record names")
System_Ext(gk, "GenericWorkflow, GenericPipeline, GenericProcessor, Strategy", "Calls Attribute and NameHelper")
System_Ext(cl, "ClassLoader", "wattleflow.helpers.system")
Rel_L(gk, sys, "Validates configuration and requests names")
Rel_R(sys, cl, "Loads class from string")
@enduml
```

## 04. User Diagram

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | generička klasa ili dekorator (`PresetDecorator`) koji prima `**kwargs` | pozivatelj, ime, tip, rječnik |
| **A2** | `ClassLoader` (`wattleflow.helpers.system`) | niz s putanjom klase |
| **A3** | `AttributeException` | pogreška s pozivateljem i uzrokom |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 provjerava konfiguraciju → `evaluate`, `allowed`, `mandatory`, `get`, `optional` |
| **EV02** | A1 traži ime za zapis → `NameHelper.*` |
| **EV03** | A1 čita atribut neovisno o `__dict__`/slotovima → `get_attr` |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Attribute, NameHelper

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "GenericWorkflow, GenericPipeline, GenericProcessor, Strategy" as A1
  actor "ClassLoader" as A2
    usecase "Check configuration" as EV01
    usecase "Get name for record" as EV02
    usecase "Read attribute (__dict__ / slots)" as EV03
  A1 --> EV01
  A2 --> EV01
  A1 --> EV02
  A1 --> EV03
}
@enduml
```

## 07. Constraints and Preconditions

1. Pozivatelj je objekt s imenom (`name`) ili običan objekt; `exists` zahtijeva `IWattleflow`.
2. Za učitavanje iz niza `ClassLoader` je dostupan i klasa ima oblik koji ovaj očekuje.
3. Rječnik `kwargs` predan kao `**kwargs` je **kopija**: izmjena ne vidi pozivatelj.

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | `evaluate`: ako tip stoji, nema učinka; inače `AttributeException` s imenom pozivatelja, nađenim i očekivanim tipom |
| 2 | `allowed`: ključ izvan popisa daje `AttributeException("Restricted: …")`; prazan popis zabranjuje svaki ključ; inače `True` |
| 3 | `mandatory`: ključ je obvezan; vrijednost tipa `cls` postavlja se na pozivatelja i vraća se `True`; za `IWattleflow` tip i niz klasa se učitava (podtip se provjerava prije instanciranja) i instanca se sprema |
| 4 | `get` i `optional`: `get` uklanja ključ iz rječnika, `optional` ga čita; niz se učitava kao klasa, instanca se sprema na pozivatelja; `None` je odsutno |

Cjelovit tok s alternativama prikazan je u odjeljku 09.

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Attribute, NameHelper

  participant "GenericProcessor | Strategy" as G
  participant "Attribute" as A
  participant "ClassLoader" as L
  G -> A : allowed(caller, allowed, **kw)
  activate A
  A --> G : True, False or AttributeException
  deactivate A
  G -> A : mandatory(caller, name, cls, **kw)
  activate A
  alt value is instance of cls
  A -> G : setattr(caller, name, obj)
  activate G
  deactivate G
  else cls is IWattleflow type and value is string
  A -> A : load_from_class(name, obj, cls)
  A -> L : ClassLoader(obj, **kw).instance
  activate L
  L --> A : instance
  deactivate L
  A -> G : setattr(caller, name, instance)
  end
  A --> G : True
  deactivate A
@enduml
```

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `allowed` je `None` | `False`, bez iznimke |
| `allowed` je prazan popis, a `kwargs` nisu prazni | `AttributeException`: svaki ključ je zabranjen; bez ključeva `True` |
| ključ izvan dopuštenih | `AttributeException` s popisom |
| `mandatory`: vrijednost je niz, a `cls` nije `IWattleflow` tip | `AttributeException` (pogrešan tip), bez uvoza modula |
| `ClassLoader` ne nađe modul | `AttributeException` s uzrokom `ModuleNotFoundError` |
| učitana klasa nije podtip `cls` | `AttributeException` prije instanciranja, konstruktor se ne izvršava |
| `get` ili `optional`: vrijednost pogrešnog tipa | `AttributeException`, ne tihi `None` |
| `get` bez tipa | vrijednost se vraća kakva jest |
| `get` ili `optional` s `None` | odsutno: `None` ili zadana vrijednost; obvezan `None` je greška |
| ključ u `get` nije predan, a `mandatory` je `True` | `AttributeException` |
| `NameHelper.source_name` i atribut digne iznimku | `None` |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Attribute, NameHelper

start
:Attribute.allowed(caller, allowed, **kw);
if (allowed is None?) then (yes)
:return False;
else (no)
if (key outside allowed? (empty list: any key)) then (yes)
  :AttributeException (Restricted);
  stop
else (no)
  :return True;
endif
endif
:Attribute.mandatory(caller, name, cls, **kw);
if (name in kw?) then (no)
:AttributeException;
stop
endif
if (value is instance of cls?) then (no)
if (value is a string and cls is an IWattleflow type?) then (no)
  :AttributeException (wrong type);
  stop
endif
:load_from_class(name, value, cls);
note right: the class is checked before it is instantiated
if (class is not a subclass of cls, or loading fails?) then (yes)
  :AttributeException;
  stop
endif
endif
:setattr(caller, name, value or instance);
:return True;
stop
@enduml
```

## 10. State Machine

Nije primjenjivo: `Attribute` i `NameHelper` nemaju konstruktor ni stanje (pretraga `__init__` i `self._` u `helpers.py`).

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) | kriterij 7 |
| [`NFRQ-ORG-08`](../03-NFRQ/NFRQ-ORG-08-deduplication.md) | kriterij 8: dvije kopije trojke imena (`NameHelper` je namjerna lokalna kopija zbog ciklusa) |
| [`NFRQ-SEC-02`](../03-NFRQ/NFRQ-SEC-02-attack-surface.md) | `PresetDecorator` provodi `allowed`; ulazna površina za konfiguraciju |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | `ClassLoader` se uvozi tek pri pozivu iz `wattleflow.helpers.system` |

## 12. Results

Generičke klase provjeravaju konfiguraciju na jednom mjestu i uvijek dobivaju isti oblik pogreške, a
audit zapisi uvijek dobivaju ime, bez ikakve opasnosti od iznimke.

## 13. Acceptance Criteria

1. `evaluate` i `allowed` dižu `AttributeException` s pozivateljem. ✅
2. `NameHelper.source_name` ne diže iznimku. ✅
3. `get_attr` čita i iz `__dict__` i iz slotova po MRO-u. ✅
4. Modul deklarira `__all__`; klase nemaju stanje. ✅
5. `convert` vraća pretvorenu vrijednost (`Enum` član po imenu ili vrijednosti, ili `cls(value)`) i ne dira pozivateljev rječnik. ✅
6. `mandatory` za naziv klase sprema učitanu instancu na pozivatelja, prosljeđuje pozivatelja učitavanju (pa ga nosi i uzročna iznimka) i ispravno formatira poruku (`kwargs['s']`). ✅
7. Metoda koja referira vlastitu klasu je `@classmethod` ([`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md)); izuzetak c.1: javne metode `exists`, `get`, `load_from_class`, `mandatory` i `optional` vežu domenski tip na parametar `cls` i ostaju `@staticmethod` (odgođeno, DEF-HLP-09); privatni `_resolve` je `@classmethod`. ✅
8. Isto ime ima jedan izvor. ❌ — DEF-HLP-04
9. `get`: bez tipa vraća bilo koji objekt (obvezni `None` je greška); naziv klase učitava, provjerava prema tipu i sprema na pozivatelja; neobavezna vrijednost pogrešnog tipa ili klase pogrešnog tipa diže iznimku, ne tihi `None`. ✅
10. `optional`: ključ prisutan s `None` je odsutan (uzima se zadana vrijednost ili ništa), ne naziv klase. ✅
11. `exists`: odsutnim se smatra samo `None` (`0`, `[]` i `False` su legitimne vrijednosti), a pripadnost obitelji `IWattleflow` provjerava se prije čitanja atributa. ✅
12. `evaluate`: nema prečaca za samu klasu; `list` nije `list`. ✅
13. `allowed`: prazan popis zabranjuje svaki ključ (bez ključeva je zadovoljen); sama klasa `list` umjesto popisa diže `AttributeException`. ✅
14. `print_all` i `print_prop` ništa ne vraćaju (obična petlja, ne popis `None`-ova). ✅
15. Svaka javna metoda ima izmjeren trošak, a medijan po pozivu ostaje unutar proračuna od 200 000 ns; najsporija je `list_dir`, ispod 10 µs. ✅
16. `load_from_class` razrješava klasu i provjerava je li podtip traženog tipa **prije** instanciranja; klasa pogrešnog tipa ne izvršava konstruktor. `mandatory` učitava klasu iz niza samo za `IWattleflow` tipove, a bez tipa `get` vraća vrijednost kakva jest. ✅
17. Argument pozivatelja u `**kwargs` koji se zove kao argument iznimke (`error`, `obj`, …) ne skriva grešku (`AttributeException`, ne `TypeError`). ✅
18. `print_all`, `print_prop` i `list_vars` rade i nad objektima sa `__slots__`; `NameHelper.owner` nikad ne diže iznimku. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–4 | pregled `helpers.py` | zadovoljeno |
| 5 | `workflow/tests/test_attribute_convert.py` (10 testova) | prije ispravka pada 5, nakon njega 10/10 |
| 6, 9–14, 16–18 | `workflow/tests/test_attribute_helpers.py` (65 testova, po metodi) | prije ispravaka padalo je 21 od tadašnjih 47 testova; prije spajanja s predloženom izvedbom 11 od 75 novih; nakon toga 75/75 uz `convert`; `workflow/tests` 239/239 |
| 7 | `workflow/tests/test_helpers_structure.py` (4 testa) | nijedan `@staticmethod` ne imenuje vlastitu klasu osim pet odgođenih (izuzetak c.1); popis odgođenih mora odgovarati kodu; mutacija (nova takva referencija) ruši test |
| 8 | usporedba imena | imena u `Attribute` delegiraju `NameHelper` |
| 15 | `workflow/tests/test_helpers_cost.py` (7 testova, oko 2 s) | svih 26 javnih metoda ima slučaj; zaštita pada kad se doda metoda bez slučaja; svi medijani ispod proračuna |
| posljedice | statički prolaz nad `src` | 65 poziva `evaluate`: nijedan ne predaje samu klasu; `blackwattle` skup isti prije i poslije (4 neovisna pada u `tests/metrics`, 34 greške okoline bez `PIL`, `docx`, `numpy`, `pyspark`) |

**Trojka (D-10):** alat — `unittest`, `timeit` i `ast` · kriterij — odjeljak 13 · platforma — CPython 3.12.14, Linux/WSL2,
okruženje `workflow`, dokumentacija `v0.0.5`. **Granica (D-11):** brojanje uporabe je statičko i ne vidi dinamičke pozive; 34 greške
okoline znače da dio `blackwattle` koda u testovima ne radi. Mjerenja, matrice i usporedba: odjeljak „Analiza” na kraju dokumenta.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-HLP-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

- **DEF-HLP-09 — Parametar `cls` u javnim metodama `Attribute`** (`NFRQ-ORG-05`, izuzetak c.1). `exists`, `get`, `load_from_class`,
  `mandatory` i `optional` vežu domenski tip na `cls`, pa ne mogu postati `@classmethod`. Preimenovanje (npr. u `expected`) je odluka o javnom API-ju:
  skripte workflowa pozivaju ih ključnim riječima (`caller=`, `name=`, `cls=`), a 14 takvih poziva postoji u 4 datoteke primjera
  (`blackwattle/examples/workflows`: `01_synthetic_data`, `02_excel_seifa`, `08_youtube`, `27_spark_write`). Pokus s pozicijskim parametrima prošao je
  testove, ali bi slomio taj stil, pa je vraćen. **Čeka odluku** (npr. preimenovanje uz prijelazno razdoblje).

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/helpers.py`
- Analiza: `FRQ-HLP-helpers-ANL.md`
- Testovi: `workflow/tests/test_attribute_convert.py`, `workflow/tests/test_attribute_helpers.py`, `workflow/tests/test_helpers_cost.py`, `workflow/tests/test_helpers_structure.py`

### Analiza metoda na dan 2026-10-04

Cjelovita analiza: `FRQ-HLP-helpers-ANL.md` (matrice troška i uporabe, opis svake metode, usporedba s predloženom izvedbom, odluke).
Skripte i testovi: `test_helpers_cost.py`, `test_attribute_helpers.py`,
`test_attribute_convert.py`.

**Rezultat mjerenja:** 26 javnih metoda `Attribute` i `NameHelper`; medijan po pozivu od 73 do oko 5,7 µs (najsporije `list_dir` i `load_from_class`), proračun
200 µs. Uporaba (statička, `src`): 14 metoda nema korisnika, 7 se koristi samo unutar `helpers.py`; u `workflow` rabe se `evaluate`, `source_name`, `nc` i `nt`.

**Rezultat izmjena:** ispravljene su sve točke iz popisa i `convert`; iz predložene izvedbe preuzeti su zajednički `_resolve`, zaštita od sudara imena, podrška za
`__slots__`, `NameHelper.owner`, omotači imena i rani izlaz u `evaluate`; dodana je provjera klase prije instanciranja. Odluke od 2026-10-04 (A1–A5) zapisane su u analizi, §9: tri ograničenja i metode bez korisnika ostaju kakvi jesu, a `@classmethod` samo za privatni `_resolve`.

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-04 | Otvorene stavke preimenovane u defekte `DEF-HLP-<nn>`; ispravljene sve točke iz popisa i `convert` (testovi prije ispravaka padali); `helpers.py` spojen s predloženom izvedbom (zajednički `_resolve`, zaštita od sudara imena, podrška za `__slots__`, `NameHelper.owner`, omotači imena, rani izlaz u `evaluate`) uz provjeru klase prije instanciranja; zatvoreni DEF-HLP-01, -02, -03, -04…-08 (odluke A1–A4: `mandatory` učitava samo `IWattleflow` tipove, `get` sprema samo učitano, `get` bez tipa vraća niz, metode bez korisnika ostaju; A5: `_resolve` je `@classmethod`, ostalo odgođeno po `NFRQ-ORG-05` c.1, DEF-HLP-09); kriteriji 5, 6 i 9–18; odjeljak 10 nije primjenjivo; sučelje, dijagrami klasa i toka te alternativni tokovi usklađeni s kodom; dijagram toka izbačen iz 08 (cjelovit u 09); mjerenja, matrice i usporedba u `analizi`; DEF-HLP-08 (neodlučene točke). |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: `evaluate(..., expected_type)`; sudionik „Generic class or decorator” zamijenjen klasama iz koda; uvjet grane u slijedu. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — s defektima u `Attribute` (`DEF-HLP-01…04`) |
