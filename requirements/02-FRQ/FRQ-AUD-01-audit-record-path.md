<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# FRQ-AUD-01 — Put audit zapisa od komponente do spremišta
> **Oznaka je provizorna.** Kategorija `AUD` nije u vokabularu FR registra; uvođenje traži dokumentiranu izmjenu
> (D-12). Audit je *cross-cutting* sposobnost, ne ontološki primitiv, pa pripada osi sposobnosti
> i nema nadređeni HLRQ.

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Predmet** | Obitelj `logger` temeljnog sloja: sabirnik zapisa `Audit` (`ILogger` + `IObserver`), asinkroni handler `AsyncHandler`, filtri `ContextFilter` i `MeasurementFilter`, tablica formata `LogFormat`, promatrač zapisa (`Audit.observe`) |
| **Izvedba** | `workflow/src/wattleflow/helpers/audit.py` (367 linija, `wattleflow-workflow` v0.0.1.20) |
| **Sljedivost** | `NFRQ-OBS-01/02/03`, [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md), `NFRQ-ORG-07` · D-07 |
| **Sestrinski** | [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) (ključevi zapisa) (gdje na pozivnom lancu zapis nastaje) |
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

Zapis nastaje u komponenti kao **par**: događaj i imenovana polja. Ovaj zahtjev opisuje **put**
tog para do spremišta — što ga sastavlja, tko ga prima i u kojem obliku. *Što* zapis nosi uređuje
[`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md), *gdje na pozivnom lancu* nastaje.

Sabirnik je terminalna karika kooperativnog `__init__` lanca: troši logging argumente i guta
ostatak, pa do `object.__init__` ne stiže ništa.

Konstruktor koristi pet imenovanih argumenata: `formatting` (zadano `LogFormat.DEFAULT`),
`propagate`, `logger`, `handler` i `level`. `level` koji nije naveden (`None`) nije isto što i
`NOTSET`: ne poništava razinu koju je netko drugi postavio za klasu. Klasa je `__slots__`
(`_level`, `_logger`, `_handler`); dijeljeno stanje drže atributi klase `_lock`, `_instances`
(klase kojima je zadani handler već izgrađen) i `_observer`.

## 02. Class Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Class Diagram
title Audit record path

top to bottom direction

  interface ILogger
  interface IObserver
  class Wattleflow
  class "logging.Logger" as L
  class "logging.Handler" as H
  class "logging.Filter" as F
  class Audit {
    +name : str
    +levelname : str
    +debug(msg, *args, **kwargs)
    +info(msg, *args, **kwargs)
    +warning(msg, *args, **kwargs)
    +error(msg, *args, **kwargs)
    +critical(msg, *args, **kwargs)
    +fatal(msg, *args, **kwargs)
    +exception(msg, *args, **kwargs)
    +set_level(level)
    +{static} resolve_level(level)
    +{static} observe(observer)
    +subscribe_handler(subscriber)
    +subscribe(observer)
    +update(event, **kwargs)
  }
  class ContextFilter {
    +filter(record)
  }
  class MeasurementFilter {
    +TARGETS
    +filter(record)
  }
  class AsyncHandler {
    +queue
    +emit(record)
  }
  enum LogFormat {
    DEFAULT
    Detailed
    Custom
    JSON
  }
Audit <|-up- Wattleflow
ILogger <|.down. Audit
IObserver <|.down. Audit
H <|-down- AsyncHandler
F <|-down- ContextFilter
F <|-down- MeasurementFilter
Audit -right-> L : _logger
Audit -down-> "0..1" H : _handler
Audit .right.> LogFormat
Audit .right.> ContextFilter
@enduml
```

## 03. Context Diagram

```plantuml
@startuml
!include <C4/C4_Context>
!include requirements/styles/wattleflow.puml
caption Context Diagram
title Audit record path

LAYOUT_TOP_DOWN()
skinparam nodesep 40
skinparam ranksep 50

  System(sys, "Audit", "Turns a level call into a record in the repository")
System_Ext(em, "Wattleflow", "Base class; its descendants call the level methods")
System_Ext(lg, "logging", "Logger and handlers of the standard library")
System_Ext(ob, "Monitor", "Receives records that carry a step")
System_Ext(sp, "Record repository", "Stream or file")
Rel_L(em, sys, "Calls level with named fields")
Rel_R(sys, lg, "Passes record to handler")
Rel_D(sys, ob, "Reports step records to")
Rel_U(sys, sp, "Writes formatted record")
@enduml
```

## 04. User Diagram

| uloga | što radi | zatečeno |
|---|---|---|
| **A1** `Wattleflow` (emitent) | potomak temeljne klase; poziva `debug` · `info` · `warning` · `error` · `critical` · `fatal` · `exception` s imenovanim poljima | da |
| **sabirnik** | razdvaja logging argumente od podatkovnih, serijalizira podatkovna u tekst, predaje stdlib loggeru | da |
| **A2** handler | formatira i upisuje u spremište | da — **jedan**, zadani stream handler; dodatni se pretplaćuju kroz `subscribe_handler` |
| **A3** promatrač zapisa | prima zapis koji nosi `step` ili zatvara jedinicu rada, prije razinske brane | da — jedan po procesu; postavlja ga `workflow.py` (`Audit.observe(monitor.hook)`) za trajanje prolaza i vraća prethodnog |
| **A4** `GenericWorkflow` | za trajanje prolaza (`run`) postavlja promatrača zapisa (`Audit.observe`) i vraća prethodnog | da |
| **A5** `WorkflowFactory` | postavlja razinu iz konfiguracije (`Audit.resolve_level`, `set_level`) | da |
| **A6** `IObservable` | šalje događaj sabirniku kroz `update` | da (ugovor; pozivatelja u živim stablima nema) |

Spremište nije sudionik koji išta pokreće.

Dijagram korisnika: slijedi.

## 05. Events

| okidač | što se dogodi | akter |
|---|---|---|
| **EV01** | emitent poziva razinu (`debug` … `exception`) s imenovanim poljima | A1 |
| **EV02** | promatrani događaj stiže u `update` i postaje INFO zapis | A6 |
| **EV03** | zapis nosi `step` ili je `msg` jednak `Event.Processed`: promatrač prima zapis | A3 |
| **EV04** | handler prima zapis (`emit`), formatira ga i upisuje | A2 |
| **EV05** | prolaz workflowa postavlja promatrača zapisa | A4 |
| **EV06** | razina se postavlja ili mijenja (`set_level`, `resolve_level`) | A5 |
| **EV07** | handler se pretplaćuje (`subscribe_handler`) | A2 |

## 06. Use Case Diagrams

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Use Cases
title Audit record path

left to right direction

skinparam nodesep 10
skinparam ranksep 20

package {
  actor "Wattleflow" as A1
  actor "logging.Handler" as A2
  actor "Monitor" as A3
  actor "GenericWorkflow" as A4
  actor "WorkflowFactory" as A5
  actor "IObservable" as A6
    usecase "Emit level record" as EV01
    usecase "Record observed event" as EV02
    usecase "Receive step record" as EV03
    usecase "Format and write record" as EV04
    usecase "Install record observer" as EV05
    usecase "Change level" as EV06
    usecase "Subscribe handler" as EV07
  A1 --> EV01
  A6 --> EV02
  A3 --> EV03
  A2 --> EV04
  A4 --> EV05
  A5 --> EV06
  A2 --> EV07
}
@enduml
```

## 07. Constraints and Preconditions

Slijedi. Preduvjeti i ograničenja nisu zapisani u zatečenom dokumentu.

## 08. Sequence Diagrams

### Normalan tok

1. Emitent poziva razinu s imenovanim poljima; `stacklevel` se postavlja na 3 da zapis pokaže
   pozivatelja, ne sabirnik. Prije svega ostalog sabirnik provjerava **razinsku branu**
   (`isEnabledFor(level)`): ako razina nije omogućena, zapis se odbacuje bez serijalizacije i
   formatiranja (v0.0.1.14).
2. Sabirnik odvaja **kontrolne** argumente (`exc_info`, `stack_info`, `stacklevel`, `extra`) od
   **podatkovnih**. Kontrolni idu stdlib pozivu, podatkovni u tekst.
3. Polje `target`, ako postoji, putuje i kao atribut zapisa `wf_target` (`extra`, vrijednost kao
   tekst), da handler može birati zapise po vrsti bez raščlanjivanja poruke. Čita ga
   `blackwattle/src/wattleflow/metrics/logs.py`; `MeasurementFilter` ga mjeri prema
   `MetricTarget.measurements()`, ali ga nitko ne priključuje (odjeljak 15).
4. Podatkovna polja se serijaliziraju po vrijednosti:
   - skalar i `None` — doslovno `ključ=vrijednost`;
   - zbirka — **samo na razini INFO** skraćena na oznaku tipa i duljinu; na ostalim razinama ide
     `repr`;
   - ostalo — `repr` odrezan na 100 znakova.
5. Serijalizirana polja se **konkateniraju u poruku**. Iza ove točke pojedinačno polje više ne
   postoji — zapis je rečenica.
6. Stdlib logger predaje zapis svojim handlerima; handler formatira i upisuje.

**Rasprostiranje.** Logger je jedan **po klasi** (`getLogger(type(self).__name__)`), ne po
instanci. Zadani handler se gradi jednom po klasi, inače bi svaka instanca umnožila zapis.
Razina se primjenjuje kad god stigne eksplicitno — **zadnji eksplicitni pisac pobjeđuje** — jer
je logger dijeljen; izostavljena razina ne resetira tuđu postavku. Primjenjuje se na logger i na
handler koji klasa sama posjeduje (`_apply_level`); **pretplaćeni handler se ne dira**, jer je
razinu izabrao njegov pretplatnik (konzola na INFO, prosljeđivač na DEBUG). Logger ipak odlučuje
prvi: prosljeđivač ne vidi DEBUG ako logger nije na DEBUG. Razinu nakon gradnje mijenja
`set_level(level)`, a `Audit.resolve_level(level)` daje isto preslikavanje (ime ili `int`; nepoznato
ime je `ValueError`, drugi tip `TypeError`) pozivatelju koji konfigurira tuđi logger — tako ih
koristi `workflow.py`.

**Detekcija okvirnih objekata.** Zbirke nalik `DataFrame`u prepoznaju se pretragom `__mro__`, ne
`hasattr`om na instanci: sonda na živom objektu pokrenula bi `__getattr__` kuku — rdflib bi
emitirao upozorenje, lijeni proxy otvorio vezu samo da bi bio zapisan.

**Drugi ulaz.** Sabirnik je i promatrač: promatrani događaj ulazi u isti put kao INFO zapis s
`msg=Notify`, događajem (`name` ako postoji, inače `str`) kao imenovanim poljem `event` i ostatkom
argumenata kao poljem `kwargs`; `stacklevel` je 4. To je točka na koju bi se priključilo
prosljeđivanje vanjskim sustavima (`CLAUDE.md` §6.2).

**Promatrač zapisa.** `Audit.observe(observer)` postavlja jedan promatrač na razini procesa
(poziv `(owner, msg, step, fields)`; vraća prethodnog, a `None` ga uklanja). Poziva se iz
`_log_msg` kad zapis nosi `step` ili je `msg` jednak `Event.Processed`, i to **prije**
razinske brane, pa promatrač vidi zapis i kad razina nije omogućena. `observe(None)` ga uklanja.
Mjerenje tako ne ovisi o razini DEBUG (v0.0.1.14).

### Dijagram slijeda

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Sequence Diagram
title Audit record path

  participant "Wattleflow" as E
  participant "Audit" as B
  participant "logging.Logger" as L
  participant "logging.Handler" as H
  participant "Monitor" as O
  E -> B : debug(msg, **kwargs)
  activate B
  opt step given or msg is Processed
    B -> O : observe(self, msg, step, kwargs)\n(observe_summary below level OPERATIONS)
    activate O
    deactivate O
  end
  opt level enabled on logger
    B -> B : separate control from data arguments
    B -> B : target also travels as attribute wf_target
    B -> B : serialise fields, concatenate into message
    B -> L : log(level, msg, *args, **pass_through)\n(stacklevel=3, extra)
    activate L
    L -> H : emit(record)
    activate H
    alt write succeeds
      H -> H : format and write
    else write fails
      H -> H : handleError (business flow is not interrupted)
    end
    deactivate H
    deactivate L
  end
  deactivate B
@enduml
```

## 09. Flow Chart Diagrams

Normalan tok je glavni put; grane odlučivanja su alternativni tokovi iz tablice alternativnih tokova.


```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Flow Chart
title Audit record path

start
:Wattleflow descendant calls level with named fields (stacklevel 3);
if (step given or msg is Processed?) then (yes)
:Monitor receives (owner, msg, step, kwargs);
endif
if (level enabled on logger?) then (yes)
:Audit separates control from data arguments;
if (target field given?) then (yes)
  :target also travels as record attribute wf_target;
endif
if (data fields given?) then (yes)
  while (field left?) is (yes)
    if (field value type?) then (scalar or None)
      :key=value;
    elseif (collection at level INFO?) then (yes)
      :type tag and length;
    else (other)
      :repr truncated to 100 characters;
    endif
  endwhile (no)
  :fields are concatenated into message;
endif
:logging.Logger passes record to handlers;
if (write failed?) then (yes)
  :handleError; business flow is not interrupted;
else (no)
  :handler formats and writes;
endif
else (no)
:record is discarded before formatting;
endif
stop
@enduml
```

## 10. State Machine

Nije primjenjivo: `audit.py` ne sadrži automat ni stanja (pretraga `state`, `fsm`, `StateMachine`).

## 11. Non-Functional Requirements

Registar i obveze skupine: [`NFRQ-000-INDEX`](../03-NFRQ/NFRQ-000-INDEX.md).

| NFR | predmet |
|---|---|
| [`NFRQ-OBS-01`](../03-NFRQ/NFRQ-OBS-01-audit-levels.md) | razine audit zapisa: značenje, publika, mjesto |
| [`NFRQ-OBS-02`](../03-NFRQ/NFRQ-OBS-02-audit-fields.md) | polja zapisa: `msg`, `step`, `scope`, `component`, `target` |
| [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | vlasništvo, redoslijed i volumen audit zapisa |
| [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | povjerljivost audit zapisa |
| [`NFRQ-ORG-07`](../03-NFRQ/NFRQ-ORG-07-preset-allowed-declaration.md) | ulazna površina komponente (`ALLOWED`) |

## 12. Results

Zapis je stigao do handlera kao jedna rečenica (korak 5 u odjeljku 08), formatirana i upisana u spremište; pojedinačna polja iza konkatenacije ne postoje. Zapis čija razina nije omogućena odbačen je bez serijalizacije i formatiranja. Promatrač zapisa vidi zapis sa `step` i prije razinske brane.

## 13. Acceptance Criteria

| | kriterij | stanje |
|---|---|---|
| 1 | Ulazna površina ne postaje zapisna: `**kwargs` se ne prosljeđuje u logging poziv ([`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) k.1) | mjereno, uz granicu populacije (odjeljak 14) |
| 2 | Zapis pokazuje pozivatelja, ne sabirnik | drži `stacklevel` |
| 3 | Jedna instanca ne umnožava zapise svoje klase | drži gradnja handlera jednom po klasi |
| 4 | Serijalizacija ne dira instancu podatkovnog objekta | drži MRO sonda |
| 5 | Neuspjeh upisa ne prekida poslovni tok | drži stdlib `handleError`, **ne ovaj kod** |

## 14. Verification

Kriterij 1 — `wem_lint --select OBS-02`, nalaz `audit-kwargs-splat`; nad `workflow/src/wattleflow` daje
0 nalaza. Trojka (D-10): alat `wem_lint 1.18.0` · kriterij `tools/dictionary.json` 0.10.0 ·
platforma `python 3.12.7 (Linux)` · dokumentacija `v0.0.5` · stablo `src/wattleflow` · pravilo `OBS-02`.

> **Populacija pravila uža je od iskaza kriterija (D-11).** Pravilo prepoznaje samo pozive s
> primateljem `self`; zapis preko modulskog loggera ili tuđe instance nije u populaciji. Dva
> takva mjesta postoje — `workflow/src/wattleflow/concrete/workflow.py:266` i
> `blackwattle/src/wattleflow/decorators/oscal/policy.py:60`. Zeleni vektor ne dokazuje nulu iz
> iskaza kriterija. Proširenje populacije mijenja kriterij i traži dokumentiranu izmjenu.

Kriteriji 2–5 nemaju test; kriterij 5 posebno, jer ga danas drži stdlib
svojstvo koje ništa ne bi primijetilo da otpadne. Deklarirana rupa.

Run nije C-snimka: `OBS-01/02/03` još mjere po ne po.

## 15. Open issues

- Kategorija `AUD` traži dokumentiranu izmjenu (zaglavlje).
- Klasifikacijska ljestvica iz odjeljka 15 (Aspiracija) nije usvojena — kandidat za [`NFRQ-SEC-06`](../03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) ili zasebnu dokumentiranu odluku.
- Čime handler **dokazuje** sposobnost (deklaracija vs provjera) — Q5c u analizi *CIA, audit i
  OSCAL* (2026-08-12); izvor nije u repozitoriju (D-11).
- Inertni observable ugovor i populacija pravila k.1 — vidi odjeljak 14 i odjeljak 15 (Nalazi).

### Nalazi (D-11)

| nalaz | posljedica |
|---|---|
| Sučelje loggera nasljeđuje *observable* ugovor, ali je pretplata promatrača `NotImplementedError`| deklarirani ugovor je inertan; emitirajuća strana radi, pretplatna ne. Mijenja core sučelje → traži dokumentiranu izmjenu |
| Točka pretplate handlera (`subscribe_handler`) nema **nijedno** pozivno mjesto u živim stablima (samo u `__unused`); isto vrijedi za `AsyncHandler` | put s više odredišta je napisan, ali neizvršen |
| `MeasurementFilter` nema nijedno pozivno mjesto; njegov atribut `wf_target` postavlja sabirnik, a čita ga samo `blackwattle/.../metrics/logs.py` | filtar je mrtav kod, izvezen u `__all__`; mjerni zapis ne filtrira nitko |
| Kontekstni filtar čita polja koja **nitko ne postavlja**; format koji ih koristi nema potrošača</p> | filtar je no-op, a taj format bi pao na nedostajućem polju |
| Članovi tablice formata miješaju `UPPER_SNAKE` i `PascalCase` | odstupanje od §2.3 |
| Skraćivanje zbirki vrijedi samo na INFO razini | ponašanje volumena; pripada [`NFRQ-OBS-03`](../03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md), ondje nije zapisano |

### Aspiracija: put s više odredišta (D-05)

Framework predviđa više handlera prema spremištima različitog povjerenja (datoteka, Kafka, NiFi,
`Repository`), pri čemu bi zapis stizao **strukturiran**, svako polje klasificirano
(javno · identitet · mjera · klasificirano), a handler propuštao samo ono što mu deklarirana
sposobnost redakcije pokriva; neuspjeh upisa bio bi događaj za monitoring, a zapis nastao unutar
handlera ne bi se vraćao u put.

**Ništa od toga nije provedeno, i korak 5 iz odjeljka 08 je razlog:** konkatenacija u rečenicu događa se
prije handlera, pa klasifikacija i redakcija po polju nisu izvedive. Ovo
nisu tri odvojena posla nego jedan — dok zapis do handlera ne stigne strukturiran, ostali koraci
nemaju predmet.

## 16. References

- [`audit.py`](../../../workflow/src/wattleflow/helpers/audit.py)

## 17. Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-04 | Dijagrami izrađeni iznova prema `wattleflow-uml` (bez `frame`, `package`, `partition` i `System_Boundary`, `Caption`/`title` po pravilu, bez stereotipa i legende, sučelja na vrhu; crvene strelice stanja i akcija neuspjeha podebljane); renderirani s PlantUML 1.2026.8 i pregledani. |
| v0.0.5 | 2026-10-03 | Odjeljci preuređeni u standardnu strukturu FRQ dokumenta 01–17 (`wattleflow-docs` §3e); okidači izdvojeni iz slučajeva uporabe u 05, nalazi (D-11) i aspiracija (D-05) premješteni u 15, NFR iz zaglavlja u 11; u 04 popravljena tablica aktera. Odjeljci 07 i 10 označeni kao „slijedi" odnosno „nije primjenjivo". |
| v0.0.5 | 2026-10-03 | Vrijednosti u dijagramima provjerene prema kodu: uloga promatrača zapisa zamijenjena klasom Monitor (hook), poziv promatrača `observe(self, msg, step, kwargs)`, `_handler` (0..1) umjesto agregacije handlera, aktor `logging.Handler`. |
| v0.0.5 | 2026-10-03 | Uloga Emitter zamijenjena klasom `Wattleflow` u svim dijagramima (naziv iz koda). |
| v0.0.5 | 2026-10-03 | Use case dijagram prema obrascu: akteri A1–A6 i okidači EV01–EV07 zapisani u §2 i §3, slučajevi su ciljevi aktera (ne koraci sabirnika), stereotipi `<<flow>>`, `<<event>>`, `<<configuration>>`, veze `-right->`. |
| v0.0.5 | 2026-10-03 | Dijagrami usklađeni s `audit.py` i vještinom `wattleflow-uml`: dodani record observer i razinska brana (context, use case), `Wattleflow` i `logging.Logger` (class), `wf_target` i petlja po poljima (activity, sequence), legenda i redoslijed uključivanja u C4, nazivi iz koda. |
| v0.0.1 | 2026-10-02 | Konsolidacija dokumentacije i koda, izrada diagrama etc. |
| v0.0.2 | 2026-10-03 | Dodane aktivacije u dijagram slijeda. |
