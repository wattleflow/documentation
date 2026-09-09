# Metoda: mjerenje kvalitete podataka (DQI)

| | |
|---|---|
| **Sloj** | metoda (`METHODOLOGY.md` §2 — anatomija); nadređeno: `PHILOSOPHY.md`, `DOCTRINE.md` |
| **Oznaka** | dodjeljuje se s registrom metoda (otvoreno — vidi *Stanje*) |
| **Implementacija** | `wattleflow-processors`, `pipelines/quality/dqi.py` |
| **Standardi** | ISO/IEC 25012 (model dimenzija), ISO 8000-8 (mjera kao omjer), ISO 8000-61 (procesni model) |

Mjerenje je **opservacijsko**: ne mijenja podatke i ne zaustavlja tok. Rezultat putuje kao
metapodatak uz zapis do odredišta.

---

## 1. Anatomija metode

Po skeletu iz `METHODOLOGY.md` §2.

**Znanstveno pitanje.** Kolika je kvaliteta podataka i kakav je neto učinak cjevovoda na
nju? Prije mjerenja kvaliteta je informacijska praznina.

**Hipoteza — H4-DQI.** *Vektor dimenzija kvalitete predviđa operativne troškove kvalitete
podataka* (ručne ispravke, incidenti, downstream odbijanja). Obara je izostanak povezanosti
s vanjskim kriterijem na uzorku različitom od kalibracijskog.

> Hipoteza je citirana kao doktrinarna, ali `PHILOSOPHY.md` registrira samo H1–H3. Dok se
> H4 ne upiše u registar hipoteza, ovdje je vodi se kao **kandidat** (D-05).

**Kontrolne točke.** `T1` sirovi ulaz (inherentna kvaliteta izvora), `T2…` međurezultati
(može ih biti više: `T2a`, `T2b`), `T3` finalni podaci prije transfera (kvaliteta
proizvoda). Razlika `d(T3) − d(T1)` po dimenziji daje **neto učinak** cjevovoda; omjer
`|T3|/|T1|` daje **stopu zadržavanja** zapisa.

**Mjera.** Primaran rezultat je **vektor dimenzija** — potpunost, valjanost, dosljednost,
točnost, aktualnost, jedinstvenost — svaka u `[0, 1]` kao omjer zapisa koji zadovoljavaju
pravilo (omjerna skala **po dimenziji**).

`DQI = Σ wᵢ·dᵢ` je **deklarirani zbirni indeks**, konforman a ne deskriptivan: težine su
izbor, ne empirijska činjenica. Promijeni težine — promijeni se broj. Svaki iskaz DQI-ja
zato nosi deklaraciju težina i verziju konfiguracije. Agregacija prosjeka i varijance ide
Welfordovim on-line algoritmom (O(1) memorije, numerički stabilan, Besselova korekcija).

**Valjanost.** Status po protokolu iz §2 ovog dokumenta. Do prolaska V1–V4 DQI se iskazuje
kao *konformni indeks s deklariranim težinama*, nikad kao mjera kvalitete po sebi.

**Prosudba.** `failed_rules` (koja pravila, gdje), usporedba po kontrolnim točkama,
interpretacija neto učinka i zadržavanja.

**Ishod i sljedivost.** Rezultat putuje kao metapodatak uz zapis
(`facade.document.update_metadata`). Modul je agnostičan na standard: promjenom
konfiguracije isti cjevovod mjeri prema različitim okvirima; nove dimenzije su plug-in
funkcije.

---

## 2. Protokol valjanosti V1–V6 primijenjen na DQI

Generički protokol definira `METHODOLOGY.md` §5.1; ovdje je njegov status **za ovu mjeru**.

| Faza | Postupak za DQI | Status |
|---|---|---|
| **V1** reprezentacijski uvjet | Za svaku dimenziju definirati uređaj „skup A kvalitetniji od B po dimenziji d" (ekspertna prosudba na parovima uzoraka) i provjeriti da mjera čuva uređaj (homomorfizam). | *otvoreno* |
| **V2** tip skale | Dimenzije su omjeri zapisa → omjerna skala po dimenziji; zbroj preko dimenzija nije mjerenje nego indeks. Dopušteno: usporedba po dimenziji, razlika po kontrolnim točkama, vektorska/Pareto usporedba. | *deklarirano* |
| **V3** sadržajna pokrivenost | Preslikavanje pravilo → dimenzija (ISO 25012). Dimenzija bez ijednog pravila iskazuje se kao **nemjerena**, nikad kao `1.0`. | *provedivo odmah* |
| **V4** prediktivna valjanost | Povezanost s vanjskim kriterijem (trošak ispravaka, incidenti, odbijanja) na uzorku različitom od kalibracijskog. Ovo je test hipoteze H4-DQI. | *otvoreno* |
| **V5** osjetljivost | Perturbacija težina (±); stabilnost po kontrolnim točkama i granulaciji. Gdje poredak alternativa ovisi o težinama, odluka se vraća na vektor i Pareto analizu. | *provedivo odmah* |
| **V6** erozija pod upravljanjem | Indeks je **isključivo dijagnostički**; quality gate smije stajati samo na binarnim pravilima (`failed_rules`), nikad na indeksu — inače se optimizira broj, ne kvaliteta. | *propisano* |

---

## 3. Kako se koristi u frameworku

Mjerenje se ne ugrađuje u transformaciju nego se **dodaje kao kontrolna točka** u cjevovod:

1. Odredi kontrolne točke. Minimalno `T1` i `T3` — bez obje nema neto učinka, samo apsolutni
   broj koji ništa ne govori o tome što je cjevovod napravio.
2. Deklariraj pravila po dimenziji. Dimenzija bez pravila ostaje **nemjerena** (V3); ne
   popunjava se jedinicom.
3. Deklariraj težine i verziju konfiguracije ako se iskazuje zbirni `DQI`. Bez toga se
   iskazuje samo vektor.
4. Rezultat se pridružuje zapisu kao metapodatak i putuje s njim; agregat po kontrolnoj
   točki drži `RunningAggregate`.

**Granica upotrebe (V6).** Gate na `failed_rules` — da. Gate na `dqi` vrijednosti — ne.
Prag na indeksu pretvara mjeru u cilj i optimizira se broj, ne kvaliteta.

---

## 4. Implementacija

**Zašto u `wattleflow-processors`, a ne u jezgri.** Mjerenje kvalitete radi nad podatkovnim
strukturama i vuče third-party tier, pa po `CLAUDE.md` §7.1 (matična distribucija = import
closure) ne pripada clean core distribuciji. Workflow koji ga koristi povlači `processors`
kao ovisnost; sam cjevovod ostaje deklarativan.

**Nosivi tipovi.**

| Tip | Uloga |
|---|---|
| `DataQualityIndex` | izmjereni indeks za **jedan zapis** u jednoj kontrolnoj točki: `dqi`, `dimensions`, `failed_rules`, `checkpoint`, `record_id`, `timestamp`; `to_dict()` za metapodatak |
| `RunningAggregate` | on-line agregat (prosjek, varijanca, min, max) preko zapisa; Welford |
| `compute_*` funkcije | jedna po dimenziji; potpisom vezane na pravila, plug-in |

---

## 5. Stanje (D-05)

Sljedeće se vodi kao **aspiracija ili nalaz**, ne kao provedeno:

- **Implementirana je jedna dimenzija** — `compute_completeness`. Preostalih pet (valjanost,
  dosljednost, točnost, aktualnost, jedinstvenost) opisano je, ne izvedeno. Vektor je zato
  danas jednodimenzionalan, a zbirni `DQI` nema što ponderirati.
- **Modul se trenutno ne uvozi:** referira `wattleflow.helpers.datetime`, a taj je modul u
  workflow distribuciji preimenovan. Cross-distribucijski nalaz — popravak ide u
  `processors`, prijavljeno u `workflow/TODO.md`.
- **`__all__` nije deklariran**, što traži `CLAUDE.md` §2.7 t.1.
- **H4-DQI nije u registru hipoteza** (`PHILOSOPHY.md` ima H1–H3).
- **Oznaka metode** čeka registar metoda; do tada se dokument referira putanjom.
