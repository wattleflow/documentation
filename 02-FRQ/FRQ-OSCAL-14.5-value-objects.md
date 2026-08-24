# FR-OSCAL-14.5 — Vrijednosni objekti kataloga

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) (kategorija); grupiranje šest klasa u jedan zapis obrazloženo je u §1 |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Prop`, `Link`, `Part`, `Param`, `Metadata`, `BackMatter` |
| **Sestrinski** | [`14.2`](FRQ-OSCAL-14.2-control.md) kontrola · [`14.6`](FRQ-OSCAL-14.6-bases.md) baze |
| **Izvedba** | `src/wattleflow/oscal/models.py` |

## 1. Predmet

Šest klasa dijeli **isti ugovor i istu ulogu**: nepromjenjiv nositelj sadržaja koji nema identitet
u sustavu, ne sudjeluje u izboru i ne obilazi se posjetiteljem. Zaseban zapis po klasi ponovio bi
deset istih rečenica šest puta — protivno ugovoru kratkoće (skill §2 t.3) i namjeri da OSCAL
funkcionalnost ostane cjelovita, a ne rasuta.

| klasa | obvezna polja | uloga | pojava u ISM izdanju |
|---|---|---|---|
| `Prop` | `name`, `value` | imenovano svojstvo (`ns`, `class_`, `uuid` opcionalni) | sve 1130 kontrola |
| `Link` | `href` | veza (`rel`, `text`, `media_type`) | metapodaci, resursi |
| `Part` | `name` | tekstualni dio, **rekurzivan** (`parts`) | sve 1130 kontrola |
| `Param` | `id` | parametar kontrole | 0 |
| `Metadata` | `title`, `last_modified`, `version`, `oscal_version` | zaglavlje dokumenta | katalog i svaki profil |
| `BackMatter` | — | resursi dokumenta | 238 resursa |

**`Prop.name` i `Part.name` su podatak, ne identitet.** To je razlog zašto ove klase ne nose
`IWattleflow` ugovor — vidi [`14.6`](FRQ-OSCAL-14.6-bases.md) §11 t.1.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `Catalog`, `Control`, `Group`, `Profile`, `Metadata` | roditelj koji gradi |
| **A2** | Potrošač izvještaja (izvan ovog paketa) | čita sadržaj kontrole |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | roditelj gradi svoj popis → `<Klasa>.from_dict(payload)` |
| **EV02** | `Part.from_dict` nailazi na ugniježđene `parts` → rekurzija |

## 4. Preduvjeti

1. Obvezna polja iz tablice §1 postoje u dokumentu.
2. Ključevi u kebab-case obliku i `class` razrješavaju se kroz `ModelBase.KEY_ALIASES`.

## 5. Normalan tok

1. EV01 — normalizacija ključeva, pa konstrukcija; opcionalna polja koja izostanu daju `None`,
   izostali popisi daju **praznu listu**.
2. EV02 — `Part` gradi vlastite `props`, `links` i ugniježđene `parts` istim pravilima.
3. `BackMatter` je jedini koji **tolerira prazan ulaz**: `{}` i `None` daju objekt bez resursa.
   Resursi ostaju **sirovi rječnici** — ne modelira ih se.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `Prop` bez `value` | `KeyError: 'value'` |
| `Link` bez `href` | `KeyError: 'href'` |
| `Metadata` bez `last-modified` | `KeyError: 'last_modified'` — poruka nosi **normalizirano** ime |
| popis zadan kao `null` | prazna lista |
| `BackMatter.from_dict({})` / `(None)` | `resources = []` |
| izmjena polja | `FrozenInstanceError` |

## 7. Rezultat

Sadržaj kontrole je čitljiv i nepromjenjiv; jedini dio koji ostaje netipiziran su `back-matter`
resursi, namjerno — nijedan potrošač u ovom paketu ih ne čita.

## 8. Kriteriji prihvaćanja

1. Obvezna polja su obvezna; njihov izostanak je `KeyError`, ne tiha praznina. ✅
2. Opcionalna polja izostankom daju `None`, popisi praznu listu. ✅
3. Kebab-case i `class` mapiraju se u sva polja, uključujući ugniježđena. ✅
4. `Part` je rekurzivan bez ograničenja dubine. ✅
5. `BackMatter` tolerira prazan i nepostojeći ulaz. ✅
6. Nijedan od šest nije `IElement` — ne obilaze se posjetiteljem. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | negativni slučajevi | `KeyError: 'value'`, `KeyError: 'href'`, `KeyError: 'last_modified'` |
| 2 | `Param.from_dict({"id": "p", "values": None})` | `values = []`, `class_ = None` |
| 3 | `Prop`, `Link` s `class` / `media-type` | `class_ = "C"`, `media_type = "text/html"` |
| 4 | `Part.from_dict` s ugniježđenim `parts` | dijete izgrađeno, `props`/`links` prazni |
| 5 | `BackMatter.from_dict({})` i `(None)` | `{"resources": []}` u oba slučaja |
| 6 | `issubclass(..., IElement)` za svih šest | nijedan |
| pojava | ISM katalog | 1130 kontrola s `props` i `parts`; 0 s `params`; 238 resursa u `back_matter` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt ISM 2026-03-24.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4). `Param` nema nijednu
> pojavu u isporučenom katalogu — provjeren je isključivo sintetički.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFR-ORG-08` | mapiranje ključeva je naslijeđeno iz `ModelBase`; nijedna od šest klasa ga ne prepisuje |
| `NFR-SEC-02` | `back-matter` resursi ostaju sirovi — model ne izmišlja strukturu koju ne treba |

## 11. Otvoreno

1. **`BackMatter.resources` je `List[Dict[str, Any]]`** — netipizirano. Ako neki potrošač počne
   čitati resurse (npr. razrješavanje `href="#uuid"` iz profila, `FR-OSCAL-14.4` §11 t.2), tip
   treba modelirati. Do tada je netipiziranost namjerna, ne propust.
2. **Dva aliasa nemaju polje koje ih prima.** `responsible-parties` → `responsible_parties` i
   `control-id` → `control_id` postoje u `ModelBase.KEY_ALIASES`, ali nijedan model nema tako
   nazvano polje (provjereno pretragom stabla): ključ se normalizira pa tiho odbaci. Ukloniti
   aliase ili modelirati polja — dok stoje, sugeriraju podršku koje nema.
3. **Poruka o nedostajućem polju nosi normalizirano ime** (`last_modified`), a korisnik u
   dokumentu vidi `last-modified`. Sitno, ali otežava traženje uzroka.
