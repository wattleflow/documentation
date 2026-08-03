# CLAUDE.md

Obvezujuće upute i standardi za rad na projektima `wattleflow` (core), `workflow`,
`processors`, `cad`. Vrijedi za sve sesije s Claude Code agentom.

**Položaj u kaskadi (D-02).** Ovo je **policy sloj**: filozofija → *policy* → princip →
metoda. Nadređeni dokumenti su [PHILOSOPHY.md](PHILOSOPHY.md) (kišobran) i
[DOCTRINE.md](DOCTRINE.md) (registar normi `D-01…D-17`); operacionalizacija je
[METHODOLOGY.md](METHODOLOGY.md); provedivi zahtjevi su u [NFR.md](NFR.md) i
[FR.md](FR.md). `POLICY.md` je isti dokument (symlink) — doktrinarni tekstovi ga zovu
`POLICY.md`, repozitoriji `CLAUDE.md`.

**Pravila upravljanja ovim dokumentom:**

- Izmjena policyja ide **kroz zapis odluke (DR)**, ne prešutnim uređivanjem (D-03).
- Niži sloj ne nadjačava viši: gdje se ovaj dokument razilazi s `DOCTRINE.md` ili
  `NFR.md`, prednost ima viši sloj/registar, a razilaženje je **nalaz** koji se
  prijavljuje, ne rješava u tekstu (D-02, D-12).
- Tvrdnja bez svjedočanstva vodi se kao **aspiracija** i tako se označava (D-05).

---

## 1. Svrha projekta

`core` je Python skup sučelja (ugovora) baziran na dizajn patternima; nad njim se grade
sustavi poput **`wattleflow-workflow`** — frameworka za podatkovno inženjerstvo. Radna
osnova frameworka je **`src/wattleflow/concrete/`** (generičke implementacije sučelja iz
`src/wattleflow/core/`). Cilj je deklarativna izgradnja cjevovoda
(workflow → pipeline → processor → strategy) nad heterogenim izvorima.

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
ne izmišljaju se novi bez DR-a (METHODOLOGY §5, NFR-ORG-04).

**Publika dokumentacije:** dokumentacija je **sustav s više publika** (D-16) — arhitekt,
implementator, tester, sigurnosni analitičar, integrator, poslovni korisnik, revizor.
Jedan format za sve publike nije legalan cilj; publika bez artefakta deklarira se kao rupa.

---

## 2. Tehnički standardi

### 2.1 Python

- **Minimalna verzija:** Python **3.11** (`requires-python = ">=3.11"` u `pyproject.toml`;
  usklađeno s classifiers i `ruff target-version = "py311"`). Politika verzija: `DR-COR-013`.
- **Ciljane verzije:** 3.11, 3.12, 3.13.
- **Posljedice za kod:**
  - **PEP 604** union sintaksa (`X | Y` umjesto `Union[X, Y]`).
  - **PEP 585** generici iz `builtins` (`list[int]`, `dict[str, Any]`).
  - **PEP 634** `match/case` gdje povećava jasnoću.
  - **PEP 612** `ParamSpec` za tipizaciju dekoratora.

### 2.2 PEP-ovi koji se primjenjuju

| PEP | Naziv | Status |
|---|---|---|
| PEP 8 | Style Guide for Python Code | obvezno |
| PEP 20 | The Zen of Python | smjernica |
| PEP 484 | Type Hints | obvezno za javna sučelja |
| PEP 526 | Variable Annotations | preporučeno |
| PEP 585 | Type Hinting Generics in Standard Collections | obvezno |
| PEP 604 | Allow writing union types as X \| Y | obvezno |
| PEP 621 | Storing project metadata in pyproject.toml | obvezno |
| PEP 257 | Docstring Conventions | **ograničeno** — vidi §2.4 |
| PEP 420 / PEP 660 | Namespace paketi / editable installovi | obvezno za dev-stabla (§7, NFR-SEC-03) |

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
- Docstring javnog sučelja: 1–2 rečenice, bez verboznih reST blokova.
- Tip-hintovi zamjenjuju većinu dokumentacijskih potreba.

### 2.5 Sučelja iz `core/`

- Moduli u `src/wattleflow/core/` su **autoritativni**; mijenjaju se samo kroz DR
  (serija `DR-COR`).
- Konkretne implementacije idu u `src/wattleflow/concrete/`.
- Svaka nova klasa nasljeđuje odgovarajuće apstraktno sučelje iz `core/`.

### 2.6 Organizacija modula (specijalizacije)

| Situacija | Layout |
|---|---|
| Mali broj povezanih klasa (~1–3) | **Jedna `.py` datoteka**. Primjer: `helpers/datetime.py`. |
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

### 2.7 Javni API: `__all__` i `import *` (odluka 2026-06-12)

Standard vrijedi za sve `wattleflow-*` pakete (PEP 8 „Public and internal interfaces").

1. **Svaki modul i `__init__.py` deklarira `__all__`** — eksplicitna javna API površina;
   neizloženo ostaje privatno (NFR-SEC-02 t.2).
2. **`from wattleflow.<paket> import *` je podržan first-class ulaz** — `__all__` mora biti
   potpun i točan.
3. **Paketni `__init__.py` re-exporta javne klase pod-modula** agregacijom, radi izbjegavanja
   drifta:
   ```python
   from . import sub_a, sub_b
   from .sub_a import *  # noqa: F403
   from .sub_b import *  # noqa: F403
   __all__ = [*sub_a.__all__, *sub_b.__all__]
   ```
   Ručno prepisivanje imena u roditeljski `__all__` je zabranjeno (uzrok bugova: ime u
   `__all__` bez odgovarajućeg importa).

**Ograničenje zbog lazy-loadinga (§7.4):** `import *` razrješava *svako* ime u `__all__`, pa
eager re-export pod-modula koji **na vrhu modula** uvozi third-party (npr.
`strategies/documents/dataframe.py` → pandas, `graph.py` → rdflib, `pipelines/nlp/*` →
spacy/flair) čini roditeljski paket neuvozivim bez tih ovisnosti.

Pravilo pomirbe:
- **Lagani pod-moduli** (stdlib + deklarirane core ovisnosti) → re-export odmah.
- **Teški pod-moduli** (third-party na vrhu) → re-export **odgođen** uz `# NOTE` u
  `__init__.py` s razlogom, dok se uvozi ne spuste u metode (§7.4).

**Kanonsko rukovanje third-party-ovisnim pod-modulima.** Pod-modul koji ovisi o third-party
paketu — ili je specijalizacija koje **nema** u core kopiji istog paketa (npr. `blackboards/`
dijele core i processors, ali `claude.py` postoji samo u processors) — **ne re-exporta se
eagerno**; pristupa mu se eksplicitnim putem
(`from wattleflow.blackboards.claude import ClaudeBlackboard`), uz `# NOTE` u `__init__.py`.

**Iznimka — cijeli direktorij je third-party.** Kad SVI moduli paketa ovise o third-party
paketima (`drivers/`, `connections/`, `pipelines/`, `processors/`), paket je u cijelosti
processors-specifičan → primjenjuje se standardni eager `__all__` re-export. Kriterij po
paketu: *je li paket dio core distribucije?* Da → mješovit (ručni import za third-party
module). Ne → eager `__all__`.

**Cross-distribucijski uvozi** (namespace paketi, PEP 420): uvijek **eksplicitni submodul**,
nikad agregat/`__all__` (vidi TODO „PEP 420 namespace migracija").

### 2.8 Imenovanje — provodi se lintom

Imenovanje je **semiotička politika**, ne stilska preferencija (P-08): ime je znak čiji odnos
prema ulozi registar fiksira, a lint provodi.

- **Klase — NFR-ORG-02.** Pipeline klase:
  `Pipeline<Subject>(<Operation>|To<Target>)(<Qualifier>)?`; Subject = kanonski subjekt
  domenskog paketa u kojem klasa živi. Pomoćne klase: bez `Pipeline` prefiksa, domenski
  kvalificirane. **Gole generičke uloga-imenice** (`Manager`, `Helper`, `Utility`, `Parser`,
  `Piece`, `Sheet`) zabranjene su kao samostalna imena; dopuštene su kvalificirano
  (`DriverS3UriParser`).
- **Tipske varijable — NFR-ORG-03.** Ime imenuje **ulogu** (`Key`, `Value`, `Input`,
  `Output`, `Result`, `State`…), bez mehanizam-sufiksa (`T`, `Type`, `_t`); goli `T` samo za
  jedan neograničen parametar; varijanca se ne kodira u imenu.
- **Vokabular je kontroliran** (`tools/dictionary.json`): novi facet, domena, akronim ili
  uloga ulaze **kroz DR**, ne dopisivanjem u registar radi „zelenog" linta.
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

Referentni obrazac: `MailAttachmentLayout` (`strategies/documents/mail.py`). AST-popis
zatečenih kandidata (2026-07-09): **28** mjesta. **Status:** `processors` (8) ✅,
`workflow` (14/17; 3 odgođena — `cls`-param iznimka) ✅; preostaje `cad` (3).

---

## 3. Standardi dokumentacije

### 3.1 Stablo i izvori istine

Dokumentacija živi u repozitoriju **`documentation/`** (nema `docs/` stabla u ovom projektu):

```
documentation/
├── PHILOSOPHY.md   → workflow/hr/FILOZOFIJA.md    kišobran (v0.4.1): kaskada, hipoteze H1–H3
├── METHODOLOGY.md  → workflow/hr/METHODOLOGIA.md  Svezak I — Temelji (v0.3.1)
├── DOCTRINE.md                                    registar normi D-01…D-17
├── POSTULATE.md                                   registar postulata P-01…P-21
├── LITERATURE.md                                  konsolidirane reference (append-only)
├── FR.md           → workflow/hr/FR.md            FR registar (trenutno prazan — aspiracija)
├── NFR.md          → workflow/hr/NFR.md           NFR-ORG-01…05, NFR-SEC-01…05, Dodatak A
├── DICTIONARY.md   → workflow/hr/RIJECNIK.md      generirani prikaz rječnika
├── dictionary.yaml                                rječnik diskursa (izvor istine, HR)
├── dictionary-processors.yaml                     isto, za processors distribuciju
├── CLAUDE.md / POLICY.md                          ovaj dokument (policy sloj)
├── core/ , oscal/ , processors/                   po-projektni tekstovi
└── workflow/
    ├── dr/            DR-WFL serija + DR-WFL-INDEX.md
    ├── conformance/   C-snimke (vektor + trojka reproducibilnosti)
    ├── analysis/      analize, index znanja, zero-trust ADR
    ├── changes/       zapisi usklađenja s core izdanjima
    ├── concrete/      nacrti uz concrete sloj
    └── hr/ , en/      izvorni (HR) i budući (EN) tekstovi
```

Uz njih, u kodnom repozitoriju: **`tools/dictionary.json`** (kontrolirani vokabular **koda**,
UK English — kriterij koji čita lint), `tools/messages.json` (prezentacija nalaza),
`tools/wem_lint.py`, `tools/Analiza.md` (istraživački rad — bibliografija za `[M]`/SEC
zahtjeve).

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
| Doktrinarni Markdown (`documentation/**/hr/*.md`) | **Hrvatski** do v1.0 | EN inačica pri prelasku na v1.0 (`workflow/en/`) |
| Jupyter notebookovi | **UK English** uvijek | Izvršivi primjeri za međunarodne korisnike |
| Komentari u kodu (`src/**/*.py`) | **UK English** uvijek | Vidi §2.4 |
| `tools/dictionary.json` (vokabular koda) | **UK English** | Publika je alat/kod |
| Razgovori s agentom | **Hrvatski** | |
| PlantUML dijagrami | **UK English** labele, hrvatski komentari | Labela = identifikator iz koda |

### 3.3 Stil pisanja

- Koncizno, kratko, jasno. Primjeri samo gdje su nužni.
- Bez ponavljanja onoga što kod već nosi (tip-hintovi, potpisi).
- Bez duplikata definicija — definicija živi u jednom registru, ostalo je referenca.

### 3.4 Struktura prema poslovnoj analizi

Za svaki modul/komponentu odgovara se redom:

1. **Zašto?** (poslovna motivacija, problem) — primarni fokus
2. **Što?** (zahtjevi — FR i NFR, s identifikatorima)
3. **Kako?** (dizajn, implementacija)

Zahtjevi su **primarni** (iz načela i ciljeva; prethode odlukama i ograničavaju ih) ili
**izvedeni** (nastaju iz odluke). Sljedivost je **graf**, ne lanac (METHODOLOGY §7).

### 3.5 UML notacija

Standardna UML notacija kroz PlantUML: class, sequence, activity, component, state, use case
dijagrami. Dijagram je **pogled** s deklariranim gledištem i publikom (D-13, ISO/IEC/IEEE
42010), nikad izvor istine.

### 3.6 Zahtjevi

- **FR** (`FR.md`, `FR-<KATEGORIJA>-NN`, ISO/IEC/IEEE 29148): što sustav radi.
- **NFR** (`NFR.md`, `NFR-<KATEGORIJA>-NN`, sidreno na ISO/IEC 25010): koliko dobro radi.
- Svaki zahtjev nosi kriterije prihvaćanja, metodu verifikacije i sljedivost prema gore.
- Mjerni kriteriji nose oznaku **`[M]`** i podliježu povelji iz `NFR.md` (tip skale,
  dijagnostička uporaba, tranzitivno zatvorenje, promocija u gate samo kroz DR).

---

## 4. Redoslijed rada

1. ✅ `CLAUDE.md` / policy sloj
2. ✅ Doktrinarni okvir: `PHILOSOPHY`, `METHODOLOGY`, `DOCTRINE`, `POSTULATE`, `LITERATURE`,
   rječnik
3. 🔄 Registri zahtjeva: `NFR.md` (ORG-01…05, SEC-01…05) aktivan; **`FR.md` prazan** —
   popuniti ili deklarirati kao aspiraciju
4. ⏳ Dokumentacija `src/wattleflow/concrete/` (radna osnova) i `core/` (sloj sučelja)
5. ⏳ Iterativno proširivanje uz funkcionalnost frameworka

Faze se ne preskaču. Optimizacije i refaktori u `core/` ne predlažu se bez pitanja korisniku.

**Neriješene odluke (ne pretpostavljati):**

- **Test framework i CI** — nije odabran; do tada „strojno provjerljivo" znači lint na zahtjev
- **`DR-WFL-004`** — casing akronima u identifikatorima (otvoren; pravilo suspendirano na WARNING)
- **Predložak DR-a** — polje *Temelji* koje traži `DOCTRINE` D-01/D-05/D-15 nije u predlošku
  (Status/Kontekst/Odluka/Ugovor/Cijena/Svjedočanstvo/Registar); uskladiti kroz DR
- **Model odlučivanja** — jedan autor (D-01); prijelaz na konsenzus ide kroz DR

---

## 5. Uloga Claude Code agenta

**Zadaće:** generiranje boilerplatea, dijagrama i artefakata prema standardima; refaktoriranje
uz očuvanje sučelja; dokumentacija prema §3; testovi kad framework bude odabran.

**Kritičko razmišljanje vrijedi jednako za čovjeka i za agenta** (METHODOLOGY §10):

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

> **Status (2026-06-12):** OSCAL paket **nije deployan**; dok nije, `wattleflow.oscal.*` se ne
> razrješava i uvoz `connections/` / `drivers/` / `processors/` modula puca. Svaki dizajn
> dokument mora naznačiti OSCAL ovisnost komponente i mjesto enforcementa.

### 6.2 SIEM audit

Prosljeđivanje audit informacija vanjskim sustavima (kontrola, monitoring, dijagnostika).
> **Status: aspiracija (D-05, D-15).** `src/wattleflow/audit/` je prazan; postoji samo
> `AsyncHandler` + `AuditLogger.subscribe_handler()` u `concrete/logger.py`. Politika bez
> provedbenog mehanizma i evaluacijskog signala vodi se kao aspiracija, ne kao politika na snazi.

### 6.3 Docker okruženje

Razvoj, testiranje i primjeri rade u kontejnerima (orkestrirani podovi); Jupyter primjeri
nose vlastite Docker konfiguracije. Reproducibilnost dev↔prod je dizajn cilj i dio trojke
iz D-10 (platforma).

### 6.4 Observability

Metrike i dashboardi (Grafana i sl.) dolaze inkrementalno.
> **Status: aspiracija.** Format metrika i transport (Prometheus / OpenTelemetry / statsd)
> nisu odlučeni. Svaka uvedena metrika podliježe protokolu valjanosti V1–V6 (METHODOLOGY
> §3.1/§6.1) i povelji `[M]`.

---

## 7. Zero-trust: lokalnost distribucije (NFR-SEC-01/02/03)

Epistemička jezgra: **povjerenje se ne pretpostavlja nego dokazuje** (D-07). Imenovana
sigurnosna politika koja iz toga slijedi živi ovdje i razrađena je u `NFR.md`
(SEC-01 blast radius, SEC-02 napadna površina, SEC-03 supply-chain i lokalnost distribucije;
SEC-03 je nasljednik nekadašnjeg `NFR-ORG-06`). Odluke: `DR-WFL-002`, `DR-WFL-003`.

### 7.1 Pravilo

**Matična distribucija modula određena je njegovim import-closureom, ne ulogom.** Modul
pripada clean core distribuciji (`wattleflow`, `wattleflow-workflow`) **samo ako** mu cijeli
tranzitivni closure staje u tier te distribucije:

> **clean core tier = stdlib ∪ `wattleflow` ∪ allowlist iz `tools/dictionary.json`
> (`scope.core_libraries`, trenutno `typing_extensions`).**
> Runtime ovisnost workflow distribucije je `wattleflow>=0.0.0.47` (`pyproject.toml`).
> `cryptography` **nije** core ovisnost.

Modul koji — eager **ili lazy** — referira third-party paket pripada ne-core distribuciji
(`wattleflow-processors`, `wattleflow-cad`). Lazy-loading smanjuje import-time trošak, ali
**ne mijenja** matičnu distribuciju.

**Iznimka — čuvana opcionalna ovisnost (`DR-WFL-003`).** Referenca zaštićena
`try/except ImportError` granom čiji je fallback **funkcionalno potpun** i unutar tiera ne
izmješta modul. Kriterij je potpunost fallbacka, ne postojanje `try/except`-a; verifikacija je
**test maskiranja** (maskiraj paket → modul se mora uvesti i raditi), ne pregled koda.
Nositelji: `helpers/config.py`, `helpers/config_adapter.py`, `helpers/config_validator.py`.

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

- ADR/analiza: `documentation/workflow/analysis/zero-trust-architecture.md`.
- Svaka specijalizacija u izdvojenom projektu eksplicitno navodi third-party ovisnosti.
- README izdvojenog projekta nosi sigurnosno upozorenje: komponente su primjeri; korisnik je
  odgovoran za audit svake instalirane ovisnosti.

### 7.4 Iznimka: `wattleflow-processors` NIJE zero-trust paket

`wattleflow-processors` je **namjerno izvan** opsega §7.1. Sadrži specijalizacije
(`connections/`, `drivers/`, `processors/`, `pipelines/`, `documents/`, `strategies/`,
`blackboards/`) oslonjene na lazy-loading third-partyja (psycopg2, kafka-python, pyspark,
pysolr, paramiko, pytesseract…).

**Razlog:** paket je primjer implementacije nad postojećim open-source pod-sustavima; uvodi
third-party napadnu površinu i zato ne pripada clean core paketima.

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
- **`ADR` je zabranjen naziv za nove zapise** (zadržava se samo u povijesnim referencama).
- **Indeks:** `documentation/workflow/dr/DR-WFL-INDEX.md` (aktivna serija: `DR-WFL-001…006`).
- **Kad je DR obvezan:** izmjena politike, ugovora, kriterija ili članka doktrine; proširenje
  kontroliranog vokabulara; promocija `[M]` metrike u gate; promjena matične distribucije
  modula.
- **Verzijska kontrola doktrinarnih tekstova — riješeno 2026-08-03 (`documentation` v0.0.5).**
  Do tada su četiri registra (`NFR.md`, `METHODOLOGY.md`, `FR.md`, `DICTIONARY.md`) bila u
  gitu **samo kao symlink** (`mode 120000` — git je čuvao putanju, ne tekst, jer je
  `.gitignore` ignorirao `*/hr/*`), a `CLAUDE.md`/`POLICY.md` nije bio praćen uopće. D-03 je
  tražio izmjenu registra kroz DR, dok nije postojao zapis *da je izmjena nastala*. Sada je
  cijeli sadržaj `documentation/` repozitorija praćen.
- **`documentation` je lokalni repozitorij.** Push na `origin` je namjerno onemogućen
  (`git remote set-url --push origin DISABLED-local-only-repository`); `fetch` radi. Hrvatski
  tekstovi ostaju lokalni, a objavljuje se tek **UK English** izdanje na v1.0 (§3.2) — i to
  kao zaseban, namjeran čin, ne kao nuspojava `git push`-a.
- **Preostala slijepa pjega (D-11):** core serija DR zapisa i dalje nije pod verzijskom
  kontrolom — deklarirano, ne riješeno.

---

## 9. Konformnost i mjerenje

**Alat:** `python tools/wem_lint.py` (v1.12.0) — statička AST/import-graf provjera
NFR-ORG-01/02/03/07 i NFR-SEC-03. Radi bez ijedne third-party ovisnosti. Mjereni modul se
ne uvozi *radi provjere*, ali alat je **izgrađen nad `wattleflow.concrete`** — a to je
mjerena domena; samopozivanje je deklarirano u docstringu i posljedica je da lint ne može
mjeriti `concrete/` koji se ne uvozi. Modul koji se **ne parsira** daje vlastiti nalaz
(`unparsable-module`), neovisan o `--select`: nemjereno se ne smije čitati kao čisto.

- **Kriterij** se čita iz `tools/dictionary.json` (`--registry`) i **verzionira odvojeno od
  alata** (`criterion_version`). `naming_registry.yaml` je ukinut (`DR-WFL-005`); povijesne
  reference na tu putanju čitati kao `dictionary.yaml#code` → `tools/dictionary.json`.
- **Prezentacija** (`tools/messages.json`, `--messages`) versionira se odvojeno od kriterija
  i **nije** član trojke (D-13): tekst poruke ne mijenja verdikt.
- **Izlaz je vektor** po dimenzijama (ORG-01/02/03/07, SEC-03, EXC…) — **ukupna ocjena ne
  postoji** (nominalna skala; D-09).
- **Ozbiljnost:** `ERROR` ruši build (exit ≠ 0); `WARNING` je **tolerirano zatečeno stanje**
  (worklist za migraciju); `INFO` je opažanje.
- **Trojka reproducibilnosti obvezna** uz svaki nalaz (D-10): verzija alata, verzija kriterija,
  verzija platforme — uz **mjereno stablo** i **pokrenuta pravila**, jer parcijalni `--select`
  inače daje vektor oblika punog runa. Zelene runove arhivirati kao **C-snimke**
  (`--snapshot` → `documentation/workflow/conformance/`); run s greškama arhivira se kao
  `finding-vector`, nikad kao C-snimka.
- **Ne navoditi brojeve nalaza u prozi** — prikaz nije izvor istine (D-13); referirati snimku.
- Mjere ulaze u upotrebu tek sa statusom po protokolu V1–V6 (METHODOLOGY §3.1/§6.1);
  minimalno V2 (tip skale i dopuštene agregacije), V3 (pokrivenost), V6 (dijagnostičko/
  upravljačko).

---

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
| `tools/Analiza.md` | istraživački rad — temelj `[M]` povelje i SEC zahtjeva |
| `examples/` (symlink) | vanjski primjeri, uključujući Docker konfiguracije |

---

# TODO

Worklist; organizirano po cijeni izvedbe i riziku.

## Usklađivanje dokumentacije (nalaz 2026-07-30)

Nalazi iz analize `PHILOSOPHY` / `METHODOLOGY` / `DOCTRINE` / `FR` / `NFR`. Sve su izmjene
registara → idu kroz DR (D-03).

- [ ] **Ukinuti reference na `tools/naming_registry.yaml`** u `NFR.md` (4×), `FR.md`,
  `METHODOLOGIA.md` (§3b, §11) i `DOCTRINE.md` (D-12) → `tools/dictionary.json`.
- [ ] **Ispraviti putanje:** `docs/adr/helpers/DR-WFL-001` → `documentation/workflow/dr/…`;
  `docs/conformance/` → `documentation/workflow/conformance/`; `documentation/dr/`
  (FILOZOFIJA, tablica svezaka) → `documentation/workflow/dr/`; `tools/ANALIZA.md` →
  `tools/Analiza.md`; `documentation/dictionary.json` (POSTULATE, RIJECNIK) →
  `documentation/dictionary.yaml`; `documentation/literatura.md` (dictionary.yaml) →
  `LITERATURE.md`.
- [ ] **Ukloniti zastarjele brojke lint nalaza** („13 ERROR + 16 WARN" u `NFR.md`/`FR.md`) —
  zamijeniti uputom na C-snimku; nalaz bez trojke krši D-10.
- [ ] **Dvostruki registri (D-12/D-13):** `LITERATURE.md` (1–59) i `workflow/hr/LITERATURA.md`
  (1–57) dodjeljuju **ključevima 56/57 različite radove** → kršenje append-only pravila; isto
  drift `POSTULATE.md` (P-21) vs `workflow/hr/POSTULATI.md`. Odabrati jedan izvor po registru;
  uskladiti i layout (dio korijenskih dokumenata su symlinkovi na `hr/`, dio nisu).
- [ ] **Oznake odluka u `DOCTRINE.md`:** `DR-013/014/016/018` bez prefiksa; `DR-016`/`DR-018`
  nemaju zapis ni u jednom indeksu. Prefiksirati i otvoriti zapise ili označiti kao kandidate.
- [ ] **`DOCTRINE` D-02:** „suspendirano do `DR-015`" → `DR-WFL-004` (preimenovano u indeksu).
- [ ] **`METHODOLOGIA` Povijest izmjena / Bilješka t.2:** nulti rename „DR-WFL-004 →
  DR-WFL-004" → „`DR-015` → `DR-WFL-004`".
- [ ] **Predložak DR-a vs `DOCTRINE`:** polje *Temelji* (D-01/D-05/D-15) nije u predlošku →
  dodati u predložak ili uskladiti naziv sa *Svjedočanstvo*/*Registar*.
- [ ] **`NFR-ORG-06` više ne postoji** (dekomponiran u SEC-01/02/03, 2026-07-22): ažurirati
  `METHODOLOGIA` §8 i §11, `DOCTRINE` D-07 i zaglavlje `DR-WFL-002` → `NFR-SEC-03`.
  Usput: `NFR-SEC-03` citira „`DR-WFL-002/07`" — `07` je predrename oznaka (`DR-WFL-003`).
- [ ] **Akronimi — sukob registra i metode:** `NFR-ORG-02` kriterij 4 i `NFR-ORG-03` kriterij 4
  kažu „prekršaj / build pada", dok je pravilo suspendirano na WARNING do `DR-WFL-004`
  (`METHODOLOGIA` §9, `acronym_identifier_casing.status: undecided`). Kriteriji moraju nositi
  suspenziju (severity iz statusa) — inače metoda propisuje neodlučeno (D-02).
- [ ] **`ADR` ostaci u `NFR.md`** („pod ADR upravljanjem", „kroz ADR") — `ADR` je zabranjen za
  nove zapise; `NFR-ORG-03` već koristi „kroz DR".
- [ ] **`NFR-ORG-02`, implementacijske napomene:** `PipelinePDFExtractText →
  PipelinePDFExtractText` je identitet (izvorno ime izgubljeno); popis traži
  `PipelinePDFRedactText → PipelinePDFRedact`, a tablica u „Deduplikacija koda" vodi
  `PipelinePDFRedactText` — uskladiti worklist imena.
- [ ] **Duplicirane „Zajedničke definicije"** (domena, helper, kanonski subjekt) u `FR.md` i
  `NFR.md` — držati u jednom registru, drugi referira.
- [ ] **Kriva referenca sljedivosti:** `FR.md` i `NFR.md` upućuju na „`METHODOLOGY.md` §9
  *Arhitektonska sljedivost*"; §9 je *Samoopisivost i imenovanje*, sljedivost je **§7**.
- [ ] **`FR.md` je prazan** (`## FR-ORG-01 — ...`) iako ga FILOZOFIJA/METHODOLOGIA/DOCTRINE
  tretiraju kao aktivan registar (Svezak V) → popuniti ili deklarirati kao aspiraciju (D-05).
- [ ] **Nedovršene doktrinarne tvrdnje:** (a) `DOCTRINE` tvrdi da proza citira norme
  D-oznakama — nijedna D-oznaka nije u `PHILOSOPHY`/`METHODOLOGY`; (b) **H4-DQI** se koristi
  kao doktrinarna hipoteza, ali `PHILOSOPHY` §Hipoteze ima samo H1–H3; (c) `METHODOLOGIA` §3
  referira `pipelines/quality/dqi.py` koji u ovoj distribuciji ne postoji.
- [ ] **`POSTULATE.md` zaglavlje** navodi nadređene v0.4 / v0.3 → v0.4.1 / v0.3.1.
- [ ] **Kolizija DR oznaka u nacrtima:** `core/DR.md` i `workflow/concrete/DR.md` nose
  neprefiksirani `DR-001` (indeks ih vodi kao nacrte, jedan „sadržajno proturječan") →
  staviti banner „superseded" u same datoteke.
- [ ] **`DR-WFL-006` vs kod:** odluka pina `wattleflow>=0.0.0.46`, `pyproject.toml` ima
  `>=0.0.0.47` → zabilježiti kao verziju zapisa (D-03).

## Veće (značajna cijena ili arhitektonske odluke)

- [ ] **Migracija tip-hintova na PEP 585/604** — `pyupgrade --py311-plus`; 65 starih
  (`Optional`, `Union`, `List`, `Dict`, `Tuple`) prema 6 novih u `core/` + `concrete/`
- [ ] Implementirati SIEM forwarding (`audit/` prazan; iskoristiti `AsyncHandler` +
  `AuditLogger.subscribe_handler()`)
- [ ] OSCAL katalogizacija — `component-definition`, `assessment-results`, dodatni katalozi
- [ ] Observability — Prometheus / OpenTelemetry za FSM tranzicije i throughput (uz V1–V6)
- [ ] **Razdvajanje specijalizacija u zaseban projekt** — `connections/`, `drivers/`,
  `processors/`, `strategies/`, `pipelines/`, `documents/` izlaze iz clean core paketa
  (§7). **Regulira `DR-WFL-002` + `NFR-SEC-03`:** matična distribucija = najteža ovisnost;
  per-distribucija manifest; symlink → editable install (PEP 420/660); supply-chain preko
  SBOM/lock. Worklist preseljenja: `helpers/converters/`, `mappers/schema_yaml_json.py`.
  - [ ] **Self-integritet + shadowing gate (`NFR-SEC-03` kriterij 5).** Za izgrađene artefakte
    verificirati wheel `RECORD` (per-file `sha256`) — ne raditi vlastiti digest-format. Za
    dev/editable stabla `RECORD` je prazan → `wem_lint` digest-scan (`FileDigest`) koji
    istodobno detektira namespace-sjenčanje/koliziju.
  - [ ] **Dovršiti PEP 420 namespace migraciju (nastavak 2026-07-12).** `wattleflow.helpers`
    je namespace (uklonjen workflow `helpers/__init__.py`; 32 lib fajla na eksplicitnim
    submodul uvozima; bez symlinkova, cross-distribucijske klase uvijek eksplicitno, nikad
    agregat/`__all__`). **Preostaje:** (1) migrirati `examples/processors/*` + ručno riješiti
    nepostojeće simbole (`FileType`→`wattleflow.constants`, `LocalPath`/`Preset`);
    (2) `examples/todo/*` je dijelom pre-broken — počistiti ili arhivirati; (3) isti princip
    na ostale dijeljene subtree-eve (`decorators` itd.). Memorija: `helpers-pep420-namespace`.
- [ ] **Testovi** — odgođeno do odluke o test frameworku (§4)

## IWattleflow `__slots__` — MRO (nalaz 2026-06-29, djelomično riješeno)

MRO konflikt `LargeBlackboard(GenericBlackboard[Documents], IOriginator, ABC)` riješen je
Fixom B: `AuditLogger` je premješten ispred `Generic[T]` u `GenericBlackboard`, bez diranja
corea (§2.5). Redoslijed baza je zato **ograničenje, ne stil** — komentar to čuva na mjestu.

- [ ] **Fix A (niži prioritet)** — alternativa: uskladiti **core** `IOriginator` na
  `(Generic[State], IWattleflow, ABC)`. Dira autoritativni core (§2.5 → `DR-COR`) i ima širi
  domet; razmotriti ako se pojavi još C-builtin/MRO sudara.

## OSCAL crosswalk — čeka compliance sign-off

`oscal/crosswalk.py` prepisuje control-id iz izvorne taksonomije (NIST) u ciljnu (ISM) prije
provjere; unmapped ID-evi prolaze nepromijenjeni. Mapping je **kuriran compliance artefakt** —
ne izmišljati mapiranja.

- [ ] **Mapiranja su `PROPOSED`** (`status: proposed-requires-compliance-review`, 2026-05-31)
  i traže sign-off prije oslanjanja. Postgres deklaracija: `ac-3→ism-0445`, `ia-5→ism-1401`
  (E8 ML1); `sc-8→ism-0469`, `sc-13→ism-1080` (izvan Essential Eight opsega — protiv E8
  baselinea ispravno padaju).

## Konzistentnost i standardi

- [ ] **Preimenovanje paketa** `wattleflow-workflow-processors` → `wattleflow-processors` je
  provedeno u kodu; preostaje uskladiti dokumentaciju i PyPI/GitHub opise.
- [ ] **Preostali lazy-loading deferrali** (§2.7): `strategies/__init__` (documents strategije),
  `pipelines/__init__` (nlp).

### Refaktoring: migracija tip-hintova

Potvrđeno i u `wattleflow-processors` (73 datoteke s `Optional`/`Union`/`List`/`Dict`):

1. Uvesti `from __future__ import annotations` gdje nedostaje.
2. `pyupgrade --py311-plus` (ili `ruff check --select UP --fix`), **jedan commit po pod-paketu**.
3. Ručna provjera runtime-evaluiranih anotacija (Pydantic/dataclass/`get_type_hints`).
4. `ruff`/`mypy` gate u CI nakon migracije.

### Tika kao connection/driver + driver-kanal za create-strategije (nalaz 2026-06-26)

**Kontekst.** `EntityFileDocumentProcessor` bezuvjetno ekstrahira sadržaj preko Tike i
prosljeđuje `content=` u `blackboard.create`. Arhitektonski pogrešno: ekstrakcija je
odgovornost create-strategije, ne procesora.

**Odluka korisnika (2026-06-26): odgođeno.** Smjer (ne implementirati bez dogovora):

- [ ] **ConnectionTika + DriverTika** (OSCAL obavezan). Server-management (jar staging, Java
  provjera, timeout) seli iz `OCRTextProcessor` u `ConnectionTika`.
- [ ] **CreateTextDocument** koristi `DriverTika` za ekstrakciju iz `filename`;
  `CreatePdfDocument` (fitz) ostaje za PDF.
- [ ] **EntityFileDocumentProcessor** prestaje zvati Tiku; prosljeđuje samo `filename`.
- [ ] **Otvoreno pitanje — driver-kanal za create-strategije.** Create-strategije nemaju
  pristup driveru (write-strategije ga imaju preko `kwargs.get("driver")`). Smjer: isti
  mehanizam; izvedba dira `concrete/` (§2.5 — tražiti odluku).
- [ ] **Usputno:** `ReadDocumentFile` se referira u yaml-ima (03/04/05) ali **ne postoji**.

**Interim:** `tika_timeout` default 300s; suvišan Tika poziv u 04 ostaje do refaktora.

## Deduplikacija koda — DRY refaktor (nalaz 2026-06-27, ČEKA DR)

**Status:** analiza dovršena; implementacija **zaustavljena na zahtjev korisnika** dok se ne
izradi DR (arhitektura + NFR) koji vodi sustavni pristup. Ne dirati kod do tada.

> **D1 RIJEŠEN + reorg pipelinea (2026-07-06, direktiva korisnika — D2–D13 i dalje DR-gated).**
> `ocr_tokens` → **`helpers/ocr.py` `OcrText.tokens()`** (lazy pytesseract + `safe_open`).
> Razdvojena ekstrakcija od redukcije, native od generičkih:
>
> | Klasa | Modul | Uloga |
> |---|---|---|
> | `PipelineOCRExtract` (`OCRPreflightMixin`) | `pipelines/ocr/ocr.py` | tekst-ekstrakcija (Tika/tesseract, format-agnostično) |
> | `PipelineReductSpans` | `pipelines/entity/reduct.py` | samo redukcija — OCR bbox spanovi, image-only |
> | `PipelineMacroRedaction` | `pipelines/entity/macros.py` | text macro redakcija (`pii_hits`) |
> | `PipelinePDFExtractText` | `pipelines/pdf/pdf.py` | PDF tekst-ekstrakcija (native + OCR dopuna) |
> | `PipelinePDFRedactText` | `pipelines/pdf/pdf.py` | PDF redakcijski spanovi (fitz `search_for`) |
>
> `pipelines/png/` uklonjen. **Otvoreno:** pixel-apply write-strategija `WriteReductedPNG`
> nije implementirana — `PipelineReductSpans` piše spanove u metadata.
> *Napomena:* imena u ovoj tablici i worklist preimenovanja u `NFR-ORG-02` se razilaze —
> vidi „Usklađivanje dokumentacije".

| # | Klaster | ~Pojava | Lokacije (uzorak) |
|---|---|---|---|
| ~~D1~~ ✅ | `ocr_tokens` → `helpers/ocr.py OcrText.tokens()` | 2→0 | ex `pipelines/pdf/pdf.py`, `pipelines/png/reductions.py` |
| D2 | Tipovi (Tokens/RedactBoxes/…) | 3 modula | pdf/png pipelines + `strategies/documents/pdf.py` |
| D3 | Strategy `execute()` try/except→`StrategyException` | ~41 | sve `strategies/documents/*` |
| D4 | Driver resolution (3 varijante) | ~21 | kanon: `strategies/documents/pdf.py _resolve_driver` |
| D5 | `isinstance(caller/facade…)` preambule | ~38 | sve strategije (osim `pdf.py` → `Attribute.evaluate`) |
| D6 | Metadata pečat `created_by/at`,`stored_by/at` | ~21 | create/write strategije |
| D7 | Lazy import guard | ~34 | `connections/*`, `drivers/*`, `processors/youtube.py` |
| D8 | `raise DriverXxxError(caller=self, …)` wrapper | **112** | `drivers/*`, `connections/*` |
| D9 | Prazne `class DriverXError(DriverException): pass` | ~35 | kolizija `KafkaConnectionError` 2× |
| D10 | Record-document klase ~80% iste | 5 | `documents/{avro,orc,protobuf,solr,opensearch}.py` |
| D11 | Graph-document metode (8 verbatim) | 2 | `documents/graph.py`, `documents/youtube.py` |
| D12 | HTTP driver trio `_resolve_auth/_request/_json` | 3×3 | `drivers/{grafana,kibana,prometheus}.py` |
| D13 | FileScanner + source_path guard | 4 | `processors/{file,file_entity,tesseract,tika}.py` |

**Prijedlog — iskoristiti POSTOJEĆU infrastrukturu:** `FileType.detect/detect_content`,
`FormatterFactory`/`ParserFactory`, `helpers/converters` (Facade uzor), `Attribute.evaluate`,
`safe_open`, `Normaliser`/`TextMacros`, `decorators/preset.py` + `decorators/oscal/policy.py`,
`helpers/system.py`, `BaseWriteStrategy`.

- [~] **Helper/Facade:** **[x]** `helpers/ocr.py OcrText.tokens()` → **D1**;
  **[ ]** `helpers/system.py += require_module(name, pip=…)` → **D7**;
  **[ ]** `strategies/_support.py resolve_driver()` + `stamp_created/stored()` → **D4, D6**
- [ ] **Dekoratori:** `@strategy_guard` → **D3** (alternativa u core `GenericStrategy.call()`
  — §2.5, traži `DR-COR`); `@wrap_errors(DriverXxxError)` → **D8**
- [ ] **Mixini/baze:** `RecordDocument` → **D10**; `GraphDocumentMixin` → **D11**;
  `HttpJsonDriverMixin` → **D12**; `FileSourceMixin` → **D13**
- [ ] **Tipovi/konstante:** centralni `pipelines/types.py` → **D2**; `*_SUFFIX` iz `FileType`
- [ ] **D9:** ostaviti eksplicitne klase (greppabilnost), riješiti koliziju `KafkaConnectionError`

**Bugovi uočeni usput:** youtube write-strategija diže `PipelineException` (neuvezen) umjesto
`StrategyException`; `strategies/documents/text.py:149,195` `exec=e` umjesto `exc=e`;
`strategies/documents/solr.py:48-64` gradi `SolrDocument` 2×; `stored_at` nedosljedan
(`Now.utc()` vs `utc_time_stamp()`).

**Granice (§2.5):** `@strategy_guard` u coreu traži odluku korisnika; third-party helperi (OCR)
**moraju** u processors paket, lazy (§7.4). Redoslijed: (1) pipelines, (2) strategije, (3) driveri.

## NFR-ORG-01 / SEC-03 nalazi (wem_lint)

Rješenja **moraju poštovati zero-trust (§7.1)** i zahtijevaju strukturne izmjene. Stanje
vektora se ne prepisuje ovdje (§9, D-13) — pokreni `tools/wem_lint.py --snapshot` i referiraj
snimku u `documentation/workflow/conformance/`.

- [ ] **Ciklus i jednosmjerni proboj `concrete` ↔ `helpers`.** Shared helperi uzvodno uvoze
  jezgru (`config.py` → `concrete.exception`/`concrete.wattleflow`, `config_adapter.py` →
  `concrete.wattleflow`, `config_validator.py` → `concrete.workflow` uz povratni brid
  `concrete/workflow.py` → `helpers`), dok concrete uvozi `helpers.*`. Smjer (TBD, dira
  `concrete/` → §2.5): zajedničke iznimke/konstante ispod helpersa ili lokalne iznimke u
  helpersima; cilj acikličnost shared sloja. **Postalo mjerljivo tek s `wem_lint 1.12.0`**
  — prije je scope filtar brisao module prije nego ih pravilo vidi.
- [ ] **Odluka o razini dok se ciklus ne razriješi.** `domain_acyclicity` je `error`, pa ta
  četiri nalaza ruše build. Ostaviti tako, spustiti na `warning` ili dati deklarirani waiver —
  izmjena `rules` bloka u kriteriju, dakle **kroz DR** (D-03).
- [ ] **Jedno-potrošački dijeljeni helperi.** Prema ORG-01 kriteriju 1 pripadaju unutar domene
  koja ih koristi ili u njezin domain-internal shared modul; tolerirani su dok se ne presele.
- [ ] **§7.1 procurivanje third-partyja u jezgru:** `mappers/schema_yaml_json.py` →
  pandas/yaml/jsonschema (kandidat za `wattleflow-processors`). `concrete/logger.py`→pandas je
  **riješen 2026-07-22** (duck-typing `hasattr(shape, "columns")`).
- [ ] **18 processors-modula u workflow stablu** (converters/, formatters/, parsers/, protobuf,
  image_guard, generators, localmodels) — kandidati za seljenje (vezano uz „Razdvajanje
  specijalizacija"); `scope` filtar ih za sad isključuje iz opsega.

## Verzijska kontrola i objava dokumentacije (nalaz 2026-08-03)

Praćenje je riješeno (§8): `documentation` v0.0.5 prati cijeli sadržaj, push na `origin` je
onemogućen. Preostaje ono što praćenje ne rješava:

- [ ] **Hrvatski doktrinarni tekst je već javan na GitHubu.** `.gitignore` je štitio samo
  `*/hr/*`, pa su `DOCTRINE.md`, `POSTULATE.md`, `dictionary.yaml`, cijela `workflow/dr/`
  serija, `workflow/analysis/` i `core/DR.md` objavljeni na
  `github.com/wattleflow/documentation` (grana `default`, do `v0.0.4`). Odluka: prihvatiti
  zatečeno stanje, ili povući repozitorij / prepisati povijest prije v1.0.
- [ ] **Put objave za UK English izdanje.** Kad `workflow/en/**` postoji, treba mehanizam koji
  objavljuje **samo** njega — kurirana javna grana ili zaseban javni repozitorij. Do tada je
  push namjerno zaključan, pa objava ne može nastati slučajno.
- [ ] **`documentation/MANIFEST.in` je mrtav.** U tom repozitoriju nema `pyproject.toml` ni
  `setup.py`, pa se ništa ne pakira; datoteka je uz to proturječna (`recursive-include workflow
  *.md` pa `prune workflow`) i sugerira zaštitu koje nema. Obrisati ili opravdati.
- [ ] **Neprefiksirane DR oznake u `workflow/hr/dr/`** — `DR-007-iterator.md`,
  `DR-ORG-05-observable.md`. Sada su praćene, pa se vidi da krše §8 („Neprefiksirana oznaka
  nije valjana"); dodati banner ili preimenovati.

## Otvorena pitanja za razgovor

- [ ] Planirani OSCAL katalozi izvan ASD ISM (NIST 800-53, ISO 27001, CIS)?
- [ ] Format metrika za dashboarde (Prometheus, OpenTelemetry, statsd)?
- [ ] Test framework (pytest, unittest, drugo) — odluka prije dokumentacijske faze 4
- [ ] Jedinstveni layout dokumentacije: koji registri ostaju u korijenu, a koji u `workflow/hr/`
