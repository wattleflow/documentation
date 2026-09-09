# DR-WFL-006 — Pin na core ovisnost; verzioniranje ostaje ručno

| | |
|---|---|
| **Status** | **Prihvaćen** (2026-07-28) |
| **Datum** | 2026-07-28 |
| **Verzija** | 1 (2026-07-28) |
| **Realizira** | [DR-WFL-002](DR-WFL-002-distribution-locality.md) (lokalnost distribucije), `DR-COR-013` (politika verzija Pythona) |
| **Mijenja** | `pyproject.toml`, `MANIFEST.in`, `requirements-dev.txt` |

## Kontekst

Tri nalaza iz analize prijelaza core v0.0.0.38 → v0.0.0.46:

1. **`dependencies = ["wattleflow"]` bez donje granice.** Core v0.0.0.46 pretvorio je
   `IWattleflow` u ABC s apstraktnim `name`, uklonio `ISingleton` i `T`/`W` TypeVarove
   i preimenovao `behavioral.py` → `behavioural.py`. Bilo koja niža verzija pada pri
   uvozu. Nepinovana ovisnost znači da je pip smio instalirati kombinaciju koja ne radi.
2. **sdist bez ijedne `.py` datoteke.** `MANIFEST.in` je izgubio `recursive-include`
   retke; kako se wheel gradi iz sdista, distribucija v0.0.0.86 bila je prazna.
   Uz to, četiri paketa (`audit`, `managers`, `orchestrators`, `schedulers`) deklarirana
   su u `packages.find` a nikad slana — instalirali su se prazni.
3. **`[tool.setuptools_scm]` inertan.** Statični `[project].version` uvijek pobjeđuje,
   pa se tag nikad nije konzultirao. Nije bilo bezazleno: git nosi `v0.0.0.85`,
   `v0.0.0.86` i `v0.0.0.87` na **istom commitu** (`b26c5d0`), a
   `concrete/_version.py` je zaostao kao artefakt tog bloka čitajući `0.0.0.31.dev0`.

## Odluka

**(a) Pin donje granice:** `dependencies = ["wattleflow>=0.0.0.46"]`.

Gornja granica **nije** postavljena. Za nju treba politika verzija (`DR-COR-013`) da
kaže što je prekidajuće izdanje u `0.0.0.x` shemi — trenutno je svaki bump vizualno
zakrpa, a v0.0.0.46 dokazuje da to ne mora biti.

**(b) `MANIFEST.in` je jedina kontrolna točka pakiranja**, uz eksplicitni invariant:
*svaki paket koji `packages.find` deklarira mora ovdje biti slan.* Paket deklariran a
neposlan instalira se prazan i pada pri uvozu, ne pri instalaciji.

Zabilježeno je i **zašto** je MANIFEST autoritativan: `global-exclude *` u prvom retku
odbacuje zadani popis datoteka (uključujući git-tracked skup). Brisanje tog retka tiho
vraća pakiranje na „što god git prati" i zero-trust granica prestaje biti provedena.

**(c) Verzioniranje ostaje ručno.** `[tool.setuptools_scm]` i pripadni build-requirement
se uklanjaju, `concrete/_version.py` se briše — isto što je core napravio u v0.0.0.35.
Prijelaz na tagove ostaje moguć (`dynamic = ["version"]`), ali **tek nakon što se tagovi
srede**: tri oznake na jednom commitu učinile bi izvedenu verziju nedeterminističnom.

## Ugovor

* Instalacija uz core < 0.0.0.46 sada pada pri **resolveu**, ne pri uvozu — pomak
  greške ulijevo, namjeran.
* `wattleflow.schedulers`, `.audit`, `.managers`, `.orchestrators` postaju stvarno
  dostupni iz wheela. Nijedan potrošač nije mogao ovisiti o njima ranije (bili su
  prazni), pa nema loma unatrag.
* `_version.py` više ne postoji; ništa ga nije uvozilo (provjereno).

## Cijena

Runtime nula. Distribucija raste za 4 paketa (3 su prazna sidra „build once, use
often"; `schedulers` nosi `CronJobScheduler`). Suprotno NFRQ-SEC-02 (minimalna napadna
površina) samo prividno — ti su paketi već bili deklarirani, samo neisporučeni.

## Svjedočanstvo

Empirijski, mjereno buildom 2026-07-28:

| | prije | poslije |
|---|---|---|
| `.py` u wheelu | **0** | 64 |
| paketa u wheelu | 0 | 11 |
| `_version.py` u distribuciji | da (0.0.0.31.dev0) | ne |
| `core/` u distribuciji | ne | ne |
| build warninga | 5 | 0 |

Test instalacije iz wheela: svi paketi uvozivi, `AuditLogger(level='NOTSET').name` →
`'AuditLogger'`.

Clean-core mjerenje: **62 od 65 modula stdlib-only**; sva tri izuzetka
(`helpers/config.py`, `config_adapter.py`, `config_validator.py`) nose guard iz
[DR-WFL-003](DR-WFL-003-guarded-optional-dependency.md).

## Registar

`pyproject.toml`:
* `dependencies` → `wattleflow>=0.0.0.46`
* `[tool.ruff] target-version` `py310` → `py311` (nije se slagalo s `requires-python`)
* uklonjeni `[tool.setuptools_scm]`, `[tool.setuptools.package-data]` i
  `setuptools_scm` iz `build-system.requires`

`MANIFEST.in`: +4 `recursive-include`, `prune docs/documentation`, invariant blok.

`requirements-dev.txt`: `setuptools_scm` uklonjen; `setuptools>=77` po core stilu.

## Otvoreno

Tri tag oznake (`v0.0.0.85/86/87`) stoje na commitu `b26c5d0`. Dok se ne razriješi,
prijelaz na `dynamic = ["version"]` nije moguć.

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-28 | prvi zapis |
