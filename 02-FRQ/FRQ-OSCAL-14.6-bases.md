# FR-OSCAL-14.6 — Bazne klase modela

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) (kategorija); ograničenje iz §1 t.3 je posljedica jezika, ne izbora |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `ModelBase`, `OSCALElement`, `SelectableElement` |
| **Sestrinski** | svi ostali `FR-OSCAL-14.*` — ovaj zapis nosi ugovor koji oni troše |
| **Izvedba** | `src/wattleflow/oscal/models.py` |

## 1. Predmet

Tri baze dijele obveze koje bi se inače prepisivale po klasama. Svaka nosi **jedan** ugovor:

| baza | ugovor | tko je nasljeđuje |
|---|---|---|
| `ModelBase(ABC)` | deserijalizacija: `KEY_ALIASES`, `_normalise_key`, `_normalise`, `_items`, apstraktni `from_dict` | svaki model bez iznimke |
| `OSCALElement(ModelBase, IElement, ABC)` | identitet (`name`) + obilazak (`accept`) | `Control`, `Group`, `Catalog`, `Profile` |
| `SelectableElement(OSCALElement, ABC)` | izbor: apstraktni `pruned`, dijeljeni `prune_all` | `Control`, `Group` |

Tri ograničenja koja objašnjavaju **zašto baš ovako**:

1. **Ništa ne živi na razini modula.** Pomoćne metode su članovi klase; što referira vlastitu
   klasu je `@classmethod` (`cls.KEY_ALIASES`), što ne referira ništa je `@staticmethod`
   (`_items`) — `NFR-ORG-05`.
2. **`Catalog` i `Profile` nisu selektabilni.** Nemaju djecu koja se biraju, pa ne nose ugovor
   koji ne mogu ispuniti.
3. **Framework root se ne može nasljeđivati.** `wattleflow.concrete.base.Wattleflow` nasljeđuje
   `Audit`, čiji `__init__` pridružuje atribute instanci: `frozen` dataclass ga nikad ne zove
   (generirani `__init__` ne lanča), a ručni poziv puca (`FrozenInstanceError`). Uz to `Audit`
   izlaže `name` kao property, što dataclass pročita kao **default polja** — i `Prop`, čije je
   `name` podatak, prestaje biti konstruktibilan. Zato `OSCALElement` **zrcali** `IWattleflow`
   ugovor umjesto da ga naslijedi.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Model koji nasljeđuje bazu | vlastita polja |
| **A2** | `resolve()` | skup odabranih `id`-eva |
| **A3** | Posjetitelj (`IVisitor`) | obilazak |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 poziva `cls._normalise(data)` u vlastitom `from_dict` |
| **EV02** | A2 poziva `prune_all(nodes, selected)` nad popisom čvorova |
| **EV03** | ABCMeta provjerava ugovor pri instanciranju A1 |

## 4. Preduvjeti

1. Svaka konkretna klasa implementira `from_dict`; element uz to `accept`; selektabilni čvor uz
   to `pruned`.
2. `KEY_ALIASES` je čitljiv i proširiv iz podklase, bez diranja baze.

## 5. Normalan tok

1. EV01 — `_normalise` mapira svaki ključ kroz `cls._normalise_key`: alias iz `cls.KEY_ALIASES`
   ako postoji, inače zamjena `-` → `_`. Podklasa koja proširi `KEY_ALIASES` mijenja **samo
   svoje** ponašanje.
2. `_items(data, key)` vraća popis rječnika ili praznu listu — jedno mjesto koje pretvara
   odsutan/`null` popis u prazan.
3. EV02 — `prune_all` prolazi čvorove, zove `pruned` na svakom, zadržava preživjele i unira
   pronađene `id`-eve. Petlja postoji **jednom**, za `Control` i `Group`.
4. EV03 — klasa koja ne ispuni ugovor ne može se instancirati; provjeru radi ABCMeta, ne
   dogovor.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| podklasa bez `from_dict` | `TypeError: Can't instantiate abstract class … with abstract method from_dict` |
| element bez `accept` | isto, za `accept` |
| selektabilni čvor bez `pruned` | isto, za `pruned` |
| nepoznat ključ u dokumentu | normalizira se i tiho odbacuje (konstruktor uzima samo svoja polja) |
| podklasa proširi `KEY_ALIASES` | baza i sestrinske klase ostaju netaknute |

## 7. Rezultat

Deserijalizacija, identitet, obilazak i izbor imaju po jedno mjesto. Nova OSCAL klasa nasljeđuje
ugovor umjesto da ga prepiše, a propust se vidi pri instanciranju, ne u produkciji.

## 8. Kriteriji prihvaćanja

1. `ModelBase` je apstraktna; `from_dict` je apstraktni `classmethod`. ✅
2. `_normalise_key` i `_normalise` su `@classmethod` (referiraju `cls`), `_items` je
   `@staticmethod` (ne referira ništa). ✅
3. Podklasa proširuje `KEY_ALIASES` bez učinka na bazu i braću. ✅
4. `OSCALElement` traži `accept`; `SelectableElement` uz to `pruned`. ✅
5. Selektabilni su točno `Control` i `Group`. ✅
6. `name` je izveden iz tipa, nije pohranjeno stanje. ✅
7. U `models.py` nema nijedne funkcije na razini modula. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `inspect.isabstract`, `__abstractmethods__` | `True`; `from_dict` apstraktan |
| 2 | `inspect.ismethod` / `getattr_static` | dva `classmethod`, jedan `staticmethod` |
| 3 | podklasa `Prop` s dodanim aliasom | `the-ns` → `ns` radi u podklasi, u bazi ostaje `None` |
| 4 | `__abstractmethods__` baza | `OSCALElement`: `accept`, `from_dict`; `SelectableElement`: + `pruned` |
| 5 | `issubclass(..., SelectableElement)` | `Control`, `Group`; **ne** `Catalog`, `Profile` |
| 6 | `Catalog(...).name`, `Control(...).name` | `"Catalog"`, `"Control"` |
| 7 | pretraga modula za funkcijama | nijedna |
| §1 t.3 | `@dataclass(frozen=True)` nad podklasom `Wattleflow` s poljima `name`, `value` | `TypeError: non-default argument 'value' follows default argument` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–7 i §1 t.3; platforma =
Python 3.11.15, Linux (WSL2), radno stablo `oscal` 2026-08-21.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4).

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFR-ORG-05` | ovaj zapis je izravna provedba: samoreferencirajuće pomoćne metode su `@classmethod` s `cls`, nikad tvrdo kodirano ime klase |
| `NFR-ORG-08` | jedno pravilo — jedno mjesto: mapiranje ključeva, praznjenje popisa, prolaz kroz čvorove |
| `NFR-SEC-02` | pomoćne metode su privatne (`_`), javna površina modula je `__all__` |
| `NFR-SEC-03` | baze uvoze samo `abc`, `dataclasses`, `typing` i `wattleflow.core` |

## 11. Otvoreno

1. **Vrijednosni objekti nemaju core ugovor.** `Prop`, `Link`, `Part`, `Param`, `Metadata`,
   `BackMatter` nasljeđuju samo `ModelBase(ABC)`, koji nije core sučelje — u `wattleflow.core`
   ne postoji pattern za nepromjenjiv vrijednosni objekt. `CLAUDE.md` §2.5 traži da svaka klasa
   naslijedi odgovarajući pattern; ovdje ga nema. Dva puta, oba `DR-COR` razine: (a) modeli
   prestaju biti `frozen` i lančaju na `Wattleflow` — gubi se nepromjenjivost; (b) u core ulazi
   ugovor za vrijednosni objekt bez `__init__` stanja i bez `name` kolizije.
2. **`ModelBase` nije izvezen iz paketa** (`wattleflow.oscal.__init__` ga ne navodi), iako je u
   `models.__all__`. Vanjska specijalizacija modela zato mora uvoziti iz podmodula.
3. **Ugovor `name` je zrcaljen, ne naslijeđen** (§1 t.3), pa ga ništa ne provodi: `IWattleflow`
   ulazi u MRO tek kroz `IElement`, dakle samo za elemente. Klasa koja naslijedi `ModelBase` i
   zaboravi `name` nije prekršaj koji itko prijavljuje. Kandidat: provjera u lintu ili ugovor iz
   t.1.
