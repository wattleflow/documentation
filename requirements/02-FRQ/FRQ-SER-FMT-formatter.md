<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-SER-FMT — Generički formater

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | Formatter je uloga ontologije |
| **Nadređeni zahtjev** | [`HLRQ-10-SERIALISATION`](../01-HLRQ/HLRQ-10-SERIALISATION.md) — `BR-FMT-01`, `BR-FMT-02`, `BR-SER-01`, `BR-SER-02`, `BR-SER-03`, `BR-SER-04` · [`HLRQ-01`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) §3 — `BR-DRV-01`, `BR-PTN-05`, `BR-PTN-07` (`__slots__`) |
| **Predmet** | `GenericFormatter` — sadržaj dokumenta → pohranjivi oblik, na granici formata |
| **Sestrinski** | [`FRQ-SER-PAR`](FRQ-SER-PAR-parser.md) (zrcalna strana) · [`FRQ-SER-CNV`](FRQ-SER-CNV-converter.md) · [`FRQ-STR`](FRQ-STR-strategy.md) (strategija piše kroz formater) |
| **Izvedba** | `workflow/src/wattleflow/concrete/serialisation.py` (`GenericFormatter`, `FormatterError`) |
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

Zrcalo parsera: provjerava obvezni `content`, po izboru mu provjerava tip i predaje ga
specijalizaciji koja implementira `serialise`. `render` **vraća** teret i ništa ne zapisuje —
odredište je posao pozivatelja. Formati koji su po prirodi tokovni nadjačavaju `stream`. Lagana
razina: bez audita, logiranja i preseta; `__slots__ = ()`.


`render` odgovara samo `bytes` ili `str` (inače `ERROR`), a `stream` svaki kvar kodiranja ili upisa nosi kao `ERROR` s uzrokom. Imena `check`, `stream`, `encoding_of` i `serialise` su javna: `stream` je pogodnost nad `render` (nije član `IFormatter`), a `blackwattle` nadjačava `serialise`.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title GenericFormatter

top to bottom direction

  interface "IFormatter<Content>" as IFormatter {
    +render(**kwargs) : bytes | str
  }
  abstract class GenericFormatter {
    {static} CONTENT : type | None = None
    {static} ENCODING : str = "utf-8"
    {static} SUFFIX : str = ""
    {static} ERROR : type[Exception] = FormatterError
    {static} ERRORS : tuple[type[BaseException], ...] = (FormatterError,)
    +name : str
    +render(**kwargs) : bytes | str
    +check(content : Content) : None
    +stream(handle : BinaryIO, content : Content, **kwargs) : None
    +encoding_of(**kwargs) : str
    {abstract} +serialise(content : Content, **kwargs) : bytes | str
  }
  class FormatterError {
    +caller : object | None
    +error : str
  }
IFormatter <|.down. GenericFormatter
GenericFormatter .right.> FormatterError : ERROR
@enduml
```

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title GenericFormatter

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "GenericFormatter", "Writing side of the format boundary")
System_Ext(po, "Caller", "Write strategy")
System_Ext(pt, "Stream caller", "Writes to a binary stream")
System_Ext(sp, "Specialisation", "One class per format")
Rel_L(po, sys, "Calls render")
Rel_R(pt, sys, "Calls stream")
Rel_U(sp, sys, "Implements serialise")
@enduml
```

</div>

## 04. User Diagram

| oznaka | tko | ulazi s |
|---|---|---|
| **A1** | pozivatelj (strategija pisanja, pipeline) | `content=` i opcije formata |
| **A2** | specijalizacija (jedna klasa po formatu) | implementira `serialise`; deklarira `SUFFIX`, po izboru `CONTENT` |
| **A3** | pozivatelj tokom (`stream`) | otvoren binarni tok i sadržaj |

Dijagram korisnika: slijedi.

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 zove `render(content=…, **opcije)` |
| **EV02** | A3 zove `stream(handle, content, **opcije)` |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title GenericFormatter

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Caller" as A1
  actor "Specialisation" as A2
  actor "Stream caller" as A3
    usecase "Render content (render)" as EV01
    usecase "Write to stream (stream)" as EV02
  A1 --> EV01
  A2 --> EV01
  A3 --> EV02
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. `content` je obvezan ključ; vrijednost `None` smije proći ako `CONTENT` nije zadan (format koji prima sve).
2. Ako je `CONTENT` zadan, ulaz mora biti njegova instanca; inače `ERROR`.
3. `serialise` vraća `bytes` ili `str` (pa i `bytearray`/`memoryview`); drugo je `ERROR`.
4. `stream` nije član `IFormatter`; to je pogodnost nad `render`, a tokovni formati ga nadjačavaju.
5. `check`, `stream`, `encoding_of` i `serialise` su kuke s javnim imenima (zapisana odluka).

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | `render` provjerava prisutnost `content` i izvlači ga iz `kwargs` |
| 2 | ako je `CONTENT` deklariran, `check` traži `isinstance`; inače se tip ne provjerava |
| 3 | `serialise(content, **opcije)` vraća `bytes` ili `str` |
| 4 | `stream` poziva `render`, `str` kodira (`encoding=` → `ENCODING`) i piše u tok |



### Dijagram slijeda

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title GenericFormatter

  participant "Caller" as P
  participant "Specialisation" as S
  participant "BinaryIO" as H
  participant "GenericFormatter" as F
  activate P
  P -> F : render(content=..., **kwargs)
  activate F
  F -> F : "content" in kwargs?
  opt CONTENT declared
    F -> F : check(content)
  end
  F -> S : serialise(content, **kwargs)
  activate S
  S --> F : bytes or str
  deactivate S
  F --> P : payload
  deactivate F
  P -> F : stream(handle, content, **kwargs)
  activate F
  F -> F : encoding_of(**kwargs)
  F -> F : render(content=content, **kwargs)
  F -> H : write(payload)
  activate H
  deactivate H
  deactivate F
  deactivate P
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| nema `content` | `ERROR` („mandatory 'content' not found”) |
| `CONTENT` zadan, ulaz pogrešnog tipa | `ERROR` s nađenim i očekivanim tipom |
| `serialise` digne iznimku | `ERROR` s uzrokom; iznimka iz `ERRORS` prolazi nepromijenjena |
| `serialise` vrati nešto što nije `bytes`/`str` | `ERROR` koji imenuje tip |
| `stream`: tekst se ne može kodirati ili upis padne | `ERROR` s uzrokom (ne sirova `UnicodeEncodeError`/`OSError`) |
| `stream` s `encoding=` | `str` se kodira tim kodiranjem, inače `ENCODING` razreda; `bytes` se zapisuju kakvi jesu |

### Dijagram toka

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title GenericFormatter

start
:render(content=..., **kwargs);
if (content present?) then (no)
  :<b><color:red>FAILED: ERROR, mandatory content missing</color></b>;
  kill
endif
if (CONTENT declared and type mismatch?) then (yes)
  :<b><color:red>FAILED: ERROR (found and expected type)</color></b>;
  kill
endif
:serialise(content, **kwargs);
if (exception?) then (yes)
  if (exception in ERRORS?) then (yes)
    :<b><color:red>FAILED: propagates unchanged</color></b>;
    kill
  endif
  :<b><color:red>FAILED: ERROR from the cause</color></b>;
  kill
endif
if (payload is bytes or str?) then (no)
  :<b><color:red>FAILED: ERROR, wrong payload type</color></b>;
  kill
endif
if (called as stream?) then (yes)
  :encode str (encoding= or ENCODING), write to the handle;
  if (encoding or write failed?) then (yes)
    :<b><color:red>FAILED: ERROR from the cause</color></b>;
    kill
  endif
endif
:return the payload;
stop
@enduml
```

</div>

## 10. State Machine

Slijedi. Zatečeni dokument nema ovog odjeljka.

## 11. Non-Functional Requirements

| NFR | posljedica |
|---|---|
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | generička razina ne uvozi treću stranu |
| [`NFRQ-ORG-13`](../03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) | javna površina je sučelje — vidi odjeljak 15 t.2 |
| [`NFRQ-FUN-01`](../03-NFRQ/NFRQ-FUN-01-conversion-fidelity.md) | vjernost pretvorbe mjeri se u specijalizacijama, ne ovdje |
| [`NFRQ-MEM-01`](../03-NFRQ/NFRQ-MEM-01-memory.md) | `BR-PTN-07`: klasa deklarira `__slots__` ili ima zapisanu iznimku; vidi odjeljak 15 |

## 12. Results

Pozivatelj dobiva gotov teret ili je jasno odbijen; formater nikad ne zna kamo se piše.

## 13. Acceptance Criteria

1. `content` je obvezan; tip se provjerava kad je `CONTENT` zadan. ✅
2. `render` vraća teret (`bytes` ili `str`) i ništa ne zapisuje. ✅
3. Kvar `serialise` stiže kao `ERROR` s uzrokom; iznimka iz `ERRORS` prolazi nepromijenjena. ✅
4. Teret koji nije `bytes`/`str` odbija se kao `ERROR`. ✅
5. `stream` kodira tekst prema pozivu pa razredu i svaki kvar kodiranja ili upisa nosi kao `ERROR` s uzrokom. ✅
6. Ne nasljeđuje `Wattleflow`; nema audita; `__slots__ = ()`. ✅

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1, 2 | `workflow/tests/test_serialisation.py` (`FormatterTest`) | obvezni `content`; ulaz bez `CONTENT` prolazi i `None`; zadani tip se provodi; opcije bez `content` stižu u `serialise` |
| 3 | `FormatterTest` | uzrok i poruka; vlastita obitelj prolazi |
| 4 | `FormatterTest` | `None`, broj, lista odbijeni; `bytearray` dopušten |
| 5 | `StreamTest` | `latin-1` iz poziva i iz razreda, `bytes` kakvi jesu, nekodirljiv tekst i neispravno odredište su `ERROR` s uzrokom |
| 6 | `SlotsTest` | prazni slotovi |
| mutacije | ručno | bez provjere tereta, `stream` bez omota — svaka ruši test |

**Trojka reproducibilnosti (D-10):** alat — `unittest`; kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2, okruženje `workflow`.

## 15. Open issues

Stavke su defekti i nose oznaku `DEF-FMT-<nn>`; zatvoren defekt se uklanja, a oznaka se ne preuzima.

Nema otvorenih stavki.

## 16. References

- Izvedba: `workflow/src/wattleflow/concrete/serialisation.py` (`GenericFormatter`, `FormatterError`)
- Testovi: `workflow/tests/test_serialisation.py`

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml`; dijagram toka iz odjeljka 08 uklonjen (nosi ga odjeljak 09). |
| v0.0.5 | 2026-10-04 | Prvi put izvršeno (`test_serialisation.py`, 40 testova za sva tri dokumenta), svi defekti zatvoreni: `DEF-FMT-01` `CONTENT` neobvezan zapisano kao svojstvo (format koji prima sve); `-02` javna imena kuka zapisana kao odluka; `-03` `content=None` ide kroz provjeru samo kad `CONTENT` nije zadan; `-04` docstring više ne spominje nepostojeći `GenericFormatterAudit`; `-05` `__slots__ = ()` bez učinka na `__dict__` zapisano. **Novo:** `render` je prihvaćao bilo kakav povrat `serialise` (npr. `None`), a `stream` je nad njim dizao sirovi `TypeError` — sada `ERROR`; nekodirljiv tekst i neispravno odredište u `stream` su `ERROR` s uzrokom. Kriteriji 4 i 5, mutacije. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); unutarnje reference preusmjerene. |
| v0.0.5 | 2026-10-03 | Uklonjena imena klasa i poveznice izvan distribucije `workflow`. |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: tipovi i potpisi, odredište write je BinaryIO. |
| v0.0.5 | 2026-10-02 | Dodane aktivacije u dijagram slijeda. |
| v0.0.5 | 2026-10-02 | `Verzija` zamjenjuje `Status`; prijašnji status: Provedeno u kodu — obrnuto inženjerstvo zatečenog |
