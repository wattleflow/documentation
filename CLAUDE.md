# CLAUDE.md

Obvezujuće upute i standardi za rad na projektima `wattleflow` (core), `workflow`,
`processors`, `cad`. Vrijedi za sve sesije s Claude Code agentom.

**Položaj u kaskadi (D-02).** Ovo je **policy sloj**: filozofija → *policy* → princip →
metoda. Nadređeni dokumenti su [PHILOSOPHY.md](PHILOSOPHY.md) (kišobran) i
[DOCTRINE.md](DOCTRINE.md) (registar normi `D-01…D-17`); operacionalizacija je
[METHODOLOGY.md](METHODOLOGY.md); provedivi zahtjevi su u [`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md),
[`02-FRQ/`](02-FRQ/0-FRQ-EN.md) i [`01-HLRQ/`](01-HLRQ/0-HLRQ-EN.md). `POLICY.md` je isti dokument (symlink) — doktrinarni tekstovi ga zovu
`POLICY.md`, repozitoriji `CLAUDE.md`. **Postoji jedan primjerak, ovaj**: kodni repozitoriji
ga symlinkaju (`processors/CLAUDE.md → ../documentation/CLAUDE.md`, od 2026-08-23). Zatečena
kopija u `wattleflow-processors` povučena je i arhivirana kao
[`processors/CLAUDE-superseded-2026-08-23.md`](processors/CLAUDE-superseded-2026-08-23.md).

**Pravila upravljanja ovim dokumentom:**

- Izmjena policyja ide **kroz zapis odluke (DR)**, ne prešutnim uređivanjem (D-03).
- Niži sloj ne nadjačava viši: gdje se ovaj dokument razilazi s `DOCTRINE.md` ili
  s NFR registrom, prednost ima viši sloj/registar, a razilaženje je **nalaz** koji se
  prijavljuje, ne rješava u tekstu (D-02, D-12).
- Tvrdnja bez svjedočanstva vodi se kao **aspiracija** i tako se označava (D-05).

---

## 1. Svrha projekta

**Core** (`wattleflow`) je skup apstraktnih sučelja (ugovora) izvedenih iz dizajn patterna.
Koriste ga svi wattleflow projekti; sam ne nosi implementaciju.

**Workflow** (`wattleflow-workflow`) je framework za podatkovno inženjerstvo: nad `core/`
sučeljima gradi `concrete/` — generičke implementacije koje su radna osnova frameworka — i
iz njih cjevovode (workflow → pipeline → processor → strategy) za preuzimanje, transformaciju
i skladištenje podataka, u skladu s *Privacy Act 1988* (Cth), APP 3 i APP 11 (`DR-WFL-008`).

**Processors** (`wattleflow-processors`) nosi specijalizacije za heterogene izvore i spremišta
te sloj usklađenosti. Oslonjen je na third-party biblioteke i zato **nije** zero-trust paket
(§7.4).

**Slojevi** (imena su `wattleflow.*` pod-paketi, ne nužno direktoriji jednog stabla — svaki
sloj pripada distribuciji koju određuje §7.1):
- `core/` — apstraktna sučelja i dizajn patterni (autoritativno). **Živi u zasebnoj
  distribuciji i repozitoriju** (`wattleflow`, GitHub `wattleflow/core`); `workflow` i
  `processors` stabla ga **nemaju** pod `src/`, nego ga povlače kao ovisnost (`DR-WFL-006`).
  Putanja `src/wattleflow/core/` u ovom dokumentu znači stablo core repozitorija.
- `concrete/` — generičke implementacije sučelja iz `core/` (radna osnova frameworka)
- `connections/`, `drivers/`, `processors/`, `pipelines/`, `strategies/`, `documents/` —
  specijalizacije nad `concrete/` (ne pripadaju clean core distribuciji, §7)

**Domenska ontologija** (Workflow, Processor, Pipeline, Driver, Repository, Blackboard,
Strategy, Connection, Document, Memento) je autoritativna: to su rezervirani primitivi i
ne izmišljaju se novi bez DR-a (METHODOLOGY §4, NFR-ORG-04).

**Publika dokumentacije:** dokumentacija je **sustav s više publika** (D-16) — arhitekt,
implementator, tester, sigurnosni analitičar, integrator, poslovni korisnik, revizor.
Jedan format za sve publike nije legalan cilj; publika bez artefakta deklarira se kao rupa.

---

## 2. Tehnički standardi

### 2.1 Python

- **Minimalna verzija:** Python **3.11** (`requires-python = ">=3.11"` u `pyproject.toml`;
  usklađeno s classifiers i `ruff target-version = "py311"`). Politika verzija: `DR-COR-013`.
- **Ciljane verzije:** 3.11, 3.12, 3.13.

### 2.2 PEP-ovi koji se primjenjuju

- **Obvezno:** PEP 8 (stil), PEP 484 za javna sučelja, PEP 585 generici iz `builtins`,
  PEP 604 union sintaksa, PEP 621 metapodaci u `pyproject.toml`, PEP 420/660 za dev-stabla
  (§7, NFR-SEC-03). PEP 257 vrijedi **ograničeno** — vidi §2.4.
- **Gdje povećava jasnoću:** PEP 634 `match/case`, PEP 612 `ParamSpec`, PEP 526 anotacije.

### 2.3 Stil

- Formatter/linter: **ruff**.
- Maksimalna duljina linije: **100 znakova** (`pyproject.toml [tool.ruff]`).
- Imena: `snake_case` za funkcije/varijable, `PascalCase` za klase, `UPPER_SNAKE` za
  konstante.
- **`# region` / `# endregion`** — projektna konvencija za vizualno grupiranje (Imports,
  Constants, Interface, Implementation…). Zadržava se u svim slojevima.

### 2.4 Komentari i docstrings

- Komentari uvijek na **UK engleskom**.
- **Minimalno** — komentar je opravdan samo kad WHY nije očit iz koda.
- Docstring javnog sučelja: kratko i jasno 1–2 rečenice, bez verboznih reST blokova.
- Tip-hintovi zamjenjuju većinu dokumentacijskih potreba.

### 2.5 Sučelja iz `core/`

- Sučelja u `src/wattleflow/core/` su **autoritativna**; mijenjaju se samo kroz DR
  (serija `DR-COR`).
- Konkretne implementacije frameworka idu kroz klase u `src/wattleflow/concrete/`.
- Nova klasa **mora** naslijediti odgovarajuće apstraktno sučelje iz `core/`.
- Ako odgovarajući pattern ne postoji, pitati treba li ga uvesti u `core/` — agent ga ne
  dodaje sam (§5 t.3).

### 2.6 Organizacija modula (specijalizacije)

| Situacija | Layout |
|---|---|
| Mali broj povezanih klasa (~1–3) | **Jedna `.py` datoteka**. Primjer: `helpers/dtime.py`. |
| Veći skup ili klase različitih odgovornosti | **Poddirektorij**, jedna klasa po datoteci. Primjer: `decorators/oscal/policy.py`. |
| Svaka konfiguracija | Poddirektorij nosi `__init__.py` s čitljivim `__all__`. |

**Razlog:** navigacija, jasniji review diff, manje merge konflikata, eksplicitna API
površina (NFR-SEC-02 t.2). Lokalne pomoćne funkcije ostaju uz klasu koju opslužuju.

**Smještaj helpera propisuje NFR-ORG-01** (lokalnost ovisnosti): helper jedne domene ostaje
u toj domeni; helper dvaju pod-paketa iste domene ide u domain-internal shared modul; samo
helper **dviju ili više domena** ide u dijeljeni `helpers/`, organiziran **po sposobnosti**
(`io`, `text`, `routing`, `validation`), nikad po potrošačkom sloju. Iz `helpers/` ne smije
postojati import-brid prema domenskom paketu (acikličnost). Cross-cutting sposobnost je
**pozivljivi helper**, ne specijalizacija domenskog primitiva (NFR-ORG-04, `DR-WFL-001`).

### 2.7 Javni API: `__all__` i `import *`

Standard vrijedi za sve konacne `wattleflow-*` pakete (PEP 8 „Public and internal interfaces");
puna odluka, oblici agregata s primjerima i worklist migracije: [`DR-WFL-007`](workflow/dr/DR-WFL-007-lazy-aggregate-public-api.md).

1. **Svaki modul i `__init__.py` deklarira `__all__`** — eksplicitna javna API površina;
   neizloženo ostaje privatno (NFR-SEC-02 t.2).
2. **`from wattleflow.<paket> import *` je podržan ulaz** — `__all__` mora biti potpun i
   točan; imenovani uvoz je ulaz koji radi i na djelomičnoj instalaciji.
3. **Paketni `__init__.py` agregira javna imena pod-modula**, nikad ih ne prepisuje ručno.
   Oblik agregata — eager ili s odgođenim razrješavanjem imena (PEP 562) — bira **test
   maskiranja**, ne procjena: *maskiraj svaku opcionalnu biblioteku distribucije → uvezi
   svaki `__init__.py`*; paket koji padne mora na odgođeni oblik. Kriterij je strojno izvediv,
   ali **nije automatiziran** — deklarirana slijepa pjega (D-11), ne pokrivenost.
4. **Cross-distribucijski uvoz je uvijek eksplicitan submodul**, nikad agregat: agregat
   pripada točno jednoj distribuciji (NFR-SEC-03 kriterij 2) pa ne smije nabrajati tuđe
   module. Odgođeni oblik ovo ograničenje ne ukida.

### 2.8 Imenovanje — provodi se lintom

Imenovanje je **semiotička politika**, ne stilska preferencija (P-08): ime je znak čiji odnos
prema ulozi registar fiksira, a lint provodi.

- **Gramatiku imena propisuju registri:** klase — `NFR-ORG-02`, tipske varijable —
  `NFR-ORG-03`. Ovdje se ne prepisuje; kriterij koji lint čita je `tools/dictionary.json`.
- **Vokabular je kontroliran:** novi facet, domena, obitelj baza, akronim ili uloga ulaze
  **kroz DR**, ne dopisivanjem u registar radi „zelenog" linta. Ime klase se u prozi ne
  navodi (§3.3) — nosi ga `bases` u rječniku, što je i mehanizam protiv ontology creepa.
- **Akronimi: odluka je otvorena (`DR-WFL-004`).** Do odluke lint upozorava (WARNING,
  deklarirani waiver, `acronym_identifier_casing.status: undecided`) i **ne ruši build**.
  Ne provoditi nijednu stranu (`PDF` vs `Pdf`) masovnim preimenovanjem — provođenje
  neodlučenog krši kaskadu (D-02).

### 2.9 `@staticmethod` vs `@classmethod` i code-wrapping (odluka 2026-07-09, NFR-ORG-05)

Srodne **bezstanjne** pomoćne funkcije koje dijeli **više klasa/strategija** omataju se u
**kvalificiranu klasu** (ne slobodne module-funkcije, ne generički `Helper`) koja drži
pripadne **konstante kao class-atribute** i **metode**:

1. **`@classmethod` kad metoda referira vlastitu klasu** — kroz `cls` (`cls.CONST`,
   `cls.sibling()`). Tvrdo kodirano ime klase unutar `@staticmethod` je **prekršaj** (lomi
   preimenovanje i override).
2. **`@staticmethod` samo za čiste metode** (ne diraju nijedan član svoje klase). Izbor
   dekoratora **izražava namjeru**.
3. **Konstante bez redundantnog prefiksa** — `Layout.SUBDIR`, ne `Layout.LAYOUT_SUBDIR`.

**Iznimka (kriterij 1).** Metoda čiji potpis već veže parametar imenom `cls` (npr.
`Attribute.mandatory/optional/exists(caller, name, cls: type, …)`) ostaje `@staticmethod` dok
se parametar ne preimenuje (`cls`→`kind`/`expected`) — zasebna API odluka; lint to priznaje
kao odgođeno.

Migracija zatečenih mjesta vodi se u `workflow/TODO.md`; popis se dobiva AST pretragom, ne
prepisuje u ovaj tekst (§9, D-13).

---

## 3. Standardi dokumentacije

### 3.1 Stablo i izvori istine

Dokumentacija živi u repozitoriju **`documentation/`** (nema `docs/` stabla u ovom projektu).
Doktrinarni sloj je u korijenu, zahtjevi i zapisi u numeriranim kategorijama:

```
documentation/
├── PHILOSOPHY.md              kišobran (v0.4.2, EN): kaskada, hipoteze H1–H3
├── METHODOLOGY.md             Svezak I — Temelji (v0.3.2, HR)
├── DOCTRINE.md                registar normi D-01…D-17 (HR)
├── POSTULATE.md               registar postulata P-01…P-21 (HR)
├── LITERATURE.md              konsolidirane reference, ključevi 1–64 (append-only)
├── DICTIONARY.md   → workflow/hr/RIJECNIK.md   generirani prikaz rječnika (symlink)
├── dictionary.yaml            rječnik diskursa (izvor istine, HR)
├── CLAUDE.md / POLICY.md      ovaj dokument (policy sloj; POLICY.md je symlink)
│
├── 01-HLRQ/                   zahtjevi visoke razine — indeks `0-HLRQ-EN.md` + zapis po sposobnosti
├── 02-FRQ/                    FR registar — indeks `0-FRQ-EN.md` + zapis po zahtjevu
├── 03-NFRQ/                   NFR registar — indeks `0-NFRQ-EN.md` + zapis po zahtjevu
├── 04-DR/                     DR-PRC serija (processors)
├── 05-METHOD/                 metode (DQI)
├── 06-ANALYSIS/               analize, datirane
├── 07-CHANGES/                zapisi usklađenja s izdanjima koda, datirani
│
├── processors/                po-projektni tekstovi + `dictionary-processors.yaml`
└── workflow/
    ├── dr/            DR-WFL serija + `DR-WFL-INDEX.md`
    ├── core/dr/       DR-COR serija + `DR-INDEX.md`
    ├── conformance/   C-snimke (vektor + trojka reproducibilnosti)
    ├── Analiza.md     istraživački rad — podloga `[M]` i SEC zahtjeva
    ├── analysis/      analize i index znanja
    ├── changes/       zapisi usklađenja s core izdanjima
    ├── concrete/      nacrti uz concrete sloj
    ├── TODO.md / DONE.md  worklist (stanje rada, ne norma)
    └── hr/            izvorni HR tekstovi i stubovi preseljenih registara
```

Uz njih, u **kodnom** repozitoriju: `tools/dictionary.json` (kontrolirani vokabular **koda**,
UK English — kriterij koji čita lint), `tools/messages.json` (prezentacija nalaza),
`tools/wem_lint.py`.

> **Razlamanje registara je zatečeno stanje, ne odobrena norma** (D-03): `FR.md` i `NFR.md`
> razlomljeni su 2026-08-24 u `02-FRQ/` i `03-NFRQ/` bez DR zapisa. `workflow/hr/FR.md`,
> `workflow/hr/NFR.md` i `workflow/hr/METHODOLOGIA.md` su stubovi dok se ulazne reference ne
> prevedu; brisanje vodi `workflow/TODO.md`.

**Pravila:**

- **Prikaz nije izvor istine (D-13).** `DICTIONARY.md` se generira iz `dictionary.yaml` ili
  se prema njemu verificira; isto vrijedi za dijagrame i generirane preglede.
- **Pet kontroliranih vokabulara (D-12):** identifikatori koda (`tools/dictionary.json`),
  pojmovi diskursa (`dictionary.yaml`), reference (`LITERATURE.md`, **append-only** — novi
  ključ isključivo na kraj), tvrdnje (`POSTULATE.md`), norme (`DOCTRINE.md`). Sukob značenja
  razrješava DR, ne prešutan izbor u tekstu.
- Registri se mijenjaju **kroz DR** (D-03); izmjena bez DR reference je nalaz.
- Dijagrami: **PlantUML** (izvor `.puml` + generirana slika). Praktični primjeri: Jupyter
  notebook (`.ipynb`).

### 3.2 Jezik

| Artefakt | Jezik | Napomena |
|---|---|---|
| Doktrinarni Markdown | **Hrvatski** do v1.0 | na v1.0 sve prelazi na UK English; norma, ne opis zatečenog — vidi bilješku |
| Jupyter notebookovi | **UK English** uvijek | Izvršivi primjeri za međunarodne korisnike |
| Komentari u kodu (`src/**/*.py`) | **UK English** uvijek | Vidi §2.4 |
| `tools/dictionary.json` (vokabular koda) | **UK English** | Publika je alat/kod |
| Razgovori s agentom | **Hrvatski** | |
| PlantUML dijagrami | **UK English** labele, hrvatski komentari | Labela = identifikator iz koda |

> **Zatečeno stanje se s ovom normom razilazi (nalaz pod D-02, deklariran po D-11).** Engleski
> tekst postoji prije v1.0 na tri mjesta: `PHILOSOPHY.md` (HR izvornik `workflow/hr/FILOZOFIJA.md`
> je razišao), indeksi `0-HLRQ-EN.md` / `0-FRQ-EN.md` nad hrvatskim zapisima, i **cijeli
> `03-NFRQ/` registar, koji HR izdanje uopće nema**. Razrješenje — proširiti §3.2 na dvojezični
> registar uz HR kao autoritativan, ili povući EN do v1.0 — traži DR (D-03); vodi se u
> `workflow/TODO.md`. Do odluke EN datoteke nose deklaraciju, ne odobrenje (D-05).

### 3.3 Stil pisanja

- Koncizno, kratko, jasno. Primjeri samo gdje su nužni.
- Bez ponavljanja onoga što kod već nosi (tip-hintovi, potpisi).
- Bez duplikata definicija — definicija živi u jednom registru, ostalo je referenca.
- **Proza govori o ulogama i obiteljima, ne o imenima klasa.** Popis imena po obitelji nosi
  ontološki rječnik (`bases` u kriteriju koda) — on je i mehanizam protiv *ontology creepa*,
  jer novo ime ulazi kroz DR, a ne pojavom u tekstu. Ime klase u prozi opravdano je samo kad
  je ono sámo predmet odluke; inače se piše obitelj (`root`, `blackboard`, `strategy`…).
  Posljedica: preimenovanje klase mijenja jedan zapis u rječniku, ne N mjesta u tekstovima.

### 3.4 Struktura prema poslovnoj analizi

Za svaki modul/komponentu odgovara se redom:

1. **Zašto?** (poslovna motivacija, problem) — primarni fokus
2. **Što?** (zahtjevi — FR i NFR, s identifikatorima)
3. **Kako?** (dizajn, implementacija)

Zahtjevi su **primarni** (iz načela i ciljeva; prethode odlukama i ograničavaju ih) ili
**izvedeni** (nastaju iz odluke). Sljedivost je **graf**, ne lanac (METHODOLOGY §6).

### 3.5 UML notacija

Standardna UML notacija kroz PlantUML: class, sequence, activity, component, state, use case
dijagrami. Dijagram je **pogled** s deklariranim gledištem i publikom (D-13, ISO/IEC/IEEE
42010), nikad izvor istine.

### 3.6 Zahtjevi

- **FR** ([`02-FRQ/`](02-FRQ/0-FRQ-EN.md), `FR-<KATEGORIJA>-NN`, ISO/IEC/IEEE 29148): što sustav radi.
- **HLRQ** (`HLRQ-<NN>`): zahtjev visoke razine — narativ, poslovna pravila `BR-nn` i opseg za
  skupinu FR-ova ([`01-HLRQ/`](01-HLRQ/0-HLRQ-EN.md)). Razred je **u uporabi, ali nije u
  registru** — uvođenje traži DR (D-12).
- **NFR** ([`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md), `NFR-<KATEGORIJA>-NN`, sidreno na ISO/IEC 25010): koliko dobro radi.
- Svaki zahtjev nosi kriterije prihvaćanja, metodu verifikacije i sljedivost prema gore.
- Mjerni kriteriji nose oznaku **`[M]`** i podliježu povelji `NFR-DEF-02` (tip skale,
  dijagnostička uporaba, tranzitivno zatvorenje, promocija u gate samo kroz DR).

---

## 4. Redoslijed rada

1. `CLAUDE.md` / policy sloj
2. Doktrinarni okvir: `PHILOSOPHY`, `METHODOLOGY`, `DOCTRINE`, `POSTULATE`, `LITERATURE`,
   rječnik
3. Registri zahtjeva: `03-NFRQ/`, `02-FRQ/`, `01-HLRQ/`
4. Dokumentacija `src/wattleflow/concrete/` (radna osnova) i `core/` (sloj sučelja)
5. Iterativno proširivanje uz funkcionalnost frameworka

Dokle se stiglo vodi `workflow/TODO.md`, ne ovaj popis. Faze se ne preskaču. Optimizacije i refaktori u `core/` ne predlažu se bez pitanja korisniku.

**Neriješene odluke (ne pretpostavljati):**

- **Test framework i CI** — nije odabran; do tada „strojno provjerljivo" znači lint na zahtjev
- **`DR-WFL-004`** — casing akronima u identifikatorima (otvoren; pravilo suspendirano na WARNING)
- **Predložak DR-a** — polje *Temelji* nije u predlošku; vidi `DOCTRINE.md` §Bilješke t.4
- **Model odlučivanja** — jedan autor (D-01); prijelaz na konsenzus ide kroz DR

---

## 5. Uloga Claude Code agenta

**Zadaće:** generiranje boilerplatea, dijagrama i artefakata prema standardima; refaktoriranje
uz očuvanje sučelja; dokumentacija prema §3; testovi kad framework bude odabran.

**Kritičko razmišljanje vrijedi jednako za čovjeka i za agenta** (METHODOLOGY §9):

1. **Nema odluka na autoritet ili naviku.** Osobna preferencija nije opravdanje (D-05);
   tvrdnja se brani načelom, standardom, rezoniranjem ili dokazom.
2. **Kod nedoumice — pitati**, izložiti trošak i alternative; ne pogađati.
3. **Agent ne donosi arhitektonske odluke samostalno**, ne mijenja `core/` (§2.5) i ne
   proširuje kontrolirane vokabulare bez DR-a (D-03, D-12).
4. **Nalaz bez trojke reproducibilnosti nije nalaz** (D-10): alat + kriterij + platforma.
5. **Nema skalarne ocjene** kvalitete/konformnosti — izlaz je vektor po dimenzijama (D-09);
   gate stoji samo na binarnim pravilima, indeksi su dijagnostički.
6. **Slijepe pjege se deklariraju, ne prešućuju** (D-11); tiho nestala deklarirana iznimka
   je nalaz.
7. **Konformnost nije kvaliteta** (P-19): zeleni vektor dokazuje usklađenost s deklariranim
   kriterijem, ništa više.

---

## 6. Integrirani standardi i okruženje

### 6.1 OSCAL

NIST standard za strojno čitljive sigurnosne kontrole. Provodi se dekoratorima
`@oscal_connection`, `@oscal_driver`, `@oscal_processor` (`wattleflow.decorators.oscal`) nad
specijalizacijama; dekorater čita `OSCAL_CONTROLS` ClassVar i izvršava `OSCALPolicy.verify()`
(semantika: `declared ⊆ baseline`, uz crosswalk translaciju taksonomije).

**Cijeli sloj usklađenosti živi u `wattleflow-processors`** (`DR-WFL-015`): paket
`wattleflow.oscal`, dekorateri i PSPF dekorater. Clean core distribucije nemaju nijednu OSCAL
referencu ni ovisnost; tko traži provjeru kontrola, instalira processors — dakle paket koji
nije zero-trust (§7.4). Svaki dizajn dokument komponente koja nosi dekorater mora naznačiti
njezinu OSCAL ovisnost i mjesto enforcementa.

> **Dva deklarirana nalaza (provjereno 2026-08-22), ne stanje koje ovaj odjeljak propisuje**
> (D-02, D-11): (1) dekorateri stoje na **konkretnim klasama**, a moduli `connections/oscal.py`,
> `drivers/oscal.py` i `processors/oscal.py` ne postoje — `FR-OSCAL-14.13` opisuje suprotno i
> vodi se kao „Provedeno"; (2) **vrata su inertna** — provjera prethodi konstrukciji
> (`DR-WFL-014`), ali se uz `strict=False` ne izvodi dok nijedno mjesto ne predaje
> `oscal_policy=`; tko predaje aktivnu politiku otvoreno je pitanje (`HLRQ-14` §7 t.2).

### 6.2 SIEM audit

Prosljeđivanje audit informacija vanjskim sustavima (kontrola, monitoring, dijagnostika).
> **Status: aspiracija (D-05, D-15).** Nema namjenskog podsustava; postoji samo asinkroni
> handler i točka pretplate u `logger` obitelji temeljnog sloja — dakle mjesto na koje bi se
> prosljeđivanje priključilo, ne i samo prosljeđivanje. Politika bez provedbenog mehanizma i
> evaluacijskog signala vodi se kao aspiracija, ne kao politika na snazi.

### 6.3 Docker okruženje

Razvoj, testiranje i primjeri rade u kontejnerima (orkestrirani podovi); Jupyter primjeri
nose vlastite Docker konfiguracije. Reproducibilnost dev↔prod je dizajn cilj i dio trojke
iz D-10 (platforma).

### 6.4 Observability

Metrike i dashboardi (Grafana i sl.) dolaze inkrementalno.
> **Status: aspiracija.** Format metrika i transport (Prometheus / OpenTelemetry / statsd)
> nisu odlučeni. Svaka uvedena metrika podliježe protokolu valjanosti V1–V6 (METHODOLOGY
> §5.1) i povelji `[M]`.
>
> **Audit zapis nije aspiracija:** razina, imena polja i volumen po jedinici posla uređeni su
> registrom (`NFR-OBS-01/02/03`, `DR-WFL-018`) i mjere se lintom. Sadržaj zapisa ostaje pod
> `DR-WFL-008` (`AU-3`), povjerljivost pod `NFR-SEC-06`.

---

## 7. Zero-trust: lokalnost distribucije (NFR-SEC-01/02/03)

Epistemička jezgra: **povjerenje se ne pretpostavlja nego dokazuje** (D-07). Imenovana
sigurnosna politika koja iz toga slijedi živi ovdje i razrađena je u `03-NFRQ/`
(SEC-01 blast radius, SEC-02 napadna površina, SEC-03 supply-chain i lokalnost distribucije;
SEC-03 je nasljednik nekadašnjeg `NFR-ORG-06`). Odluke: `DR-WFL-002`, `DR-WFL-003`.

### 7.1 Pravilo

**Matična distribucija modula određena je njegovim import-closureom, ne ulogom.** Modul
pripada clean core distribuciji (`wattleflow`, `wattleflow-workflow`) **samo ako** mu cijeli
tranzitivni closure staje u tier te distribucije:

> **clean core tier = stdlib ∪ `wattleflow` ∪ allowlist iz `tools/dictionary.json`
> (`scope.core_libraries`).** Sadržaj allowliste i pin core ovisnosti čitati iz registra
> odnosno `pyproject.toml` — vrijednosti se ovdje ne prepisuju.

Modul koji — eager **ili lazy** — referira third-party paket pripada ne-core distribuciji
(`wattleflow-processors`, `wattleflow-cad`). Lazy-loading smanjuje import-time trošak, ali
**ne mijenja** matičnu distribuciju.

**Iznimka — čuvana opcionalna ovisnost (`DR-WFL-003`).** Referenca zaštićena
`try/except ImportError` granom čiji je fallback **funkcionalno potpun** i unutar tiera ne
izmješta modul. Kriterij je potpunost fallbacka, ne postojanje `try/except`-a; verifikacija je
**test maskiranja** (maskiraj paket → modul se mora uvesti i raditi), ne pregled koda.
Popis nositelja iznimke vodi `DR-WFL-003`.

### 7.2 Posljedice za dizajn

- `core/` i `concrete/` moraju biti samostalni — bez importa iz `connections/`, `drivers/` itd.
- Specijalizacije nasljeđuju iz `concrete/` (jednosmjerna ovisnost).
- **Jedinstveno vlasništvo:** svako `wattleflow.*` podstablo pakira točno jedna distribucija;
  svaka distribucija deklarira manifest podstabala.
- **Bez symlinkova u pakiranju**; dev koristi editable installove (PEP 420/660).
- `MANIFEST.in` je jedina kontrolna točka pakiranja (`DR-WFL-006`): svaki paket koji
  `packages.find` deklarira mora biti i slan, inače se instalira prazan.
- Integritet vlastitih modula: **wheel `RECORD`** (per-file `sha256`) se verificira, ne
  re-implementira; za dev/editable stabla `RECORD` je prazan → digest-scan u lintu.
- Third-party integritet: hash-pinned lock + SBOM (CycloneDX/SPDX), ne vlastiti mehanizam.

### 7.3 Dokumentacijska obveza
- Svaka specijalizacija u izdvojenom projektu eksplicitno navodi third-party ovisnosti.
- README izdvojenog projekta nosi sigurnosno upozorenje: komponente su primjeri; korisnik je
  odgovoran za audit svake instalirane ovisnosti.

### 7.4 Iznimka: `wattleflow-processors` NIJE zero-trust paket

`wattleflow-processors` je **namjerno izvan** opsega §7.1. Sadrži specijalizacije
(`connections/`, `drivers/`, `processors/`, `pipelines/`, `documents/`, `strategies/`,
`blackboards/`) oslonjene na lazy-loading third-partyja (psycopg2, kafka-python, pyspark,
pysolr, paramiko, pytesseract…) te **sloj usklađenosti** (`oscal/`, `decorators/oscal/`,
`decorators/pspf.py`) koji je ovamo prešao odlukom `DR-WFL-015`.

**Razlog:** paket je primjer implementacije nad postojećim open-source pod-sustavima; uvodi
third-party napadnu površinu i zato ne pripada clean core paketima.

**Odgoda mora preživjeti agregat (`DR-WFL-007`).** Odgođeno učitavanje u modulu ne vrijedi
ništa ako ga paketni `__init__.py` poništi eager re-exportom: uvoz bilo kojeg imena tada
povlači cijeli tier. Paketi ovog paketa izlažu javni API preko `__getattr__` (§2.7).

**Obveza:** istaknuti to u svakom opisu i dizajn dokumentu (README, docs, PyPI opis):
instalacija povlači opcionalne third-party ovisnosti; korisnik je odgovoran za audit svake.

> Zero-trust pravila §7.1–§7.3 i dalje vrijede za `wattleflow` i `wattleflow-workflow`.

---

## 8. Zapisi odluka (DR)

Evidencija odluka je **obvezna funkcija** (D-14); DR format je zamjenjiva pod-metoda.

- **Predložak:** Status / Kontekst / Odluka / Ugovor / Cijena / Svjedočanstvo / Registar
  (+ Povijest gdje se odluka mijenjala). *Svjedočanstvo* razdvaja dokazano od aspiracije;
  *Registar* veže odluku uz strojno provjeriv kriterij.
- **Oznake — serija po projektu:** `DR-COR` (core), `DR-WFL` (workflow), `DR-PRC`
  (processors), `DR-CAD` (cad). Odluka pripada seriji **onog projekta čiji artefakt mijenja**.
  Neprefiksirana oznaka nije valjana.
- **Indeksi vode opseg serija, ne ovaj tekst:** [`workflow/dr/DR-WFL-INDEX.md`](workflow/dr/DR-WFL-INDEX.md)
  i [`workflow/core/dr/DR-INDEX.md`](workflow/core/dr/DR-INDEX.md). Serija `DR-PRC` otvorena je
  2026-08-20 ([`04-DR/`](04-DR/)) i **još nema indeks** — deklarirana rupa (D-11).
- **Kad je DR obvezan:** izmjena politike, ugovora, kriterija ili članka doktrine; proširenje
  kontroliranog vokabulara; promocija `[M]` metrike u gate; promjena matične distribucije
  modula.
- **`documentation` je lokalni repozitorij.** Push na `origin` je namjerno onemogućen
  (`git remote set-url --push origin DISABLED-local-only-repository`); `fetch` radi. Hrvatski
  tekstovi ostaju lokalni, a objavljuje se tek **UK English** izdanje na v1.0 (§3.2) — i to
  kao zaseban, namjeran čin, ne kao nuspojava `git push`-a.
- **Preostale slijepe pjege (D-11):** u **core** repozitoriju je `documentation/` i dalje
  gitignoriran, pa je `DR-COR` serija tamo nepraćena — ovdje živi njezin primjerak
  (`workflow/core/dr/`). Hrvatski doktrinarni tekst je uz to **već objavljen** na
  `github.com/wattleflow/documentation` (do `v0.0.4`), prije nego što je push onemogućen;
  zatečeno stanje, vodi se u `workflow/TODO.md`.

**Granica evidencije — `v0.0.0.92`.** Razvoj do te verzije tekao je bez ovih smjernica, pa je
razilaženje koda i dokumentacije **zatečeno stanje**, ne nalaz. Postojeći DR zapisi vrijede i
odnose se na razdoblje prije nje; ne rekonstruiraju se unatrag. Od `v0.0.0.93` nadalje izmjena
ostavlja trag **na mjestu izmjene**: komentar uz kod nosi verziju i oznaku odluke, a DR nosi
tag kao svjedoka. Konsolidacija zatečenog ide postupno i vodi se u worklistu, ne kao propust.

---

## 9. Konformnost i mjerenje

Konformnost se **ne prosuđuje nego mjeri**, primjenom metode koja zadovoljava anatomiju iz
`METHODOLOGY.md` §2. Ovaj sloj propisuje samo što mjerenje **obvezuje**; *kako* se izvodi
opisuje metoda, a *čime* — dokument pripadnog alata.

- **Verdikt obvezuje:** `ERROR` ruši build; `WARNING` je tolerirano zatečeno stanje i ide u
  worklist; `INFO` je opažanje.
- **Kriterij je odvojen od alata i verzioniran zasebno** — izmjena kriterija ne smije proći
  kao nadogradnja alata, ni obrnuto. Proširenje kontroliranog vokabulara ide kroz DR (D-03).
- **Izlaz je vektor po dimenzijama; ukupna ocjena ne postoji** (nominalna skala, D-09).
- **Nalaz bez trojke reproducibilnosti nije nalaz** (D-10) — uz alat, kriterij i platformu
  trojka nosi i **mjereno stablo** i **pokrenuta pravila**, jer parcijalni `--select` inače daje
  vektor oblika punog runa. Zeleni run arhivira se kao **C-snimka**; run s greškama kao
  **finding-vector**, nikad kao C-snimka.
- **Prezentacija nije član trojke** (D-13): tekst poruke versionira se odvojeno od kriterija i
  ne mijenja verdikt.
- **Nemjereno se ne smije čitati kao čisto** — modul koji alat ne obradi daje vlastiti nalaz.
- **Ne navoditi brojeve nalaza u prozi** — prikaz nije izvor istine (D-13); referirati snimku.
- **Mjera ulazi u upotrebu tek sa statusom po protokolu valjanosti** (`METHODOLOGY.md`
  §5.1): minimalno tip skale, pokrivenost i dijagnostička/upravljačka uloga.

| Sloj | Nosi | Dokument |
|---|---|---|
| metoda | ponovljivi test, anatomija, valjanost mjere | `METHODOLOGY.md` |
| alat | pokretanje, prekidači, opseg, ključevi registra | `tools/README.md` |
| kriterij | vokabular koda, verzija kriterija | `tools/dictionary.json` |
| rezultat | C-snimke i finding-vektori | `workflow/conformance/` |

## 10. Reference

| Artefakt | Uloga |
|---|---|
| `src/wattleflow/core/` (core repo) | autoritativna sučelja i dizajn patterni (serija `DR-COR`) |
| `src/wattleflow/concrete/` | generičke implementacije — radna osnova frameworka |
| `pyproject.toml`, `MANIFEST.in` | metapodaci i pakiranje (`DR-WFL-006`) |
| `documentation/` | doktrinarni okvir i registri (§3.1) |
| `documentation/workflow/dr/` | zapisi odluka + indeks |
| `documentation/workflow/conformance/` | C-snimke konformnosti |
| `tools/wem_lint.py`, `tools/dictionary.json` | provedba NFR-ORG-01/02/03/07, SEC-03 i vokabular koda |
| `tools/messages.json` | prezentacija nalaza (en/hr), versionirana odvojeno od kriterija |
| `documentation/workflow/Analiza.md` | istraživački rad — temelj `[M]` povelje i SEC zahtjeva |
| `examples/` (symlink) | vanjski primjeri, uključujući Docker konfiguracije |
| `workflow/TODO.md` | worklist (stanje rada, ne norma) |
