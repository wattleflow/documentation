# DR-WFL-002 — Lokalnost distribucije, pakiranje i validacija opskrbnog lanca

| | |
|---|---|
| **Status** | **Aktivan** (Prihvaćeno 2026-07-09, uz manje izmjene) — podložno reviziji do v1.0; predlaže **`NFR-SEC-03`** |
| **Datum** | 2026-07-09 |
| **Verzija** | 2 (2026-07-28) — v1: 2026-07-09 |
| **Realizira** | §7 Zero-trust (packaging), NFR-ORG-01 (Dependency Locality) preko granice distribucije |
| **Predlaže** | `NFR-SEC-03` — lokalnost distribucije i supply-chain (zapisan 2026-07-09 kao `NFR-ORG-06`, povučen i dekomponiran 2026-07-22) |
| **Kontekst rada** | Prijelaz s monolitnog razvoja na više distribucija (`wattleflow`, `wattleflow-workflow`, `wattleflow-processors`, `wattleflow-cad`); povod: `helpers/converters/` (python-docx/PyMuPDF) gitignoran u workflow stablu |
| **Sljedivost** | `PHILOSOPHY.md` (Zero-trust, Samoopisivost) · `METHODOLOGY.md` §5, §9 · `CLAUDE.md` §7, §2.6/§2.7 · `03-NFRQ/` NFR-ORG-01 |

---

## 1. Kontekst (problem / prilika)

Dosad je wattleflow razvijan **monolitno**: jedno stablo, dev-symlinkovi dijele
`core`/`concrete`/`helpers`/`constants` između projekata. Prelaskom na više
distribucija (zero-trust razdvajanje, §7; `wattleflow-processors` i `wattleflow-cad` već izdvojen) 
pojavljuju se tri isprepletena problema koja monolit nije imao:

1. **Kome pripada modul** kad ga dijeli više distribucija, a distribucije nisu
   ravnopravne (clean core = zero-trust vs processors/cad = third-party dopušten)?
   Konkretno: `helpers/converters/` ovisi o python-docx/PyMuPDF, a fizički živi u
   workflow stablu — gitignore ga skriva umjesto da ga preseli. Isti obrazac prijeti
   svemu što dijeli od poput: helpers, converters, decorators, factories/builders/
   creators, pa i dijelovi `constants`.
2. **Kako pripremiti deployment** tako da svaka distribucija nosi **samo svoje**
   biblioteke, a ne procuri third-party u clean core (supply-chain napadna površina).
3. **Kako se razvijati** kad se napusti symlink model — hoće li raditi kombinacija
   „jedan paket u `site-packages`, drugi u lokalnom projektnom stablu"?

Povod baziran na prakticnom ne teoretskom pristupu: 4 od 14 `@staticmethod→@classmethod` izmjena (2026-07-09) sletjele
su u `helpers/converters/` koji je git-ignoran i ne pripada nijednoj distribuciji — dokaz
da bez formalnog pravila datoteke završavaju u krivoj distribuciji „po inerciji".

## 2. Odluka

### 2.1 Lokalnost distribucije = najteža ovisnost, ne uloga

**Matična distribucija modula određena je njegovim import-closureom, ne njegovom
ulogom.** Modul pripada clean core (`wattleflow`/`wattleflow-workflow`) **samo ako**
mu cijeli tranzitivni closure staje u dopušteni tier te distribucije. Čim modul —
eager **ili** lazy — referira third-party paket, pripada ne-core distribuciji
(`wattleflow-processors`/`wattleflow-cad` etc.), jer je i sam kod (ne samo pip-ovisnost)
dio napadne površine (§7.1). Lazy-loading (§7.4) ublažava *import-time* trošak, ali **ne**
mijenja matičnu distribuciju.

> **Izmijenjeno `DR-WFL-003` (2026-07-15) — čuvana opcionalna ovisnost.** Gornje pravilo
> vrijedi za eager i lazy reference **bez fallbacka**. Modul čija je third-party referenca
> *čuvana* — `try/except ImportError` s **funkcionalno potpunim** fallbackom unutar tiera
> distribucije — ostaje u clean coreu, jer mu je efektivni closure stdlib (primjer:
> `helpers/config*.py` → `yaml`/`jsonschema` uz `helpers/yaml.py` shim). Kriterij je
> potpunost fallbacka, provjerena **testom maskiranja**, a ne postojanje `try/except`-a.

### 2.2 Tier po distribuciji + manifest

| Distribucija | Dopušteni tier | Zero-trust |
|---|---|---|
| `wattleflow` (core) | stdlib | da |
| `wattleflow-workflow` | stdlib (+ eksplicitni core allowlist) | da |
| `wattleflow-processors` | stdlib + third-party (lazy, §7.4) | **ne** (namjerno) |
| `wattleflow-cad` | stdlib + third-party (CAD-specifičan) | **ne** (namjerno) |

Svaka distribucija deklarira **distribution manifest** — koja `wattleflow.*` podstabla
posjeduje (već postoji kao `[tool.setuptools.packages.find] include` u `wattleflow-cad`).
Manifest je izvor istine za pakiranje i za lint gate (2.4a).

### 2.3 Symlinkovi se povlače → PEP 420 namespace + PEP 660 editable

Dev-symlinkovi se **ukidaju**. Ciljni model **već postoji**: sva četiri projekta nemaju
`wattleflow/__init__.py` (PEP 420 implicitni namespace), a `wattleflow-cad` već pakira
samo svoja podstabla (`namespaces=true` + `include` allowlist, tretira
`wattleflow.drivers/pipelines/strategies` kao namespace-roditelje koje ne posjeduje).

- **Pakiranje:** generalizirati cad obrazac na sve distribucije — svaka posjeduje točno
  svoja podstabla; nijedna ne pakira tuđi `__init__.py`.
- **Razvoj:** `pip install -e` (PEP 660) svake distribucije koja se razvija →
  `site-packages` dobiva finder koji pokazuje **natrag na lokalni izvor**, pa se oba
  puta razrješavaju na lokalni kod (rješava korisnikovu bojazan „jedan u site-packages,
  drugi lokalno"). Distribucije koje se **ne** razvijaju instaliraju se normalno; PEP 420
  ih spoji u isti `wattleflow.*` namespace. Uvjet: **svako podstablo posjeduje točno
  jedna distribucija** (cad to već poštuje). Alat: `uv`/`pip` workspace za dev bootstrap.

### 2.4 Dvije odvojene validacije (ne miješati)

**(a) Locality gate — statička, build-time, *naše* pravilo.** „Je li datoteka u ispravnoj
distribuciji?" `wem_lint` već računa tier (`core_libraries` allowlist → out-of-scope za
third-party); nadograđuje se da **ruši build** kad modul u distribuciji krši njezin tier
ili nije u njezinu manifestu. Jeftino, nadovezuje se na postojeći lint.

**(b) Supply-chain integritet — third-party, install/deploy-time.** „Je li instalirana
biblioteka ona koju smo verificirali?" **Ne re-implementirati vlastiti digest-registar** —
usvojiti standarde koji se izravno vežu na postojeći OSCAL sloj:
- **hash-pinned lockfile** (`pip --require-hashes`, `uv.lock`) — prikovane verzije;
- **SBOM** (CycloneDX/SPDX) — standardizirana „evidencija digest vrijednosti";
- **atestacije/potpisi** (Sigstore, PEP 740; PEP 458/480 TUF za PyPI).

`wattleflow` core je **nositelj politike** (dopušteni paketi/licence/known-bad) i drži
**validator koji čita SBOM** ciljne distribucije te javlja prekršaje — ne kopiju
integritetskog mehanizma. OSCAL referira SBOM kao assessment evidence.

**Razgraničenje:** `FileDigest` (`helpers/digest.py`, već postoji) služi integritetu
**vlastitih** wattleflow modula (je li naš kod diran), **ne** vettingu third-partyja.

**Profinjenje (2026-07-12).** Ni self-integritet vlastitih modula ne treba vlastiti format:
izgrađeni wheel već nosi `RECORD` (per-file `sha256`) — **verificira se on**, ne paralelni
registar (izbjegava drift; Occam/DRY). Ključno ograničenje otkriveno u praksi: `RECORD`
**editable** installa sadrži samo `.pth`, pa je u dev stablu **prazan**. Ondje `wem_lint`
digest-scan (`FileDigest`) pokriva self-integritet i usput detektira namespace-sjenčanje
(dvostruko vlasništvo, §4) — točno klasa buga koju standardni alati ne vide. „Distribucijski
digest" (hash `RECORD`-a / SBOM korijen) stabilan je samo za izgrađene artefakte. Formalizirano
kao **NFR-ORG-06 kriterij 5**. (Primjer epistemičkog rasta iz `PHILOSOPHY.md` „Doktrina".)

### 2.5 Raspetljavanje zamršenih helpera (ports/adapters)

Helperi koji su *dijeljeni ali teški* (npr. `concrete/logger.py`→pandas,
`mappers/schema_yaml_json.py`→pandas/yaml/jsonschema, `helpers/converters/`→docx/fitz)
razdvajaju se po ovisnosti: **čisto sučelje/baza ostaje u core, third-party adapter seli
u pripadajuci projekt.** Gdje je apstrakcija stvarno dijeljena, **duplicirati tanku apstrakciju**
umjesto curiti ovisnost u core. Odluka je **per-helper** (ADR-vrijedna), ne mehanička;
`wem_lint` nalazi (2026-06-29: 2 procurivanja) su početni worklist koja moraju biti otklonjenja do verzije 1.0.


## 3. Načela i sljedivost (`METHODOLOGY.md` §5)

| Načelo | Primjena |
|---|---|
| Zero-trust (§7.1) | matična distribucija = najteža ovisnost; `core` nosi **nula third-party** koda |
| Dependency Inversion / Stable Abstractions | čista baza u core, konkretni adapter u processors |
| Information Hiding (Parnas) | third-party specifičnosti skrivene iza core apstrakcije |
| SRP / Separation of Concerns | locality (naše pravilo) ⟂ supply-chain integritet (standardi) |
| Open–Closed | nova distribucija = novi manifest, bez diranja postojećih |
| Occam / DRY | ne izmišljati integritet — SBOM/lock/atestacije već postoje |
| NFR-ORG-01 | isti princip lokalnosti, proširen s intra-tree na inter-distribution |

## 4. Posljedice

**Pozitivno**
- Clean core (`wattleflow`/`wattleflow-workflow`) postaje uistinu zero-trust — nula
  third-party koda, ne samo nula eager importa.
- Deployment po distribuciji nosi samo svoje biblioteke; napadna površina minimalna i
  eksplicitna po komponenti.
- Symlink-uzrokovani problemi (lažno vlasništvo, git-tracking kaos, dvostruko brojanje)
  nestaju; dev ostaje live preko editable installova.
- Model **već dokazan** (cad); generalizacija je konfiguracija + migracija, ne izum.

**Negativno / rizik**
- **Migracijski trošak:** preseljenje `converters/`, `concrete/logger.py`,
  `mappers/*` i sličnih iz workflow u processors; per-helper ports/adapters odluke.
- **Dev bootstrap kompleksniji:** umjesto „kloniraj i radi", potreban je editable-install
  korak (`uv sync`/`pip install -e ...`) i red instalacije koji poštuje namespace vlasništvo.
- **Rizik dvostrukog vlasništva:** ako dvije distribucije pakiraju isto `wattleflow.*`
  podstablo, namespace merge puca (import shadowing). Manifest + lint gate to sprječavaju.
- **Prijelazni period:** dok migracija traje, dio koda je u „krivoj" distribuciji;
  gate se uvodi kao WARN pa ERROR (kao ORG-01 tolerancija).

## 5. Status i otvorena pitanja

- **NFR-ORG-06** (predložen ovim DR-om, **upisan u `03-NFRQ/` 2026-07-09**; kriterij 5
  self-integritet preko `RECORD`-a dodan 2026-07-12): manifest postoji; closure ⊆ tier;
  nema dvostrukog vlasništva; SBOM prisutan po distribuciji; self-integritet preko `RECORD`-a
  (izgrađeno) / `wem_lint` digest-scan (dev). Verifikacija: `wem_lint` gate (a) + SBOM validator (b).
- **Redoslijed migracije:** ~~(4) ukidanje symlinkova~~ **provedeno 2026-07-09** — svih 9
  dev-symlinkova (8 processors→workflow, 1 workflow→processors) uklonjeno; razrješavanje
  svih 18 domena verificirano na ispravnog vlasnika preko editable installova (jedinstveno
  vlasništvo, DR cilj postignut). Git-vidljivo: 2 brisanja u processors (`decorators`,
  `helpers` — bili tracked). Preostaje: (1) formalizacija manifesta + `wem_lint` distribution-gate,
  (2) preseljenje jasnih slučajeva (`converters/`), (3) ports/adapters za zamršene
  (`logger`/`mappers`), (4) editable-bootstrap dokumentacija. Dira `core/`/`concrete/`
  (§2.5 — tražiti odluku po koraku).
- **Alat za supply-chain:** izbor SBOM formata (CycloneDX vs SPDX) i lock alata
  (`uv` vs `pip-tools`) — zasebna **detaljna analiza** do v1.0; **Snyk** (GitHub-integriran)
  kao kandidat za sigurnosnu validaciju; OSCAL vezu razraditi u OSCAL dokumentaciji.
- **Umbrella distribucije (budući `DR-WFL-003`):** registar podržanih distribucija unutar
  `wattleflow` kišobrana (core → workflow → processors/cad/oscal/…), verzijska matrica
  kompatibilnosti i politika izdavanja. Prati koje distribucije čine podržani skup.
- **`@staticmethod` migracija (ORG-05):** `cad` slice (3) je *ortogonalan* lokaciji
  (dekorator putuje s datotekom) — može se dovršiti neovisno o ovoj migraciji.

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-09 | prvi zapis, oznaka `ADR-ORG-06` / `DR-ORG-06` |
| 2 | 2026-07-28 | oznaka → `DR-WFL-002`; sadržaj odluke nepromijenjen |
