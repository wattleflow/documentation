# DR-WFL-003 — Čuvana opcionalna ovisnost sa stdlib fallbackom (iznimka na ORG-06 §2.1)

| | |
|---|---|
| **Status** | **Aktivan** (Prihvaćeno 2026-07-15) — mijenja `NFR-SEC-03` kriterij 1 |
| **Datum** | 2026-07-15 |
| **Verzija** | 2 (2026-07-28) — v1: 2026-07-15 |
| **Realizira** | §7 Zero-trust (packaging), uz očuvanu funkcionalnost core konfiguracije |
| **Mijenja** | `DR-WFL-002` §2.1 (Lokalnost distribucije), `NFR-SEC-03` kriterij 1 (tada `NFR-ORG-06`) |
| **Kontekst rada** | Dovršetak decouplinga `wattleflow-workflow` corea od third-partyja (2026-07-15) |
| **Sljedivost** | `PHILOSOPHY.md` (Zero-trust, Kritičko razmišljanje) · `METHODOLOGY.md` §1, §6, §8 · `POLICY.md` §7 · [`NFR-SEC-03`](../../03-NFRQ/NFR-SEC-03-supply-chain-locality-EN.md) |

---

## 1. Kontekst (problem / prilika)

Dovršavanjem migracije third-party modula iz `wattleflow-workflow` corea u
`wattleflow-processors` (2026-07-15) core je sveden na **stdlib + `wattleflow`**. Empirijska
provjera — maskiranje *svih* third-party paketa i uvoz svakog modula — pokazala je da se
32 od 33 modula `helpers/` i `decorators/` uvozi čisto. (Jedini pad, `helpers/memory.py`,
nije ovisnosni nego zatečeni bug: `from abc import staticmethod`.)

Ostao je jedan klaster koji ne pada ni u jednu postojeću kategoriju:
`helpers/config.py`, `helpers/config_adapter.py` i `helpers/config_validator.py` referiraju
`yaml` i `jsonschema`, ali **iza `try/except` s potpunim stdlib fallbackom**:

```python
try:
    import yaml
    from jsonschema import validate
except Exception:
    from wattleflow.helpers.yaml import yaml, validate
```

`helpers/yaml.py` je vlastiti stdlib shim (mini-parser + `validate` stub) — nije stub koji
podiže iznimku, nego funkcionalna zamjena. Modul je stoga uvozom i radom **potpun bez
third-partyja**; PyYAML ga samo ubrzava i proširuje.

`DR-WFL-002` §2.1 propisuje bezuvjetno: *„Čim modul — eager **ili** lazy — referira
third-party paket, pripada ne-core distribuciji"*. Doslovna primjena tražila bi seljenje
sva tri `config*` modula u processors. Cijena je nesrazmjerna: `concrete/workflow.py`
gubi `ConfigAdapter`, pa clean core ostaje **bez YAML konfiguracije** — a to je nosivi
mehanizam frameworka, ne specijalizacija.

Napetost je stvarna i nije stilska: slovo pravila i njegova svrha (zero-trust) ovdje se
razilaze. Modul s čuvanim fallbackom **ne izlaže** korisnika kompromitiranoj biblioteci
ako ta biblioteka nije instalirana — što je upravo ono što §7.1 štiti.

## 2. Odluka

### 2.1 Kategorija: čuvana opcionalna ovisnost (guarded optional dependency)

Uvodi se **treća kategorija** referenci na third-party, između „nema reference" i
„referira → ne-core":

> Modul ostaje u clean core distribuciji unatoč third-party referenci **ako i samo ako**
> je referenca *čuvana* — zaštićena `try/except ImportError` granom čiji je fallback
> **funkcionalno potpun** i unutar dopuštenog tiera te distribucije (stdlib + core
> allowlist).

Ključni kriterij je **potpunost fallbacka**, ne postojanje `try/except`-a. `try/except`
koji u `except` grani diže `ModuleNotFoundError`, vraća `None` ili degradira funkciju na
neupotrebljivu **nije** čuvana ovisnost — to je lazy referenca i ORG-06 §2.1 vrijedi
neizmijenjen.

### 2.2 Razgraničenje prema §2.1 (što se NE mijenja)

`DR-WFL-002` §2.1 ostaje na snazi za sve ostalo. Konkretno, **lazy import bez fallbacka i
dalje fiksira matičnu distribuciju**. Ista migracija (2026-07-15) po tom pravilu je iz
corea iselila:

| Modul | Third-party | Zašto nije čuvana ovisnost |
|---|---|---|
| 4 cloud secret resolvera → `helpers/cloud/cloud_secrets.py` | `boto3`, `azure-*`, `google-cloud-*`, `hvac` | `except: return None` — nema fallbacka, funkcija tiho otkazuje |
| `records()` → `helpers/random_data.py` | `numpy` | lazy import bez ikakve alternative |

Razlika je epistemički provjerljiva, ne stvar prosudbe: **maskiraj paket i uvezi modul.**
Čuvana ovisnost radi; lazy referenca bez fallbacka ne radi ili tiho laže.

### 2.3 Obveza dokumentiranja i verifikacije

* Svaka čuvana ovisnost nosi `# NOTE` uz `try/except` s razlogom i imenom fallback modula.
* Fallback mora biti **verificiran testom maskiranja**, ne pretpostavljen. Test je
  ponovljiv (`METHODOLOGY.md` §1 t.2) i ulazi u lint/CI kad pipeline postoji.
* `scope.core_libraries` u kriteriju koda (`tools/dictionary.json`; tada `naming_registry.yaml`) **ne** dobiva `yaml`/`jsonschema`:
  oni nisu core biblioteke nego opcionalno ubrzanje. Allowlist ostaje minimalan.

## 3. Načela i sljedivost (`METHODOLOGY.md` §6)

| Načelo | Implikacija |
|---|---|
| Zero-trust (§7.1) | svrha je da kompromitirana biblioteka ne dopre do core korisnika; čuvani fallback to jamči jer core radi i bez nje |
| Occamova britva | seljenje `config*` u processors rješava formalni prekršaj po cijenu gubitka nosivog mehanizma — složenije rješenje za isti sigurnosni ishod |
| Information Hiding (Parnas) | `try/except` skriva izbor implementacije parsera iza jednog imena (`yaml`); potrošač ne zna koja grana je aktivna |
| Open–Closed | dodavanje PyYAML mijenja performanse, ne ugovor |
| Kritičko razmišljanje (`PHILOSOPHY.md`) | pravilo se ne primjenjuje „po inerciji" — sukob slova i svrhe razrješava se **dokumentiranom revizijom**, ne tihim odstupanjem |

Doktrinarno, ovaj DR je primjer pravila *„niži sloj ne nadjačava viši; svako odstupanje je
izvod iz višeg sloja ili dokumentirana revizija tog sloja"*. Metoda (ORG-06 §2.1) revidirana
je jer ju je viši sloj (policy zero-trusta) nadjačao u točki gdje su se razišli — i to je
zabilježeno, pa je odluka provjerljiva i opoziva.

## 4. Posljedice

**Pozitivne**

* Clean core zadržava YAML konfiguraciju — `concrete/workflow.py` ostaje netaknut.
* Kriterij postaje **empirijski provjerljiv** (test maskiranja) umjesto sintaktičkog
  („postoji li import").
* Formalizira se obrazac koji je u kodu ionako zatečen, umjesto da ga se tolerira prešutno.

**Negativne / rizici**

* **Shim mora ostati funkcionalno dostatan.** Ako `helpers/yaml.py` zaostane za zatečenim
  YAML-ima, fallback tiho daje drukčiji rezultat od PyYAML-a — to je gori kvar od
  `ImportError`-a. Ovo je glavni preostali rizik ove odluke.
* **Dvije implementacije istog ugovora** = dvostruki teret održavanja i mogućnost
  divergencije semantike (DRY napetost, svjesno prihvaćena).
* Kategorija se može zloupotrijebiti — „dodat ću `try/except` da prođe lint". Zato §2.1
  traži *potpun* fallback, a §2.3 test, ne deklaraciju.

## 5. Status i otvorena pitanja

* [ ] **Diferencijalni test shima vs PyYAML** — ista YAML datoteka kroz obje grane mora
  dati isti objekt. Bez toga rizik iz §4 nije pokriven. **Prioritet.**
* [ ] **Automatizacija u `wem_lint`** — kriterij traži razlikovanje čuvane reference od
  lazy reference (AST: je li `import X` unutar `try` s `except ImportError` granom koja
  uvozi core fallback). Do tada je ORG-06 kriterij 1 pregledom-vođen za ovaj slučaj.
* [ ] **Opseg kategorije** — vrijedi li i za `cryptography` ako se vrati u core? Trenutno
  neriješeno; `cryptography` je izbačen iz requirementsa, a `scope.core_libraries` ga još
  navodi (zaostatak za pregled).

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-15 | prvi zapis, oznaka `ADR-ORG-07` / `DR-ORG-07` |
| 2 | 2026-07-28 | oznaka → `DR-WFL-003`; sadržaj odluke nepromijenjen |
