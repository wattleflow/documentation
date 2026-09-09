# FRQ-OSCAL-14.1 — Katalog sigurnosnih kontrola

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — zapisan obrnutim inženjerstvom zatečenog koda, ne unaprijed |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Catalog(OSCALElement, ISyncAggregate[Control])` — nepromjenjiv prikaz OSCAL kataloga i agregat kontrola |
| **Sestrinski** | `Control`, `Group`, `Profile`, loaderi, registar, resolver, crosswalk, policy — zapisi slijede, oznake se dodjeljuju redom |
| **Izvedba** | `src/wattleflow/oscal/models.py` u `wattleflow-processors` |

## 1. Predmet

Katalog je **izvor istine o skupu kontrola**: ono što profil bira, registar indeksira, a policy
provjerava. Zahtjev pokriva njegovu deserijalizaciju iz OSCAL JSON-a i dva pogleda koja izlaže —
ravni niz kontrola (Iterator) i obilazak stabla (Visitor).

| ugovor | odakle | što donosi |
|---|---|---|
| `ISyncAggregate[Control]` | `wattleflow.core` | `create_iterator() -> IIterator[Control]` |
| `IElement` | `wattleflow.core` | `accept(visitor)` — obilazak stabla |
| `IWattleflow` (zrcaljen) | `OSCALElement` | `name` izveden iz tipa, nepromjenjiv |
| deserijalizacija | `ModelBase` | `from_dict`, mapiranje kebab-case ključeva |

Katalog **ne razrješava profile** i **ne provodi politiku** — to su odvojeni zahtjevi. Nema
`modify`, ni parametarskih preinaka: ovdje živi samo struktura.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Vendorirani artefakt `resources/asd/ISM_catalog.json` (ASD ISM, OSCAL 1.1.2) | JSON dokument |
| **A2** | `ASDOSCALCatalogLoader` | putanja do dokumenta |
| **A3** | `Catalog` — predmet ovog zahtjeva | struktura kataloga |
| **A4** | `OSCALCatalogRegistry` | traži ravni niz kontrola za indeks |
| **A5** | `resolve()` | gradi novi katalog iz podskupa postojećeg |
| **A6** | Posjetitelj (`IVisitor`) | traži obilazak stabla |

`Control`, `Group`, `Metadata` i `BackMatter` **nisu akteri** — sudionici su strukture, ne
pokretači.

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A2 pročita dokument i pozove `Catalog.from_dict(payload)` |
| **EV02** | A4 registrira katalog i traži `create_iterator()` |
| **EV03** | A5 razrješava profil i konstruira **novi** katalog iz preživjelih čvorova |
| **EV04** | A6 traži `accept(visitor)` |

## 4. Preduvjeti

1. Dokument je OSCAL **katalog** — omotan (`{"catalog": {…}}`) ili gol; profil odbija loader,
   prije nego zahtjev dođe do ovog predmeta.
2. `uuid` je neprazan, `metadata` postoji i nosi `title`, `last-modified`, `version`,
   `oscal-version`.
3. Ključevi u kebab-case obliku i rezervirana riječ `class` razrješavaju se kroz
   `ModelBase.KEY_ALIASES`; nepoznat ključ prolazi kroz zamjenu `-` → `_`.

## 5. Normalan tok

1. EV01 — ako je dokument omotan, `from_dict` ga odmota; oba oblika daju **istu** strukturu.
2. Ključevi se normaliziraju, pa se rekurzivno grade `Metadata`, `Param`, `Control`, `Group` i
   `BackMatter`. Odsutan ili `null` popis daje **praznu listu**, nikad `None`.
3. `__post_init__` odbija prazan `uuid` prije nego objekt postoji.
4. EV02 — `create_iterator()` vraća `_ControlIterator` (`LazyIterator[Control]`), koji delegira na
   `walk_controls()`; svaki poziv daje **nov prolaz**, a instanca ostaje nedirnuta.
5. Obilazak je dubinski: kontrola prije svoje djece, sve kontrole prije grupa.
6. EV04 — `accept` posjeti katalog, pa kontrole, pa grupe; poredak je isti kao u t.5.
7. Instanca je `frozen`: izvedena inačica (razriješeni profil, orezano stablo) je **novi objekt**,
   izvorni katalog ostaje netaknut i dijeljiv među potrošačima.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `uuid` prazan | `ValueError: Catalog.uuid must not be empty` — konstrukcija ne uspijeva |
| `metadata` nedostaje | `KeyError: 'metadata'` iz `from_dict` |
| `controls` / `groups` odsutni ili `null` | prazna lista; katalog bez kontrola je valjan objekt |
| dokument je profil | odbija ga loader (`ValueError`), `from_dict` se ne poziva |
| pokušaj izmjene polja | `FrozenInstanceError` |
| isti `control-id` u dva kataloga | katalog ne razrješava sukob — pravilo „kasniji pobjeđuje" pripada registru |
| katalog bez ijedne kontrole | iterator daje prazan niz; nije kvar ovog predmeta |

## 7. Rezultat

Struktura kataloga je u memoriji, nepromjenjiva i dijeljiva: registar je indeksira, resolver iz
nje izvodi podskup, posjetitelj je obilazi — bez ijedne kopije koja bi mogla odlutati od izvora.
Neispravan dokument pada pri konstrukciji, ne pri uporabi.

## 8. Kriteriji prihvaćanja

1. `from_dict` prihvaća omotani i goli oblik i daje **istu** strukturu. ✅
2. Prazan `uuid` zaustavlja konstrukciju; nedostatak `metadata` je greška, ne tiha praznina. ✅
3. Kebab-case i `class` mapiraju se bez gubitka (`last-modified` → `last_modified`,
   `oscal-version` → `oscal_version`, `back-matter` → `back_matter`, `class` → `class_`). ✅
4. `create_iterator()` daje **sve** kontrole, uključujući ugniježđene, dubinski i ponovljivo. ✅
5. `accept` obilazi katalog → kontrole → grupe, istim poretkom kao iterator. ✅
6. Instanca je nepromjenjiva. ✅
7. Ugovori `ISyncAggregate[Control]` i `IElement` provedeni su — nijedna apstraktna metoda ne
   ostaje neimplementirana. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `asdict(from_dict(raw)) == asdict(from_dict(raw["catalog"]))` | jednako |
| 2 | negativni slučajevi | `ValueError: Catalog.uuid must not be empty`; `KeyError: 'metadata'` |
| 3 | čitanje polja nakon učitavanja ISM kataloga | `oscal_version = 1.1.2`, `last_modified` postavljen, `back_matter` prisutan, `class_ = ISM-principle` |
| 4 | dva uzastopna prolaza nad ISM katalogom | 1130 kontrola, nizovi identični, tip `_ControlIterator` je `IIterator` |
| 5 | posjetitelj koji bilježi tipove | 1695 posjeta: 1 katalog, 564 grupe, 1130 kontrola; prvi posjet je katalog |
| 6 | pokušaj pridruživanja | `FrozenInstanceError` |
| 7 | `__abstractmethods__` konkretne klase | prazno |

**Trojka reproducibilnosti (D-10):** alat = skripta `fr_oscal_01.py`; kriterij = §8 t.1–7 ovog
dokumenta; platforma = Python 3.11.15, Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt
`resources/asd/ISM_catalog.json` (ISM izdanje 2026-03-24).

> **Slijepa pjega (D-11), dvije.** (a) Skripta iz trojke **nije u repozitoriju** — test framework
> nije odabran (`CLAUDE.md` §4), pa je izvedba jednokratna i nije ponovljiva iz stabla. (b)
> Isporučeni ISM katalog **nema ugniježđenih kontrola ni kontrola na vrhu** (svih 1130 leži unutar
> 564 grupe, ravno): rekurzivna grana kriterija 4 i 5 provjerena je samo sintetičkim stablima
> (281 nasumično stablo, usporedba s prethodnom izvedbom), ne isporučenim artefaktom.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-SEC-03` | uvozno zatvorenje paketa je `stdlib ∪ wattleflow` — nijedan third-party uvoz; sloj stoji u čistom tieru, premda ga nosi distribucija koja to nije (`CLAUDE.md` §7.4) |
| `NFRQ-SEC-02` | javna površina modula deklarirana kroz `__all__`; `_ControlIterator` ostaje privatan |
| `NFRQ-ORG-05` | pomoćne metode deserijalizacije i obilaska su članovi klasa (`ModelBase`, `SelectableElement`), ne funkcije modula; samoreferencu nose kroz `cls`/`self` |
| `NFRQ-ORG-08` | mapiranje ključeva postoji na **jednom** mjestu (`ModelBase.KEY_ALIASES`); nijedan model ga ne prepisuje |

OSCAL kontrole (`OSCAL_CONTROLS`) **ne pripadaju ovom predmetu**: deklariraju ih komponente koje se
provjeravaju, a ovaj paket je mehanizam provjere, ne njezin subjekt.

## 11. Otvoreno

1. ~~Kategorija i numeracija.~~ **Riješeno** [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) (2026-08-21): `OSCAL` je u vokabularu, broj
   sposobnosti je `14`, djeca nose oblik `FRQ-OSCAL-14.M`. Preostaje da razred `HLRQ` sam nije u
   vokabularu (`CLAUDE.md` §3.6) — vrijedi za `HLRQ-14` kao i za `HLRQ-13`.
2. ~~**`CLAUDE.md` §6.1 tvrdi da OSCAL paket nije deployan.**~~ **Otpalo 2026-08-22:** OSCAL
   više nije zaseban paket nego je vendiran u `wattleflow-processors`; §6.1 je usklađen.
3. **Katalog ne provjerava jedinstvenost `control-id`.** Duplikat unutar **jednog** kataloga
   prolazi nezapažen; registar ga vidi tek preko pravila „kasniji pobjeđuje". Je li to kvar
   kataloga ili prihvaćeno ponašanje — nije odlučeno.
4. **`params` na razini kataloga se prenose, ali ih ništa ne troši.** Ni resolver ni policy ih ne
   čitaju. Ostaviti kao vjerni prikaz OSCAL modela ili ukloniti — otvoreno.
