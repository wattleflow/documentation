# DR-WFL-001 — Cross-cutting sposobnosti kao helper klase (routing i file-discovery)

| | |
|---|---|
| **Status** | Prihvaćeno (implementirano) — predlaže **NFRQ-ORG-04** (Draft, čeka ratifikaciju u `03-NFRQ/`) |
| **Datum** | 2026-07-02 |
| **Verzija** | 2 (2026-07-28) — v1: 2026-07-02 |
| **Realizira** | NFRQ-ORG-01 (Dependency Locality), NFRQ-ORG-02 (Class Nomenclature) |
| **Predlaže** | NFRQ-ORG-04 (Cross-cutting sposobnost = helper, ne domenski primitiv) |
| **Kontekst rada** | `wattleflow-processors` — `EntityFileDocumentProcessor`, primjer `03_pii_complex_workflow` |
| **Sljedivost** | `PHILOSOPHY.md` (Ontologija, Samoopisivost) · `METHODOLOGY.md` §5, §9 · `03-NFRQ/` NFRQ-ORG-01/02 |

---

## 1. Kontekst (problem / prilika)

Workflow treba, pri perzistenciji, poslati dokument na **odredište** koje ovisi o
transportu: pod-direktorij (filesystem), topic (Kafka), indeks (Elasticsearch),
tablica (SQL). Konkretan povod: jedan procesor po tipu datoteke (pdf/eml/msg)
mora posluživati više kategorija i svaku datoteku usmjeriti u njen pod-direktorij.

Iz toga su izronila tri pitanja koja nadilaze konkretni workflow i ponavljat će se
**gdje god je routing potreban**:

1. **Kome pripada logika rutiranja** — Pipeline, Driver, Strategy ili helper?
2. **Kako imenovati** nove klase da budu NFR-usklađene i samoopisive?
3. **Gdje ih smjestiti** (lokalnost ovisnosti) bez bloata jezgre i bez ciklusa?

Uz to, procesor je miješao dvije odgovornosti: *otkrivanje datoteka* (skeniranje,
filtriranje po imenu) i *perzistenciju* (ekstrakcija, kreiranje dokumenta).

## 2. Odluka

### 2.1 Routing je **sposobnost (helper)**, ne `Strategy`

Rutiranje se modelira kao **pozivljiva sposobnost** u `helpers/routing.py`, s
apstraktnim ugovorom i per-transport implementacijama — **ne** kao `Strategy`.

- **`Strategy` je rezervirani ontološki primitiv** (create/read/write koje delegira
  Repository/Blackboard, `PHILOSOPHY.md` — Ontologija). Rutiranje nije primitiv.
- **GoF Strategy** poziva *kontekst*; ne kombinira se u susjedne strategije →
  kao `Strategy`, routing ne bi mogle koristiti druge strategije (ni procesori/driveri).
- Kao helper, routing je **pozivljiv iz bilo kojeg potrošača**.

### 2.2 Neutralni ključ je `route`, ne `partition`

Logička labela rute putuje u `document.metadata` pod `ROUTE_KEY = "route"`.
`partition` je **odbačen** jer u Kafki/Sparku označava fizičku particiju
(`msg.partition`, `partition_by`) — semantička kolizija. Router prevodi `route` u
driverov *native* parametar (`subdir`/`index`/`topic`).

### 2.3 Uvedene helper klase (NFRQ-ORG-02 usklađene)

| Klasa / funkcija | Odgovornost | Modul |
|---|---|---|
| `RoutingLabel(name, target)` | par labela→odredište | `helpers/routing.py` |
| `RoutingRule` | `classify(name) → RoutingLabel` (format `{label}`) | `helpers/routing.py` |
| `PatternSpec` | parsira `pattern` Union (str \| dict) → glob + rule | `helpers/routing.py` |
| `route_label` / `route_target` | čitanje rute iz metapodataka (dvoskok) | `helpers/routing.py` |
| `DestinationRouter` (apstraktni) | `resolve(route) → driver.write kwargs` | `helpers/routing.py` |
| `LocalStorageDestinationRouter` | filesystem: `route → {subdir, filename, suffix}` | `helpers/routing.py` |
| `FileSourceScanner` | config-vođeni generator: `yield Path` po **imenu** | `helpers/files.py` |

Sva su imena **kvalificirana i uloga-eksplicitna** (bez gole generičke imenice;
`Destination`/`Route` kvalificiraju role-nouns `Router`/`Rule`/`Label`), konzistentno
s postojećim `FileScanner`, `FormatterFactory`.

### 2.4 `pattern` postaje `Union[str, dict]`

Jednostavno: `pattern: "*.pdf"` (samo glob). Složeno: `pattern: {glob, format, labels[{name,target,…}]}`.
`PatternSpec.parse` normalizira oboje; `categories`/`category_format`/`category`
(interim ključevi) su **uklonjeni**.

Svaka labela bira matcher po prioritetu: **`regex`** (sirovi regex) > **`match`**
(GLOB, `fnmatch`, case-insensitive, npr. `2026-05*.pdf`) > zajednički **`format`**
s `{label}` (regex, ime umetnuto) > (fallback) ime kao glob. Glob je prvorazredan
jer je to framework konvencija (`FileScanner`, `pattern: "2026-06*.pdf"`) i prirodan
način izražavanja datuma u imenima.

### 2.5 Odvajanje otkrivanja od perzistencije + matchanje po **imenu**

`FileSourceScanner` preuzima otkrivanje (glob + name-exclusion; vlada `PatternSpec`-om),
`yield Path`. Procesor zadržava routing (`scanner.rule.classify(path.name)`) i
perzistenciju. **Klasifikacija je po imenu datoteke, nikad po putanji** (funkcionalni
zahtjev: procesor pretražuje nazive, ne путању).

### 2.6 Provenijencija: `metadata` nije config ključ

`metadata` je Document-ova riječ. Procesor prosljeđuje klasificiranu labelu kao
par `{label.name: label.target}` + pokazivač `ROUTE_KEY → label.name` kroz
`blackboard.create(...)`; **Create strategija** žigoše proslijeđene k-v u
`document.metadata` (create-time provenijencijski kanal). Ugniježđeni `metadata:` blok
je uklonjen.

## 3. Načela i sljedivost (`METHODOLOGY.md` §5)

| Načelo | Primjena |
|---|---|
| Ontologija prije implementacije | `Strategy` ostaje rezerviran; routing je fabricirana sposobnost |
| GRASP — Pure Fabrication, High Cohesion, Low Coupling | cross-cutting odgovornost bez matične domene → kohezivan helper |
| SRP / Separation of Concerns (Parnas, Dijkstra) | „gdje ide" (router) ⟂ „kako se perzistira" (write) ⟂ „koji fajlovi" (scanner) |
| DIP / Stable Abstractions | strategije ovise o `DestinationRouter` apstrakciji, ne o konkretima |
| Open–Closed | novi transport = novi `*DestinationRouter`, bez diranja postojećih |
| NFRQ-ORG-01 t.4 | dijeljeni helper imenovan po **sposobnosti** (`routing`), ne po sloju |
| NFRQ-ORG-02 | kvalificirana, uloga-eksplicitna imena; akronimi velikim slovom |
| KISS / YAGNI / Occam | isporučen samo filesystem router; Kafka/ES apstrakcija oblikovana, ne napisana |
| Information Hiding | `route` je transport-neutralan; driver-specifičnosti skrivene u routeru |

## 4. Posljedice

**Pozitivno**
- Routing je pozivljiv iz strategija, procesora i (budućih) drivera — jedna sposobnost, više potrošača.
- `helpers/routing.py` je **stdlib-only** → poštuje zero-trust granicu clean core (§7.4).
- `wem_lint` ostaje na baselineu (9 ERROR + 6 WARN) — nove klase ne uvode ORG-01 povredu (acikličnost, kriterij 3, čista).
- Procesor je tanji; `FileSourceScanner` je ponovno iskoristiv za druge file-procesore.

**Negativno / rizik**
- **Per-class fan-in (NFRQ-ORG-01 kriterij 1).** `RoutingRule`/`PatternSpec` trenutno koristi samo `processors`, `*DestinationRouter` samo `strategies`. Zajednički `ROUTE_KEY` + apstraktni ugovor su genuino ≥2 domene i sidre modul u dijeljeni `helpers/` (izbjegavaju processors↔strategies brid). Kad druga domena (npr. `drivers`) usvoji router, kriterij 1 je zadovoljen i za konkrete. Do tada: prihvaćeni WARN (tolerancija).
- Create strategije `file.py`/`pdf.py` sada žigošu proslijeđene k-v; ostale create strategije **nisu** ažurirane (naknadno, po potrebi).
- Dvoskok dohvat (`route_target`) je indirektan; opravdan jer čuva `{name: target}` par kao provenijenciju bez dupliranja targeta.

## 5. Status i otvorena pitanja

- **NFRQ-ORG-04** (predložen ovim DR-om): *„Cross-cutting sposobnost (routing, adresiranje) modelira se kao pozivljiva helper-sposobnost, ne kao domenski `Strategy` primitiv; smješta se po sposobnosti (NFRQ-ORG-01 t.4)."* — čeka unos u `03-NFRQ/` + `tools/naming_registry.yaml` (dodati helper role-nouns po potrebi).
- **Promocija u globalni `helpers/`** je već učinjena (modul je stdlib-only, sidren zajedničkim `route` vokabularom); revidirati per-class fan-in kad se pojavi drugi domenski potrošač.
- **Matcher vs stvarni podaci:** klasifikacija je nad **imenom**; labela bira `regex`/`match`(glob)/`format{label}`. Prilagoditi stvarnoj konvenciji (npr. glob `2026-05*.pdf`).

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-02 | prvi zapis, oznaka `ADR-ORG-04` / `DR-ORG-04` |
| 2 | 2026-07-28 | oznaka → `DR-WFL-001` (serija po projektu); `ORG-NN` veza premještena u polje *Realizira*; sadržaj odluke nepromijenjen |
