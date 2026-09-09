# FRQ-OSCAL-14.7 — Učitavanje OSCAL dokumenata

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `ASDOSCALLoader` (baza), `ASDOSCALCatalogLoader`, `ASDOSCALProfileLoader` |
| **Sestrinski** | [`14.1`](FRQ-OSCAL-14.1-catalog.md) katalog · [`14.4`](FRQ-OSCAL-14.4-profile.md) profil |
| **Izvedba** | `src/wattleflow/oscal/loaders.py` |

## 1. Predmet

Loader je **granica prema disku**: jedino mjesto u paketu koje otvara datoteku i pretvara bajtove
u model. Dva su, po vrsti dokumenta, i oba su strategije (`IStrategy`) — ne funkcije — pa se
registriraju i pozivaju kao svaka druga strategija u frameworku.

| klasa | ugovor | vraća |
|---|---|---|
| `ASDOSCALLoader(Wattleflow, IStrategy, ABC)` | `_require_path`, `_read_json`; apstraktni `execute` | — |
| `ASDOSCALCatalogLoader` | `execute(caller, path=…)` | `Catalog` |
| `ASDOSCALProfileLoader` | `execute(caller, path=…)` | `Profile` |

**Vrsta dokumenta se provjerava, ne pretpostavlja.** Katalog i profil su različiti OSCAL
dokumenti; zamjena bi tiho dala prazan ili besmislen rezultat, pa je odbijanje eksplicitno u oba
smjera.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Pozivatelj (`Workflow`, `Processor`, test, skripta) | `caller` i `path` |
| **A2** | `ASDOSCAL*Loader` — predmet | putanja |
| **A3** | Datotečni sustav / vendorirani `resources/asd/` | JSON dokument |
| **A4** | `Catalog` / `Profile` | `from_dict` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 pozove `execute(caller=…, path=…)` |
| **EV02** | A2 čita dokument s A3 |
| **EV03** | A2 provjerava vrstu dokumenta |
| **EV04** | A2 predaje sadržaj A4 |

## 4. Preduvjeti

1. `path` je zadan kao imenovani argument; `str` i `Path` su oba prihvatljivi (`Pathish`).
2. Putanja pokazuje na postojeću **datoteku** čiji je sadržaj valjan JSON.
3. Dokument je one vrste koju loader očekuje.

## 5. Normalan tok

1. EV01 — `_require_path` potvrdi da `path` postoji među argumentima; poruka greške imenuje
   **konkretni** loader (izvedeno iz `cls`, ne iz prepisanog niza).
2. EV02 — `_read_json` potvrdi da je putanja datoteka, pa je pročita u UTF-8.
3. EV03 — katalog loader odbija dokument s ključem `profile`, profilni odbija onaj s `catalog`.
   Poruka nosi putanju **i** ime loadera koji je trebalo upotrijebiti.
4. EV04 — `Catalog.from_dict` / `Profile.from_dict`; omotani i goli oblik oba prolaze
   (`FRQ-OSCAL-14.1` §5, `FRQ-OSCAL-14.4` §5).
5. `caller` se prima jer ga ugovor `IStrategy` traži; loader ga **ne koristi**.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `path` izostane | `ValueError: <Loader> requires 'path' keyword argument` |
| putanja ne postoji ili nije datoteka | `FileNotFoundError: OSCAL JSON not found: <putanja>` |
| sadržaj nije valjan JSON | `json.JSONDecodeError` — propušta se nepromijenjen |
| katalog loader nad profilom | `ValueError: <putanja> is an OSCAL Profile document — use ASDOSCALProfileLoader` |
| profilni loader nad katalogom | zrcalna poruka |
| dokument prođe provjeru, ali mu nedostaje polje | greška dolazi iz modela (`KeyError`), ne iz loadera |

## 7. Rezultat

Model je izgrađen ili je pozivatelj dobio grešku koja imenuje **datoteku, loader i ispravan
loader**. Nema djelomično učitanog stanja: `from_dict` ili uspije ili digne iznimku.

## 8. Kriteriji prihvaćanja

1. `path` je obvezan; poruka imenuje konkretni loader, ne bazu. ✅
2. Nepostojeća datoteka daje `FileNotFoundError` s putanjom. ✅
3. Zamjena vrste dokumenta odbija se u **oba** smjera, s uputom na ispravan loader. ✅
4. Uspješan poziv vraća `Catalog`, odnosno `Profile`. ✅
5. Baza je apstraktna i nosi ugovor `IStrategy`; obje izvedbe su konkretne. ✅
6. Nijedna pomoćna funkcija ne živi na razini modula. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `execute(caller=None)` na oba loadera | `ASDOSCALCatalogLoader requires 'path' keyword argument`; isto za `ASDOSCALProfileLoader` |
| 2 | nepostojeća putanja | `FileNotFoundError: OSCAL JSON not found: …/nope.json` |
| 3 | unakrsni pozivi nad isporučenim artefaktima | `ValueError` u oba smjera, s putanjom u poruci |
| 4 | `execute` nad ISM katalogom i ML1 profilom | `Catalog`, `Profile` |
| 5 | `inspect.isabstract`, `issubclass` | baza apstraktna, `IStrategy` i `Wattleflow` u MRO |
| 6 | pretraga modula za funkcijama | nijedna |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakti ISM 2026-03-24.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4). Ponašanje na neispravnom
> JSON-u (`JSONDecodeError`) **nije** provjereno izvršavanjem — zaključeno je čitanjem koda.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-SEC-02` | jedina točka koja otvara datoteku u paketu; površina prema disku je jedna, ne raspršena |
| `NFRQ-SEC-03` | učitavanje koristi `json` iz stdlib-a — bez YAML-a i bez third-party parsera |
| `NFRQ-ORG-05` | `_require_path` je `@classmethod` (koristi `cls.__name__`), `_read_json` `@staticmethod` |
| `NFRQ-ORG-08` | čitanje i provjera argumenata postoje jednom, na bazi |

## 11. Otvoreno

1. **`caller` se ne koristi ni za što** — ni za zapis, ni za provjeru prava. Ugovor `IStrategy`
   ga traži, pa stoji u potpisu kao mrtav argument. Iskoristiti ga (audit trag tko je učitao koji
   dokument) ili zapisati da je namjerno neiskorišten.
2. **Nema provjere OSCAL verzije dokumenta.** `OSCAL_VERSION = "1.1.2"` postoji u `_version.py`,
   ali loader ne uspoređuje `metadata.oscal_version` s njim; dokument druge verzije učitat će se
   tiho.
3. **Nema ograničenja veličine ni vremena** pri čitanju. Za vendorirane artefakte nebitno; za
   putanju koju zada korisnik to je otvorena površina.
