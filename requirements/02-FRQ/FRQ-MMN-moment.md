<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-MMN — Moment

> **Oznaka je provizorna.** Kategorija `MMN` nije u registru; uvođenje traži dokumentiranu izmjenu (D-12).

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Nadređeni zahtjev** | [`HLRQ-MMN`](../01-HLRQ/HLRQ-MMN-moment.md) — `BR-MMN-01…14` |
| **Predmet** | `Moment` (vrijednost), `MomentHelper` (zajedničko), `MomentAwareHelper` (trenutak), `MomentNaiveHelper` (zidno vrijeme), `MomentRangeError` |
| **Izvedba** | `workflow/src/wattleflow/helpers/moment/` (`base.py`, `helper.py`) |
| **Dijagrami** | inline (02, 03, 06, 08, 09) — pogled, ne izvor istine (D-13) |

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

`Moment` je nepromjenjiva vrijednost: nanosekunde od 1970-01-01 (`ns`), vrsta (`aware`) i zona
prikaza (`tz`, samo za trenutak). Gradi se izravno ili kroz pomoćni razred svoje vrste, a jedina
točka nastanka (`__new__`, `_raw`) provjerava domenu (`BR-MMN-05`). Pomoćni razredi imaju samo
metode razreda: ulaz u `Moment`, izlaz u druge oblike i serijalizaciju.

| uloga | razred | što radi |
|---|---|---|
| vrijednost | `Moment` | čuva `ns`, `aware`, `tz`; jednakost i poredak unutar vrste; zbrajanje i oduzimanje s `timedelta`; razlika dva `Moment`a iste vrste |
| greška raspona | `MomentRangeError` | vrijednost izvan domene; nasljeđuje `OverflowError` i `ValueError`, pa je hvataju oba |
| zajedničko | `MomentHelper` | `configure` i `workflow_zone` (zona workflowa, `BR-MMN-12`), `zone`, `key`, `of` (izbor pomoćnog razreda po vrsti), `to_dict`/`from_dict`, `to_bytes`/`from_bytes`, `to_int64_ns` (most prema stupcima), `text` i `parse_iso` (tekst kad vrstu odlučuje izvor) |
| trenutak | `MomentAwareHelper` | ulaz: `moment`, `from_iso`, `from_str`, `from_unix`, `from_unix_ns`, `now`; izlaz: `to_datetime`, `to_utc`, `to_local`, `to_str`, `to_iso`, `to_unix_ns`, `to_unix`, `to_date`, `to_time`, `to_struct`, `to_rfc2822`, `to_http`, `to_jd`, `to_mjd`; zona: `with_tz`, `strip` |
| zidno vrijeme | `MomentNaiveHelper` | ulaz: `moment`, `from_iso`, `from_str`; prijelaz u trenutak: `localize(m, zone)` s obveznom zonom; izlaz: `to_datetime`, `to_str`, `to_iso`, `to_date`, `to_time`, `to_struct`; nema `to_rfc2822`, Unix, HTTP, JD ni prijelaza u trenutak |

Goli `int` pomoćni razred jedne vrste čita kao `ns` te vrste: UTC za trenutak, zidno vrijeme za
zidno vrijeme. `MomentHelper` goli `int` odbija jer mu ne zna vrstu.

**Atributi razreda (`ClassVar`).**

| atribut | značenje | postavlja | čita ga |
|---|---|---|---|
| `Moment.MIN_NS` | `-62_135_596_800·10⁹` = 0001-01-01T00:00:00 | `Moment` | `__new__`, `_raw` |
| `Moment.MAX_NS` | `253_402_300_800·10⁹ − 1` = 9999-12-31T23:59:59.999999999 | `Moment` | `__new__`, `_raw` |
| `MomentHelper.AWARE` | `None` (bilo koja vrsta), `True`, `False` | svaki pomoćni razred | provjera vrste ulaza |
| `MomentHelper.ENV_ZONE` | `WATTLEFLOW_TIME_ZONE`: globalna postavka zone workflowa | `MomentHelper` | `workflow_zone` |
| `MomentHelper.INT64_MIN_NS`, `INT64_MAX_NS` | `-2⁶³ + 1`, `2⁶³ − 1` (`-2⁶³` je `NaT`) | `MomentHelper` | `to_int64_ns` |

## 02. Class Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Moment and its helpers

top to bottom direction

class Moment {
  {static} MIN_NS : int
  {static} MAX_NS : int
  +ns : int
  +aware : bool
  +tz : str | None
  #{static} _raw(ns, aware, tz) : Moment
}
class MomentRangeError
class OverflowError
class ValueError
abstract class MomentHelper {
  {static} AWARE : bool | None
  {static} INT64_MIN_NS : int
  {static} INT64_MAX_NS : int
  +{static} of(x) : type[MomentHelper]
  +{static} to_bytes(x) : bytes
  +{static} from_bytes(b) : Moment
  +{static} to_int64_ns(x) : int
}
class MomentAwareHelper {
  {static} AWARE = True
  +{static} moment(x, tz) : Moment
  +{static} to_datetime(x, tz) : datetime
  +{static} to_rfc2822(x, tz) : str
  +{static} strip(x, tz) : Moment
}
class MomentNaiveHelper {
  {static} AWARE = False
  +{static} moment(x) : Moment
  +{static} to_datetime(x) : datetime
}

OverflowError <|-- MomentRangeError
ValueError <|-- MomentRangeError
MomentHelper <|-- MomentAwareHelper
MomentHelper <|-- MomentNaiveHelper
MomentAwareHelper ..> Moment : builds (_raw)
MomentNaiveHelper ..> Moment : builds (_raw)
MomentAwareHelper ..> MomentNaiveHelper : strip, one way
Moment ..> MomentRangeError : outside MIN_NS..MAX_NS
MomentHelper ..> MomentRangeError : outside int64
@enduml
```

</div>

## 03. Context Diagram

<div align="center">

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Moment

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

System(sys, "Moment", "wattleflow.helpers.moment: time as nanoseconds since 1970")
System_Ext(po, "Caller", "Parser, strategy, pipeline, driver")
System_Ext(sl, "Standard library", "datetime, zoneinfo, email.utils, struct")
System_Ext(tz, "IANA zone database", "Through zoneinfo")
System_Ext(col, "Column consumer", "numpy, pandas, Arrow: int64 ns")

Rel_D(po, sys, "moment, from_*, to_*")
Rel_D(sys, sl, "Wraps")
Rel_R(sys, tz, "Reads zones")
Rel_D(sys, col, "to_int64_ns")
@enduml
```

</div>

## 04. User Diagram

| oznaka | tko | ulazi s |
|---|---|---|
| **A1** | pozivatelj (parser, strategija, pipeline, driver) | ulaz vremena: `datetime`, tekst, broj, bajtovi, `dict` |
| **A2** | pomoćni razred jedne vrste | ulaz i tražena zona |
| **A3** | `Moment` | `ns`, vrsta, zona |
| **A4** | standardna biblioteka i IANA baza zona | `datetime`, `zoneinfo`, `email.utils` |
| **A5** | potrošač stupaca | `int64` nanosekundi |

## 05. Events

| oznaka | trigger |
|---|---|
| **EV01** | A1 gradi `Moment` iz ulaza (`moment`, `from_iso`, `from_str`, `from_unix`, `from_unix_ns`, `now`, `from_dict`, `from_bytes`) |
| **EV02** | A1 traži izlaz (`to_datetime`, `to_iso`, `to_str`, `to_rfc2822`, `to_http`, `to_jd`, `to_dict`, `to_bytes`) |
| **EV03** | A1 prevodi trenutak u zidno vrijeme (`strip`) ili mijenja zonu prikaza (`with_tz`) |
| **EV04** | A1 računa s `Moment`om (`+`, `−` s `timedelta`; razlika; usporedba) |
| **EV05** | A1 predaje vrijednost stupcu (`to_int64_ns`) |
| **EV06** | pokretač workflowa postavlja zonu workflowa jednom (`configure`); bez pokretača razrješava se pri prvoj upotrebi |

## 06. Use Case Diagrams

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Moment

left to right direction
skinparam nodesep 10
skinparam ranksep 20

package {
  actor Caller as CALLER
  actor ColumnConsumer as COL

  usecase "Build a Moment" as EV01
  usecase "Write a form" as EV02
  usecase "Strip or change zone" as EV03
  usecase "Compute" as EV04
  usecase "Hand to a column" as EV05

  CALLER --> EV01
  CALLER --> EV02
  CALLER --> EV03
  CALLER --> EV04
  EV05 --> COL
  CALLER --> EV05
}
@enduml
```

</div>

## 07. Constraints and Preconditions

1. Ulaz trenutka nosi zonu ili pomak; ulaz zidnog vremena ga ne nosi.
2. Ime zone postoji u IANA bazi koju čita `zoneinfo`.
3. Vrijednost je u domeni `MIN_NS…MAX_NS`; inače `MomentRangeError` pri nastanku.
4. Ulaz je u proleptičkom gregorijanskom kalendaru i skali UTC (`HLRQ-MMN` §3).

## 08. Sequence Diagrams

### Normalan tok

| korak | ponašanje |
|---|---|
| 1 | A1 predaje ulaz A2 svoje vrste (`EV01`) |
| 2 | A2 provjerava vrstu ulaza; ulaz druge vrste je `TypeError` |
| 3 | A2 računa `ns` i zonu te gradi A3 kroz `_raw`, koji provjerava domenu |
| 4 | Za izlaz (`EV02`) A2 piše A3 kroz A4 u zoni izlaza; za stupac (`EV05`) `to_int64_ns` vraća cijeli broj unutar int64 |

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Moment: build and hand to a column

participant "A1 Caller" as A1
participant "A2 MomentAwareHelper" as A2
participant "A3 Moment" as A3
participant "A5 Column consumer" as A5

activate A1
A1 -> A2 : moment(datetime with zone)
activate A2
A2 -> A2 : check kind, compute ns and zone
A2 -> A3 : _raw(ns, True, tz)
activate A3
A3 -> A3 : MIN_NS <= ns <= MAX_NS
A3 --> A2 : Moment
deactivate A3
A2 --> A1 : Moment
A1 -> A2 : to_int64_ns(Moment)
A2 -> A2 : INT64_MIN_NS <= ns <= INT64_MAX_NS
A2 --> A1 : int
deactivate A2
A1 -> A5 : int64 ns
deactivate A1
@enduml
```

</div>

## 09. Flow Chart Diagrams

### Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| ulaz druge vrste (zidno vrijeme pomoćnom razredu trenutka i obrnuto) | `TypeError` koji imenuje očekivanu vrstu |
| zidno vrijeme sa zonom | `ValueError` |
| nepoznato ime zone | `ZoneInfoNotFoundError` |
| vrijednost izvan `MIN_NS…MAX_NS` (izgradnja, aritmetika, `from_*`, `strip`) | `MomentRangeError` |
| prikaz u zoni pomiče vrijednost izvan godina 1–9999 | `MomentRangeError` (`BR-MMN-06`) |
| `to_int64_ns` izvan int64 ili `-2⁶³` | `MomentRangeError` (`BR-MMN-07`) |
| usporedba ili razlika `Moment`a različitih vrsta | `TypeError` |

### Dijagram toka

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
title Activity Diagram: building a Moment and handing it on

|Helper|
start
:input of one kind;
if (kind matches the helper?) then (no)
  :<b><color:red>TypeError</color></b>;
  kill
endif
:compute ns and zone;
|Moment|
if (MIN_NS <= ns <= MAX_NS?) then (no)
  :<b><color:red>MomentRangeError</color></b>;
  kill
endif
:Moment;
|Helper|
if (to a column?) then (yes)
  if (within int64 and not -2^63?) then (no)
    :<b><color:red>MomentRangeError</color></b>;
    kill
  endif
  :int64 ns;
else (no)
  :datetime, ISO or other form;
endif
stop
@enduml
```

</div>

## 10. State Machine

Nije primjenjivo: `Moment` je nepromjenjiv, a pomoćni razredi nemaju stanje.

## 11. Non-Functional Requirements

| [NFR](../../GLOSSARY.md#abbr-nfr) | posljedica |
|---|---|
| [`NFRQ-DEF-04`](../03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md), [`NFRQ-DEF-05`](../03-NFRQ/NFRQ-DEF-05-moment-naive-helper.md) | dva modela vremena koja se ne miješaju |
| [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) | rubovi `MIN_NS`, `MAX_NS` uključivi; ±1 odbijen |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | samo standardna biblioteka |

## 12. Results

`Moment` unutar domene, ili oblik koji je pozivatelj tražio: `datetime`, tekst (ISO 8601,
RFC 5322, HTTP), broj (Unix, JD), `dict`, bajtovi s vrstom i zonom, ili `int64` za stupac.
Vrijednost izvan domene ne nastaje.

## 13. Acceptance Criteria

1. `Moment` je nepromjenjiv; kopija je isti objekt, a pickle čuva `ns`, vrstu i zonu.
2. Zidno vrijeme ne nosi zonu; nepoznata zona je greška.
3. Zona ne ulazi u jednakost ni `hash`.
4. Usporedba i razlika `Moment`a različitih vrsta daju `TypeError`; pomoćni razred odbija ulaz druge vrste.
5. Trenutak se gradi samo iz ulaza sa zonom ili pomakom; zidno vrijeme samo iz ulaza bez njih.
6. `strip` daje zidno vrijeme u navedenoj ili spremljenoj zoni, uz ispravan pomak ljeti i zimi.
7. `MIN_NS` i `MAX_NS` se prihvaćaju; `MIN_NS − 1` i `MAX_NS + 1` daju `MomentRangeError` u konstruktoru, `_raw`, aritmetici, `from_dict`, `from_bytes` i `from_unix` (uključujući `inf` i `nan`).
8. Prikaz u zoni koji pomiče vrijednost izvan godina 1–9999 (`to_datetime`, `to_iso`, `to_local`, `strip`) daje `MomentRangeError`.
9. `to_int64_ns` vraća `ns` unutar `INT64_MIN_NS…INT64_MAX_NS`; izvan toga i za `-2⁶³` daje `MomentRangeError`.
10. Serijalizacija čuva nanosekunde, vrstu i zonu; `datetime` i ISO zaokružuju na mikrosekundu naniže.
11. `MomentAwareHelper.to_rfc2822` piše pomak zone (`+0000` za UTC); `MomentNaiveHelper` nema `to_rfc2822`.
12. Paket uvozi samo standardnu biblioteku.
13. `now()` nosi zonu workflowa. Zona workflowa je ona predana `configure(zone)`, inače `WATTLEFLOW_TIME_ZONE`, inače IANA ime zone sustava (`TZ`, `/etc/localtime`), inače `UTC` uz jedno upozorenje; nepoznato ime zone je greška.
14. Zona workflowa razrješava se jednom: kasnija promjena okoline ne djeluje dok se `configure` ne pozove ponovno.
15. `MomentHelper.text(x)` piše `Moment` ili `datetime` bilo koje vrste u zadanom ISO obliku (mikrosekunde u cijelosti; UTC kao `Z`): trenutak s pomakom spremljene zone, zidno vrijeme bez pomaka; ništa se ne pretvara.
16. `MomentHelper.parse_iso(text)` vraća trenutak ako tekst nosi pomak ili `Z`, inače zidno vrijeme; tekst koji nije ISO 8601 je `ValueError`.
17. `MomentNaiveHelper.localize(m, zone)` čita zidno vrijeme u navedenoj zoni i vraća trenutak s tom zonom prikaza; zona je obvezna (`BR-MMN-13`, `BR-MMN-14`), a trenutak na ulazu je `TypeError`.

## 14. Verification

| kriterij | metoda | rezultat |
|---|---|---|
| 1–6, 10, 12 | `test_moment.py` (`TestMoment`, `TestAware`, `TestNaive`, `TestCommon`); pregled uvoza | prolazi |
| 7 | `TestDomain.test_the_bounds_are_the_years_of_datetime`, `test_nothing_outside_the_domain_comes_into_being`; mutacija (bez gornje granice) ruši 4 testa | prolazi |
| 8 | `TestDomain.test_a_zone_that_moves_past_the_edge_is_a_range_error` | prolazi |
| 9 | `TestDomain.test_one_bridge_to_int64_columns` | prolazi |
| 11 | `TestAware.test_rfc_5322_and_http`, `TestNaive.test_a_wall_time_has_no_rfc_5322_form` | prolazi |
| 13, 14 | `TestWorkflowZone` (6 testova); `test_workflow_factory.py` (`test_time_zone_sets_the_workflow_zone_once`) | prolazi |
| 15–17 | `TestTextOfEitherKind`; `blackwattle/tests/strategies/test_time_serialisation.py` (JSON, Postgres, `iso_utc`) | prolazi |

**Trojka reproducibilnosti (D-10):** alat — `unittest` (`PYTHONPATH=src:../core/src`); kriterij — odjeljak 13; platforma — CPython 3.12.14, Linux/WSL2. Trošak provjere domene: usporedba iste kopije paketa bez provjere i s njom, četiri izmjenična prolaza po 200 000 zapisa (najbolji od sedam), 2026-10-06: konstruktor `Moment(ns, True)` +~25 ns (~5%; 509–547 naspram 546–570 ns); `from_unix_ns` i zbrajanje s `timedelta` unutar šuma mjerenja. Puni pregled operacija daje `tests/bench_moment.py`.

## 15. Open issues

1. **Proširenje izvan opsega** (skala, kalendar, razlučivost, epoha): `HLRQ-MMN` §7 t.1.
2. **JD i MJD kao jedan `float64`** (~40 µs i ~0,6 µs): `HLRQ-MMN` §7 t.2.
3. **Pomoćni razredi nemaju `core` sučelje**: `HLRQ-MMN` §7 t.3.
4. **Oznaka i kategorija.** `MMN` je provizorna (D-12).

## 16. References

- Izvedba (`workflow/src/wattleflow/helpers/moment/`):
  - [`base.py`](../../../workflow/src/wattleflow/helpers/moment/base.py) (`Moment`)
  - [`helper.py`](../../../workflow/src/wattleflow/helpers/moment/helper.py) (`MomentHelper`, `MomentAwareHelper`, `MomentNaiveHelper`)
- Testovi i mjerenje (`workflow/tests/`):
  - [`test_moment.py`](../../../workflow/tests/test_moment.py) — kriteriji 1–12
  - [`bench_moment.py`](../../../workflow/tests/bench_moment.py) — trošak operacija

## 17. Change history

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.5 | 2026-10-07 | Otvorena stavka o `helpers/dtime.py` zatvorena: `Moment` ga je zamijenio u svim bibliotekama. |
| v0.0.5 | 2026-10-07 | `text` piše zadani oblik (mikrosekunde, `Z`) kao `DateTimeKind`; `localize` s obveznom zonom (odluka D3), kriterij 17. |
| v0.0.5 | 2026-10-07 | `MomentHelper.text` i `parse_iso` (odluka D4), kriteriji 15–16; zamjena za `DateTimeKind`; prijedlog. |
| v0.0.5 | 2026-10-07 | Zona workflowa (`configure`, `workflow_zone`, `ENV_ZONE`), `now` sa zonom workflowa, EV06, kriteriji 13–14 (`BR-MMN-12`); prijedlog. |
| v0.0.5 | 2026-10-06 | Kategorija `MOM` preimenovana u `MMN` (odluka autora). |
| v0.0.5 | 2026-10-06 | Kriteriji 7–9 i 11 provedeni i provjereni (`TestDomain`, uklonjen `MomentNaiveHelper.to_rfc2822`); trošak provjere domene izmjeren usporedbom A/B. |
| v0.0.5 | 2026-10-06 | Prvi zapis: mehanizam `Moment` iz sinteze koda i odluka autora (domena pri nastanku, most prema stupcima, bez RFC 5322 za zidno vrijeme); prijedlog. |
