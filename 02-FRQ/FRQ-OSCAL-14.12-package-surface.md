# FRQ-OSCAL-14.12 — Javna površina paketa i vendorirani artefakti

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `wattleflow/oscal/__init__.py`, `_version.py`, `resources/`, `MANIFEST.in`, `pyproject.toml` |
| **Sestrinski** | svi `FRQ-OSCAL-14.*` — ovaj zapis kaže **što od toga vidi vanjski svijet** |
| **Izvedba** | paket `wattleflow.oscal` u `wattleflow-processors` |

## 1. Predmet

Paket je **jedina cjelina koju netko instalira**, pa je njegova površina zaseban zahtjev: koja
imena postoje, koje verzije tvrde što, i koji podaci putuju s kodom.

| element | sadržaj |
|---|---|
| `__init__.__all__` | 23 imena — modeli, loaderi, registar, `resolve`, `Crosswalk`, `OSCALPolicy`, verzijske konstante |
| `_version.py` | `__version__`, `OSCAL_VERSION` (specifikacija), `ASD_ISM_RELEASE` (izdanje ISM-a) |
| `resources/asd/` | ISM katalog + tri E8 baseline profila |
| `resources/crosswalk/` | NIST → ISM tablica |
| `MANIFEST.in` / `pyproject.toml` | `global-exclude *`, pa eksplicitno uključivanje `*.py` i `*.json` |

Vendoriranje je odluka s posljedicom: loaderi čitaju artefakte **iz instaliranog paketa**, pa
verzija ISM-a nije okolina nego dio isporuke.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Instalater (`pip`) | wheel / sdist |
| **A2** | Potrošač (`wattleflow-workflow` dekorateri, korisnički kod) | imena iz `__all__` |
| **A3** | Loaderi | putanje unutar `resources/` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 instalira distribuciju |
| **EV02** | A2 uvozi `from wattleflow.oscal import …` |
| **EV03** | A3 čita artefakt s diska instaliranog paketa |

## 4. Preduvjeti

1. Python ≥ 3.11 (`requires-python`).
2. Ovisnosti `wattleflow` i `wattleflow-workflow` su instalirane.
3. Podaci su uključeni u pakiranje — inače se paket instalira bez artefakata koje loaderi traže.

## 5. Normalan tok

1. EV01 — `MANIFEST.in` isključi sve (`global-exclude *`), pa vrati `*.py` i
   `resources/**/*.json`; `pyproject.toml` isto ponavlja kroz `package-data`. Dva zapisa istog
   pravila (§11 t.2).
2. EV02 — `__init__` uvozi imena iz podmodula **eksplicitno** i navodi ih u `__all__`; svako se
   ime razrješava.
3. EV03 — loader dobiva putanju od pozivatelja; ništa u paketu ne pretpostavlja lokaciju
   artefakta osim što je isporučen.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| ime nije u `__all__` | dostupno samo uvozom iz podmodula (npr. `ModelBase`) |
| artefakt izostane iz wheela | loader diže `FileNotFoundError` pri prvom čitanju — kvar se vidi tek u radu |
| potrošač bez `wattleflow-workflow` | uvoz `wattleflow.concrete` puca; paket bez te ovisnosti nije upotrebljiv |
| verzija ISM-a se promijeni uzvodno | paket i dalje isporučuje staru; nadogradnja je izdanje paketa, ne konfiguracija |

## 7. Rezultat

Instalacija daje kod **i** podatke: katalog, tri baselinea i crosswalk. Bez mreže, bez
konfiguracije, bez koraka koji korisnik mora zapamtiti.

## 8. Kriteriji prihvaćanja

1. Svako ime u `__all__` se razrješava. ✅
2. Uvozno zatvorenje paketa je `stdlib ∪ wattleflow` — nijedna third-party ovisnost. ✅
3. Artefakti su isporučeni i čitljivi iz instaliranog paketa. ✅
4. `OSCAL_VERSION` odgovara `metadata.oscal-version` isporučenog kataloga. ✅
5. `ASD_ISM_RELEASE` odgovara izdanju isporučenog kataloga. ✅
6. `MANIFEST.in` eksplicitno uključuje i `*.py` i `*.json`. ✅
7. `__version__` odgovara verziji distribucije. ❌ — vidi §11 t.1

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `hasattr` nad svakim imenom | 23 imena, nijedno nerazriješeno |
| 2 | pregled svih `import` naredbi u paketu | `abc`, `dataclasses`, `json`, `pathlib`, `typing`, `uuid`, `wattleflow.*` |
| 3 | popis `resources/**/*.json` | 5 datoteka, 2,41 MB; sve učitane u ovoj seriji provjera |
| 4 | `OSCAL_VERSION` vs katalog | `1.1.2` = `1.1.2` |
| 5 | `ASD_ISM_RELEASE` vs katalog | `2026.03.24` = `metadata.version`; `last_modified` `2026-03-24` |
| 6 | čitanje `MANIFEST.in` | `recursive-include` za oba obrasca |
| 7 | `__version__` vs distribucija | `src/wattleflow/oscal/_version.py` nosi `0.0.0.1`, a paket se izdaje s `wattleflow-processors` — verzija sloja više ne označava ništa što se objavljuje |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–7; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `processors` 2026-08-22.

> **Slijepa pjega (D-11), dvije.** (a) Skripta nije u repozitoriju (`CLAUDE.md` §4). (b) Kriterij
> 3 provjeren je nad **radnim stablom**, ne nad izgrađenim wheelom — da pakiranje stvarno nosi
> artefakte, dokazao bi tek `RECORD` izgrađenog wheela (`NFRQ-SEC-03` kriterij 5).

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-SEC-03` | paket stoji u čistom tieru (`stdlib ∪ wattleflow`), pa ga clean core potrošač smije uvoziti; `MANIFEST.in` je kontrolna točka pakiranja |
| `NFRQ-SEC-02` | `__all__` je deklarirana površina; baze modela namjerno nisu u njoj |
| `NFRQ-SEC-01` | vendorirani artefakti znače da paket ne poseže na mrežu pri radu |

## 11. Otvoreno

1. **Verzija ima tri izvora istine.** `_version.py` tvrdi `0.0.0.1`, `pyproject.toml` `0.0.5`,
   PyPI nosi `0.0.4`; uz to je `setuptools_scm` konfiguriran, pa bi verzija trebala dolaziti iz
   gita. `__version__` je ono što potrošač vidi i danas je netočan. Traži odluku koji je izvor
   (D-13: prikaz nije izvor istine).
2. **Pravilo pakiranja je zapisano dvaput** — `MANIFEST.in` i `[tool.setuptools.package-data]`.
   Razilaženje je tiho: sdist i wheel mogu nositi različit sadržaj.
3. **`ModelBase`, `OSCALElement` i `SelectableElement` nisu u paketnom `__all__`** iako jesu u
   `models.__all__`. Vanjska specijalizacija modela mora uvoziti iz podmodula — namjerno ili
   propust, nije zapisano (`FRQ-OSCAL-14.6` §11 t.2).
4. **Nema provjere verzije dokumenta pri učitavanju** (`FRQ-OSCAL-14.7` §11 t.2): `OSCAL_VERSION`
   postoji, ali ništa ne uspoređuje učitani dokument s njim.
