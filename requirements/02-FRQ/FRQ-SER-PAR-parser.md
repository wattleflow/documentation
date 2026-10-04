<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SER-PAR — Generički parser

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-10-SERIALISATION`](../01-HLRQ/HLRQ-10-SERIALISATION.md) — `BR-PAR-01`, `BR-PAR-02`, `BR-SER-01`, `BR-SER-02`, `BR-SER-03`, `BR-SER-04` · [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §3 — `BR-DRV-01`, `BR-PTN-05`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericParser` — pohranjeni oblik → sadržaj dokumenta, na granici formata |
| **Sestrinski** | [`FRQ-SER-FMT`](FRQ-SER-FMT-formatter.md) (suprotni smjer) · [`FRQ-SER-CNV`](FRQ-SER-CNV-converter.md) (drži parser u strategiji) · [`FRQ-DRV`](FRQ-DRV-driver.md) |
| **Izvedba** | `workflow/src/wattleflow/concrete/serialisation.py` (`GenericParser`, `ParserError`) |
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

Sučelje `IParser.parse` ne propisuje prijenos. Generička klasa propisuje **jedan**: točno jedan
deklariran izvor pretvara u binarni čitač i predaje ga specijalizaciji. Politika izvora živi ovdje,
jednom; specijalizacija ne otvara, ne razrješava i ne provjerava putanju. Lagana razina: bez
audita, logiranja i preseta; `__slots__ = ()`.


Izvor se provjerava po vrsti, ne samo po prisutnosti: `stream` mora imati `read`, `path` mora biti `str` ili `PathLike` (cijeli broj `open` bi uzeo za deskriptor datoteke i zatvorio ga), `payload` mora biti bytes-like (`None` i tekst se odbijaju). Imena `reader`, `decode` i `deserialise` su javne kuke specijalizacije: `blackwattle` nadjačava `reader` i zove `decode`, pa preimenovanje nije dio ovog zahtjeva.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericParser

top to bottom direction

  interface "IParser<Content>" as IParser {
    +parse(**kwargs) : Content
  }
  abstract class GenericParser {
    {static} ENCODING : str = "utf-8"
    {static} SOURCES : tuple[str, ...] = ("stream", "path", "payload")
    {static} ERROR : type[Exception] = ParserError
    {static} ERRORS : tuple[type[BaseException], ...] = (ParserError,)
    +name : str
    +parse(**kwargs) : Content
    +reader(kwargs : dict) : Iterator[BinaryIO]
    +decode(reader : BinaryIO, **kwargs) : str
    {abstract} +deserialise(reader : BinaryIO, **kwargs) : Content
  }
  class ParserError {
    +caller : object | None
    +error : str
  }
IParser <|.down. GenericParser
GenericParser .right.> ParserError : ERROR
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericParser

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "GenericParser", "Reading side of the format boundary")
System_Ext(po, "Caller", "Driver, pipeline")
System_Ext(sp, "Specialisation", "One class per format")
System_Ext(fs, "File system", "Source path")
Rel_L(po, sys, "Calls parse")
Rel_R(sp, sys, "Implements deserialise")
Rel_U(sys, fs, "Reads path")
@enduml
```

</div>

## 04. User Diagram

| oznaka | tko | ulazi s |
|---|---|---|
| **A1** | pozivatelj (driver, pipeline, strategija) | točno jedan od `stream=`, `path=`, `payload=` te opcije formata |
| **A2** | specijalizacija (jedna klasa po formatu ili motoru) | implementira `deserialise` |
| **A3** | konfiguracija klase | `ENCODING`, `SOURCES`, `ERROR`, `ERRORS` |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 zove `parse(**kwargs)` |
| **EV02** | A2 zove `decode(reader, **kwargs)` unutar `deserialise` (tekstni formati) |

## 06. Use Case Diagrams

<p align=center>

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericParser

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Caller" as A1
  actor "Specialisation" as A2
  actor "Class configuration" as A3
    usecase "Parse source" as EV01
    usecase "Decode text" as EV02
  A1 --> EV01
  A2 --> EV01
  A3 --> EV01
  A2 --> EV02
}
@enduml
```

</div>

</p>

## 07. Constraints and Preconditions

1. Zadan je **točno jedan** izvor iz `SOURCES` (ključ prisutan); vrijednost mora biti ispravne vrste (`stream` s `read`, `path` kao `str`/`PathLike`, `payload` bytes-like).
2. Specijalizacija implementira `deserialise`; klasa je inače apstraktna.
3. `stream` je otvoren i čitljiv; vlasnik ostaje pozivatelj.
4. `reader`, `decode` i `deserialise` su kuke specijalizacije s javnim imenima (zapisana odluka; `NFRQ-ORG-13` o prefiksu `_` ne primjenjuje se na kuke koje specijalizacije u `blackwattle` već nadjačavaju).
5. `decode` ne troši `encoding` iz pozivateljevih opcija (radi na kopiji); ključ ostaje dostupan daljnjim pozivima. Bez posljedica.

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | `reader` broji deklarirane izvore; različito od jedan → `ERROR` |
| 2 | ključ izvora izvlači se iz `kwargs`, pa `deserialise` prima samo opcije formata |
| 3 | `stream` posuđen bez zatvaranja · `payload` omotan u `BytesIO` · `path` otvoren `rb` i zatvoren po izlasku |
| 4 | `deserialise(reader, **opcije)` vraća sadržaj |
| 5 | `decode` čita sve i dekodira; kodiranje redom **poziv (`encoding=`) → klasa (`ENCODING`)** |



### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericParser

  participant "Caller" as P
  participant "Specialisation" as S
  participant "GenericParser" as G
  activate P
  P -> G : parse(path=..., **kwargs)
  activate G
  G -> G : reader(): exactly one source
  G -> G : open(path, rb)
  G -> S : deserialise(reader, **kwargs)
  activate S
  S --> G : content
  deactivate S
  G -> G : close reader
  G --> P : content
  deactivate G
  deactivate P
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| nijedan ili više izvora | `ERROR` (`ParserError`) s popisom pronađenih |
| izvor pogrešne vrste (`payload=None`, tekst, `stream` bez `read`, `path` kao broj) | `ERROR` koji imenuje izvor i njegov tip; ništa se ne otvara |
| `path` ne postoji ili nečitljiv | iznimka otvaranja omotana u `ERROR` s uzrokom (`BR-PTN-05`) |
| kvar u `deserialise` | omotan u `ERROR` (`raise … from e`) |
| iznimka iz `ERRORS` | prolazi nepromijenjena (vlastita greška razine) |
| nečitljivi bajtovi u `decode` | `ERROR` s uzrokom |
| specijalizacija izvan korijena iznimaka | deklarira vlastite `ERROR` i `ERRORS` |
| proširenje izvora | specijalizacija nadjačava `reader` |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericParser

start
:parse(**kwargs);
if (exactly one of SOURCES?) then (no)
  :<b><color:red>FAILED: ERROR listing the sources found</color></b>;
  kill
endif
if (source of the right kind?) then (no)
  note right: stream with read, path as str or PathLike,\npayload as bytes-like
  :<b><color:red>FAILED: ERROR naming the source and its type</color></b>;
  kill
endif
if (which source?) then (stream)
  :borrowed reader, not closed;
elseif (payload) then
  :BytesIO(payload);
else (path)
  :open(path, rb), closed afterwards;
endif
:deserialise(reader, **kwargs);
if (exception?) then (yes)
  if (exception in ERRORS?) then (yes)
    :<b><color:red>FAILED: propagates unchanged</color></b>;
    kill
  endif
  :<b><color:red>FAILED: ERROR from the cause</color></b>;
  kill
endif
:return content;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | generička razina ne uvozi treću stranu; motor se uvozi unutar metode specijalizacije ([`NFRQ-SEC-17`](../03-NFRQ/NFRQ-SEC-17-deferred-third-party-loading.md)) |
| [`NFRQ-ORG-13`](../03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) | javna površina je sučelje — vidi odjeljak 15 t.2 |
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | lagana razina ne zapisuje; kvar koji omata ne ostavlja `debug` trag; `wem_lint --select OBS-01` javlja `parse: raise without trace` (`serialisation.py:118`) |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Specijalizacija vidi uvijek isti oblik ulaza (binarni čitač i čiste opcije), a svaki kvar izlazi kao
jedna klasa iznimke s uzrokom.

## 13. Acceptance Criteria

1. Prihvaća točno jedan izvor; nula ili više se odbija. ✅
2. `stream` se ne zatvara, `path` se zatvara, `payload` ide kroz memorijski međuspremnik. ✅
3. Izvor se provjerava po vrsti: `payload=None`, tekst, `stream` bez `read` i `path` kao broj daju `ERROR` (cijeli broj se ne uzima za deskriptor). ✅
4. Kvar specijalizacije stiže kao `ERROR` s uzrokom; iznimka iz `ERRORS` prolazi nepromijenjena. ✅
5. `encoding=` poziva ima prednost pred `ENCODING` razreda, a ovaj pred `utf-8`; nečitljivi bajtovi su `ERROR`. ✅
6. Ne nasljeđuje `Wattleflow`; nema audita; `__slots__ = ()`. ✅
7. Nema third-party uvoza ([`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md)). ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2 | `workflow/tests/test_serialisation.py` (`ParserSourceTest`) | nula i dva izvora odbijeni; tok nije zatvoren, putanja jest, `payload` iz memorije; izvor se troši, opcije se prosljeđuju |
| 3 | `ParserSourceTest` | `None`, tekst, broj, lista kao `payload`; `None`/tekst/broj kao `stream`; `None`/broj/bytes kao `path`; cijeli broj nije deskriptor (deskriptor ostaje otvoren) |
| 4 | `ParserFailureTest` | uzrok i poruka; `ERRORS` prolazi; vlastiti `ERROR`; `deserialise` apstraktan |
| 5 | `DecodeTest` | zadano, razred, poziv; nečitljivi bajtovi |
| 6, 7 | `SlotsTest`; pregled uvoza | prazni slotovi; `stdlib` + `wattleflow.core` |
| mutacije | ručno | bez provjere izvora, putanja neprovjerena, iznimke iz `ERRORS` omotane — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-PAR-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/serialisation.py` (`GenericParser`, `ParserError`)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka iz odjeljka 08 uklonjen (nosi ga odjeljak 09). |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_serialisation.py`, 40 testova za sva tri dokumenta), svi defekti zatvoreni: Izvor se provjerava po vrsti (`DEF-PAR-01`: `payload=None` je davao prazan čitač; **dodatno: cijeli broj kao `path` `open` je uzimao za deskriptor datoteke i zatvarao ga**); `-02` javna imena kuka zapisana kao odluka (`blackwattle` ih nadjačava); `-03` docstring više ne spominje nepostojeću audit razinu; `-04` `decode` i `encoding` zapisano kao bez posljedica; `-05` `__slots__ = ()` bez učinka na `__dict__` zapisano. Kriteriji 3 i 5, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: tipovi i potpisi, SOURCES kao tuple nizova. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — obrnuto inženjerstvo zatečenog |
