# Wattleflow funkcionalni zahtjevi (FR registar)

## Uvod

**Verzija:** Draft v0.1  
**Izvorni jezik:** hrvatski  
**Pratioci:** [METHODOLOGY.md](METHODOLOGY.md), [PHILOSOPHY.md](PHILOSOPHY.md), [DOCTRINE.md](DOCTRINE.md)

Ovaj registar drži provedive, mjerljive funkcionalne zahtjeve Wattleflow
workflow ekosustava. Njegova uloga je prevesti arhitektonska i filozofska načela u
konkretne, provjerljive obveze.

---

## Svrha

Ovaj registar drži provedive, **mjerljive** funkcionalne zahtjeve Wattleflow Workflow ekosustava. 
Svaki FR:

* nosi jedinstveni identifikator `FR-<KATEGORIJA>-NN` (npr. `ORG` = organizacija /
  struktura);
* usidren je na ontologiju i inženjerstvo zahtjeva iz **ISO/IEC/IEEE 29148**
  (a za podatke ISO/IEC 25012);
* navodi strojno provjerljive **kriterije prihvaćanja** i metodu **verifikacije**;
* sljeduje se prema gore do načela koje ga opravdava (`METHODOLOGY.md` §9
  *Arhitektonska sljedivost*).

> **Status verifikacije.** Test-framework i CI pipeline još nisu odabrani (vidi
> `POLICY.md` otvorena pitanja). Do tada „strojno provjerljivo" znači **lint-skripta**
> nad import-grafom / gramatikom imena, pokretljiva na zahtjev; kad CI postoji, ista
> skripta postaje quality gate. Kriteriji su pisani za tu skriptu.
>
> **Izvedba:** `tools/wem_lint.py` (čita `tools/naming_registry.yaml` — registar
> kontroliranog vokabulara pod ADR upravljanjem). Pokretanje:
> `python tools/wem_lint.py` (izlazni kôd ≠ 0 na ERROR-prekršaj). Trenutno stanje
> nad zatečenim kodom: 13 ERROR + 16 WARN — to je worklist za migraciju imena (korak 3).

---

## Zajedničke definicije

Ove pojmove koristi svaki `FR-ORG-*` zahtjev.

* **Domena** — *top-level funkcionalni paket*: `pipelines`, `drivers`, `processors`,
  `strategies`, `connections`, `documents`, `blackboards`.
* **Pomoćna (support/helper) klasa** — klasa koja nije dio javnog ugovora domene
  (nije `Pipeline*`, `Driver*`, `Processor*`, strategija itd.); postoji da posluži
  drugim klasama.
* **Domain-internal shared modul** — modul koji dijele dva ili više *pod-paketa
  jedne domene* (npr. OCR helper koji koriste `pipelines/pdf` i `pipelines/png`).
  Živi **unutar te domene** (npr. `pipelines/convertors`), a ne u globalnom
  `helpers/`.
* **Dijeljeni helper** — klasa koju koriste **dvije ili više različitih domena**;
  samo se takve promiču u globalni `helpers/`, organizirane po **sposobnosti**
  (`io`, `geometry`, `text`, `validation`), nikad po potrošačkom sloju.
* **Kanonski subjekt** — puni, neskraćeni naziv subjekta korišten i kao ime
  domenskog paketa i kao vodeći facet imena klase (`dataframe`, ne `dframe`;
  `text`, ne `txt`). Registar mapira svaku zatečenu kraticu na kanonski subjekt
  tijekom migracije.

> **Zero-trust napomena (CLAUDE.md §7.4).** Dijeljeni helper koji ovisi o
> third-party paketu (npr. OCR → `pytesseract`) **ne smije** u čisti core
> `helpers/`; pripada `wattleflow-processors` paketu i lazy se učitava. Promocija u
> „dijeljeni" poštuje zero-trust granicu.

---

## FR-ORG-01 — ...