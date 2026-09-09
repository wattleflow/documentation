# OSCAL — zahtjevi

Zapisi zahtjeva za OSCAL sloj, koji **nije zasebna distribucija** nego živi u
`wattleflow-processors` (`src/wattleflow/oscal/`, `src/wattleflow/decorators/oscal/`). Registar
oznaka je [`FRQ`](../02-FRQ/FRQ-000-EN.md); ovdje je samo pregled i redoslijed čitanja.

> **Kategorija i numeracija su odlučene** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) (2026-08-21): `OSCAL` je u kontroliranom
> vokabularu, os mu je sposobnost/distribucija, broj sposobnosti je `14`. Razred `HLRQ` sam
> još nije u vokabularu (`CLAUDE.md` §3.6).

## Nadređeni zahtjev

[`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti, mjesto u dekompoziciji, poslovna pravila
`BR-OSCAL-01…BR-OSCAL-12` i nefunkcionalni zahtjevi. Zapisi ispod ga referiraju umjesto da ponavljaju kontekst.

## Redoslijed čitanja

| zapis | predmet | modul |
|---|---|---|
| [14.6](../02-FRQ/FRQ-OSCAL-14.6-bases.md) | `ModelBase`, `OSCALElement`, `SelectableElement` — ugovori koje troše svi ostali | `models.py` |
| [14.1](../02-FRQ/FRQ-OSCAL-14.1-catalog.md) | `Catalog` — agregat kontrola | `models.py` |
| [14.2](../02-FRQ/FRQ-OSCAL-14.2-control.md) | `Control` — jedinica po kojoj se sudi | `models.py` |
| [14.3](../02-FRQ/FRQ-OSCAL-14.3-group.md) | `Group` — hijerarhija kataloga | `models.py` |
| [14.4](../02-FRQ/FRQ-OSCAL-14.4-profile.md) | `Profile`, `Import`, `IncludeControls`, `Merge` — baseline kao selektor | `models.py` |
| [14.5](../02-FRQ/FRQ-OSCAL-14.5-value-objects.md) | `Prop`, `Link`, `Part`, `Param`, `Metadata`, `BackMatter` | `models.py` |
| [14.7](../02-FRQ/FRQ-OSCAL-14.7-loaders.md) | `ASDOSCALLoader` i dvije izvedbe — granica prema disku | `loaders.py` |
| [14.8](../02-FRQ/FRQ-OSCAL-14.8-registry.md) | `OSCALCatalogRegistry` — ravni indeks kontrola | `registry.py` |
| [14.9](../02-FRQ/FRQ-OSCAL-14.9-resolver.md) | `resolve()` — profil → katalog | `resolver.py` |
| [14.10](../02-FRQ/FRQ-OSCAL-14.10-crosswalk.md) | `Crosswalk` — prijevod taksonomije | `crosswalk.py` |
| [14.11](../02-FRQ/FRQ-OSCAL-14.11-policy.md) | `OSCALPolicy` — gate `declared ⊆ baseline` | `policy.py` |
| [14.12](../02-FRQ/FRQ-OSCAL-14.12-package-surface.md) | javna površina paketa i vendorirani artefakti | `__init__.py`, `_version.py`, `resources/` |
| [14.13](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md) | `OSCALConnection`, `OSCALDriver`, `OSCALProcessor` — vrata na bazi | `wattleflow-processors` |

## Otvorene stavke po zapisima

Stavke se ovdje **ne prepričavaju** (D-13) — svaka živi u §11 svojega zapisa. Ovo je samo popis
gdje ih tražiti, po težini:

| težina | stavka | gdje |
|---|---|---|
| traži DR | vrijednosni objekti nemaju core pattern (`CLAUDE.md` §2.5) | [14.6](../02-FRQ/FRQ-OSCAL-14.6-bases.md) §11 t.1 |
| ~~traži DR~~ **otpalo** | pitanje ovisnosti nestalo je vendiranjem OSCAL sloja u `wattleflow-processors`; nijedna distribucija ne ovisi o `wattleflow-oscal` | [HLRQ-14](../01-HLRQ/HLRQ-14-oscal.md) §7 t.1 |
| **nalaz** | komponentne baze ne postoje u kodu — dekorater stoji izravno na 15 klasa, a zapis se vodi kao „Provedeno" | [14.13](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md) |
| kvar u tišini | vrata su postavljena, ali inertna — nitko ne predaje `oscal_policy=` | [14.11](../02-FRQ/FRQ-OSCAL-14.11-policy.md) §11 t.5 |
| traži sign-off | crosswalk mapiranja su `proposed` | [14.10](../02-FRQ/FRQ-OSCAL-14.10-crosswalk.md) §11 t.1 |
| kvar u tišini | `strict=False` gubi nedostajuće `id`-eve bez traga | [14.9](../02-FRQ/FRQ-OSCAL-14.9-resolver.md) §11 t.1 |
| kvar u tišini | sudar `control-id` među katalozima prepisuje bez upozorenja | [14.8](../02-FRQ/FRQ-OSCAL-14.8-registry.md) §11 t.1 |
| kvar u tišini | prazna deklaracija uvijek prolazi gate | [14.11](../02-FRQ/FRQ-OSCAL-14.11-policy.md) §11 t.1 |
| kvar u tišini | `modify` / `matching` / `include-all` se prihvaćaju, a ne primjenjuju | [14.4](../02-FRQ/FRQ-OSCAL-14.4-profile.md) §11 t.1 |
| nedosljednost | `Group.accept` i `Group.walk_controls` daju različit redoslijed | [14.3](../02-FRQ/FRQ-OSCAL-14.3-group.md) §11 t.1 |
| nedosljednost | verzija ima tri izvora istine (`0.0.0.1` / `0.0.5` / PyPI `0.0.4`) | [14.12](../02-FRQ/FRQ-OSCAL-14.12-package-surface.md) §11 t.1 |
| nedosljednost | `Control` je `frozen`, ali nije hashabilan | [14.2](../02-FRQ/FRQ-OSCAL-14.2-control.md) §11 t.1 |
| čistoća | dva aliasa (`responsible-parties`, `control-id`) nemaju polje koje ih prima | [14.5](../02-FRQ/FRQ-OSCAL-14.5-value-objects.md) §11 t.2 |

## Zajednička slijepa pjega (D-11)

Sve provjere u §9 svakog zapisa izvedene su **jednokratnim skriptama u radnom okruženju**, koje
nisu u repozitoriju — test framework nije odabran (`CLAUDE.md` §4). Dok se ne odabere, „provjereno"
znači *izvršeno 2026-08-21 na Python 3.11.15 / Linux (WSL2)*, ne *ponovljivo iz stabla*.
