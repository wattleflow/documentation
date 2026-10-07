<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# HLRQ-MMN — Moment: vrijeme kao broj nanosekundi

> **Oznaka je provizorna.** Kategorija `MMN` nije u registru; uvođenje traži dokumentiranu izmjenu (D-12).

| | |
|---|---|
| **Verzija** | v0.0.5 |
| **Odluka** | Odluke autora 2026-10-06: dva modela vremena (s trenutkom i bez zone) koja se ne miješaju; domena godine 1–9999 provjerena pri izgradnji (opcija A); zidno vrijeme nema oblik RFC 5322; prijelaz u stupce u nanosekundama samo kroz jedan most; `Moment` potpuno zamjenjuje `helpers/dtime.py`; zona workflowa kao globalna postavka (`WATTLEFLOW_TIME_ZONE`) |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `wattleflow-workflow` — clean core tier (samo standardna biblioteka) |
| **Predmet** | `workflow/src/wattleflow/helpers/moment/`: `Moment`, `MomentHelper`, `MomentAwareHelper`, `MomentNaiveHelper` |
| **Djeca** | [`FRQ-MMN`](../02-FRQ/FRQ-MMN-moment.md) · analiza selidbe [`FRQ-MMN-moment-ANL`](../02-FRQ/FRQ-MMN-moment-ANL.md) |
| **Sljedivost** | [`NFRQ-DEF-04`](../03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md) · [`NFRQ-DEF-05`](../03-NFRQ/NFRQ-DEF-05-moment-naive-helper.md) · [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) · [`NFRQ-ORG-09`](../03-NFRQ/NFRQ-ORG-09-external-standard.md) |
| **Standardi** | POSIX epoha (1970-01-01T00:00:00 UTC); ISO 8601 i RFC 3339 (tekst); RFC 5322 §3.3 (pošta); RFC 9110 (HTTP datum); IANA baza zona; JD i MJD |
| **Dijagrami** | inline (§2) — pogled, ne izvor istine (D-13) |

## 1. Narativ

Podaci dolaze s vremenom u mnogo oblika: tekst ISO 8601 s pomakom ili bez njega, zaglavlje pošte,
broj sekundi, metapodaci dokumenta, stupac tablice. `Moment` je jedan oblik za sve njih: cijeli broj
nanosekundi od epohe 1970-01-01 i oznaka vrste. Vrsta je **s trenutkom** (aware: UTC trenutak, a
IANA zona služi samo za prikaz) ili **zidno vrijeme** (naive: vrijeme bez zone). Pomoćni razredi
svake vrste grade `Moment` iz ulaza i pišu ga u izlazne oblike. Zonu nikad ne izmišljaju.

**Zašto.** Broj od epohe je jednoznačan, usporedba i aritmetika su cjelobrojne, ne ovise o
lokalnim postavkama, a odgovaraju zapisu vremena u stupcima (`numpy.datetime64`, pandas, Arrow).
Model jednog trenutka po vrijednosti razdvaja dva podatka koja se lako pomiješaju: koji je
trenutak, i u kojoj ga zoni prikazati. Mjera uspjeha je da nijedna vrijednost ne promijeni
značenje na putu od ulaza do izlaza.

## 2. Mjesto u dekompoziciji

| uloga | razred | što radi |
|---|---|---|
| vrijednost | `Moment` | nanosekunde, vrsta, zona prikaza; nepromjenjiv; usporedba i aritmetika unutar vrste |
| zajedničko | `MomentHelper` | zone, izbor pomoćnog razreda po vrsti, serijalizacija koja nosi vrstu |
| trenutak | `MomentAwareHelper` | ulaz iz `datetime` sa zonom, ISO s pomakom, Unix vremena; izlaz u zoni, ISO, RFC 5322, HTTP, JD; jednosmjerni prijelaz u zidno vrijeme (`strip`) |
| zidno vrijeme | `MomentNaiveHelper` | ulaz iz `datetime` bez zone i ISO bez pomaka; izlaz u `datetime`, ISO, tekst |

<div align="center">

```plantuml
@startuml
!include requirements/styles/wattleflow.puml
caption Decomposition
title HLRQ-MMN: Moment and its helpers

top to bottom direction

package "Sources" {
  component "datetime, ISO 8601 text,\nUnix time, RFC 5322 header" as IN
}
package "helpers/moment (FRQ-MMN)" {
  component "MomentAwareHelper\n(a moment: UTC + display zone)" as AW
  component "MomentNaiveHelper\n(wall time, no zone)" as NA
  component "Moment\n(ns since 1970, kind, zone;\nyears 1-9999)" as M
  component "MomentHelper\n(zones, kind, serialisation,\nbridge to ns columns)" as H
}
package "Consumers" {
  component "datetime, ISO, RFC 5322, HTTP,\nJD, bytes and dict" as OUT
  component "int64 ns column\n(numpy, pandas, Arrow)" as COL
}

IN --> AW : moment, from_*
IN --> NA : moment, from_*
AW --> M : builds
NA --> M : builds
AW --> NA : strip (one way)
AW --> OUT : to_*
NA --> OUT : to_*
H --> COL : to_int64_ns
AW --|> H
NA --|> H
@enduml
```

</div>

## 3. Opseg

**U opsegu:** paket `helpers/moment/`: vrijednost, pomoćni razredi, domena vrijednosti i prijelaz
u stupce u nanosekundama.

**Izvan opsega:** vremenske skale osim UTC-a (TAI, GPS, TT, TDB), kalendari osim proleptičkog
gregorijanskog (CF `noleap`, `360_day` i drugi), razlučivost finija od 1 ns i epohe izvan godina
1–9999. Ulaz takve vrste odbija se s imenovanom greškom; proširenje je zaseban tip (§7 t.1).

## 4. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-MMN-01** | `Moment` je cijeli broj nanosekundi od 1970-01-01. S trenutkom znači UTC trenutak; zidno vrijeme znači vrijeme bez zone, mjereno od iste ponoći. |
| **BR-MMN-02** | Dvije vrste se ne miješaju: usporedba i razlika `Moment`a različitih vrsta su greška, a pomoćni razred jedne vrste odbija ulaz druge. |
| **BR-MMN-03** | Zona se ne izmišlja. Prijelaz iz zidnog vremena u trenutak ne postoji; prijelaz iz trenutka u zidno vrijeme (`strip`) navodi zonu ili koristi spremljenu. |
| **BR-MMN-04** | Zona trenutka služi samo prikazu: ne ulazi u jednakost, poredak ni sažetak (`hash`). |
| **BR-MMN-05** | Domena je proleptički gregorijanski kalendar, godine 1–9999 uključivo (`MIN_NS`…`MAX_NS`), za obje vrste. Vrijednost izvan domene odbija se pri nastanku, i u aritmetici, tvornicama, deserijalizaciji i `strip`; `Moment` izvan domene ne postoji. |
| **BR-MMN-06** | Prikaz u zoni koji pomakne vrijednost izvan godina 1–9999 daje imenovanu grešku raspona, ne golu grešku knjižnice. |
| **BR-MMN-07** | Stupci u nanosekundama dobivaju vrijednost samo kroz jedan most koji odbija vrijednost izvan raspona int64 i rezerviranu vrijednost `NaT` (-2⁶³): numpy izvan tog raspona na dijelu putova tiho daje krivi datum. Za punu domenu stupac je u mikrosekundama. |
| **BR-MMN-08** | Nanosekunde se čuvaju u `Moment`u i u serijalizaciji. Izlazi kroz `datetime` i ISO tekst imaju razlučivost mikrosekunde i zaokružuju naniže. |
| **BR-MMN-09** | Oblik RFC 5322 postoji samo za trenutak: zidno vrijeme nema zonu, a `-0000` po RFC 5322 §3.3 znači UTC. |
| **BR-MMN-10** | Serijalizacija nosi vrstu i zonu; čitanje provjerava vrstu. |
| **BR-MMN-11** | Paket koristi samo standardnu biblioteku ([`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md)). |
| **BR-MMN-12** | Trenutak koji nastane u sustavu (`now`) nosi **zonu workflowa**. Zona workflowa je konfigurirana (konfiguracija workflowa, inače globalna postavka `WATTLEFLOW_TIME_ZONE`), inače zona sustava; razrješava se jednom, pri pokretanju workflowa ili pri prvoj upotrebi. |
| **BR-MMN-13** | Zidno vrijeme koje je napisao operater (konfiguracija, ručni unos) čita se u zoni workflowa. |
| **BR-MMN-14** | Zidno vrijeme iz izvora podataka ostaje zidno vrijeme dok izvor ne deklarira zonu (zona izvora u njegovoj konfiguraciji). Prijelaz u trenutak uvijek ima izričitu zonu i nikad je ne izmišlja (`BR-MMN-03`). |

## 5. Nefunkcionalni zahtjevi

| `NFR` | posljedica za ovu sposobnost |
|---|---|
| [`NFRQ-DEF-04`](../03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md), [`NFRQ-DEF-05`](../03-NFRQ/NFRQ-DEF-05-moment-naive-helper.md) | dva modela vremena; `BR-MMN-02`, `BR-MMN-03` |
| [`NFRQ-DEF-03`](../03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) | rubne vrijednosti `MIN_NS` i `MAX_NS` uključive; ±1 odbijeno (`BR-MMN-05`) |
| [`NFRQ-ORG-09`](../03-NFRQ/NFRQ-ORG-09-external-standard.md) | tekstni oblici omataju standardnu biblioteku (`datetime`, `email.utils`, `zoneinfo`) |
| [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | `BR-MMN-11` |

**`OSCAL`:** paket ne nosi OSCAL dekoratere.

## 6. Verifikacija

`workflow/tests/test_moment.py` provjerava `BR-MMN-01`…`10` ([`FRQ-MMN`](../02-FRQ/FRQ-MMN-moment.md)
§14); `BR-MMN-11` provjerava pregled uvoza. Trošak se mjeri skriptom `workflow/tests/bench_moment.py`.

## 7. Otvoreno

1. **Proširenje izvan opsega (§3).** Okidač je konkretan izvor podataka. Opcije po okidaču:
   - skala osim UTC-a: atribut `scale` sa zadanom vrijednošću UTC, a pretvorbu između skala radi
     vanjska knjižnica (cijena: ovisnost izvan clean corea);
   - kalendar osim gregorijanskog (klimatski modeli, CF konvencije): zaseban tip (cijena: drugi
     tip vremena u sustavu);
   - razlučivost finija od 1 ns: zapis u dva dijela, sekunde i razlomak;
   - epohe izvan godina 1–9999 (astronomija, paleoklima): JD u dva dijela.
2. **JD i MJD kao jedan `float64`** imaju razlučivost ~40 µs i ~0,6 µs. Opcija: izlaz JD u dva
   dijela (cijeli dio i razlomak) za znanstvene knjižnice.
3. **Sučelje u `core`.** `Moment` i pomoćni razredi ne ispunjavaju nijedno `core` sučelje; razred
   pomoćnih funkcija (helper) traži pattern mapu (`wattleflow-patterns`).
4. Kategorija `MMN` (zaglavlje).

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.5 | 2026-10-07 | Selidba s `helpers/dtime.py` završena (analiza `FRQ-MMN-moment-ANL`, skupine 1–6); `dtime` uklonjen; `NFRQ-DEF-04` i `NFRQ-DEF-05` preformulirani nad `Moment`om; otvorena stavka zatvorena. |
| v0.0.5 | 2026-10-06 | Kategorija `MOM` preimenovana u `MMN` (suglasnici riječi MOMENT, odluka autora); pravila `BR-MMN-12…14` (zona workflowa, zidno vrijeme operatera i izvora); odluka o potpunoj zamjeni `dtime`. |
| v0.0.5 | 2026-10-06 | `BR-MMN-05…07` i `09` provedeni u kodu i provjereni testovima. |
| v0.0.5 | 2026-10-06 | Prvi zapis: `Moment` i pomoćni razredi; pravila `BR-MMN-01…11` prema odlukama autora (opcija A, uklanjanje RFC 5322 za zidno vrijeme, most prema stupcima u nanosekundama); prijedlog. |
