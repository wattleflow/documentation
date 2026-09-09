# FRQ-OSCAL-14.2 — Kontrola

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Control(SelectableElement)` — jedinica kontrole koju profil bira, registar indeksira, a policy provjerava |
| **Sestrinski** | [`14.1`](FRQ-OSCAL-14.1-catalog.md) katalog · [`14.3`](FRQ-OSCAL-14.3-group.md) grupa · [`14.6`](FRQ-OSCAL-14.6-bases.md) baze |
| **Izvedba** | `src/wattleflow/oscal/models.py` |

## 1. Predmet

Kontrola je **jedini čvor koji nosi identitet po kojem se sudi**: `id` je ono što profil navodi,
registar indeksira, a policy uspoređuje s baselineom. Sve ostalo u modelu postoji da bi kontrola
imala mjesto i sadržaj.

| polje | obveza | uloga |
|---|---|---|
| `id` | obvezno, neprazno | identitet u svim ostalim zahtjevima |
| `title` | obvezno, neprazno | ljudski čitljiv naziv u izvještaju |
| `class_` | opcionalno | OSCAL `class`, npr. `ISM-principle` |
| `params`, `props`, `links`, `parts` | popisi, prazni ako izostanu | sadržaj kontrole |
| `controls` | popis | ugniježđene podkontrole (OSCAL dopušta stablo) |

Kontrola **ne zna** je li odabrana ni u kojem je baselineu — to znanje živi u profilu i policyju.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `Catalog` / `Group` — vlasnik popisa | strukturni kontekst |
| **A2** | `resolve()` | skup odabranih `id`-eva |
| **A3** | `OSCALCatalogRegistry` | ravni indeks po `id` |
| **A4** | Posjetitelj (`IVisitor`) | obilazak |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | roditelj gradi stablo → `Control.from_dict(payload)` |
| **EV02** | A2 orezuje stablo → `pruned(selected)` |
| **EV03** | A3 indeksira → `walk_controls()` preko roditelja |
| **EV04** | A4 obilazi → `accept(visitor)` |

## 4. Preduvjeti

1. `id` i `title` postoje u dokumentu i nisu prazni — provjera je u `__post_init__`, dakle prije
   nego objekt postoji.
2. Ugniježđene kontrole zadovoljavaju ista pravila (rekurzija kroz isti `from_dict`).

## 5. Normalan tok

1. EV01 — normalizacija ključeva, pa rekurzivna izgradnja `params`/`props`/`links`/`parts` i
   ugniježđenih kontrola. Odsutan popis daje praznu listu.
2. `__post_init__` odbija praznu vrijednost: prvo `id`, pa `title` (poruka nosi `id`, da se
   pogrešna kontrola može naći u dokumentu).
3. EV02 — `pruned(selected)` vraća **par** (čvor, pronađeni `id`-evi):
   - kontrola je odabrana → preživljava, djeca koja nisu odabrana **otpadaju**;
   - kontrola nije odabrana ali potomak jest → preživljava kao **omotač**, i **ne** ulazi u skup
     pronađenih (nije ju profil tražio);
   - ništa nije odabrano → `(None, ∅)`.
4. Kad sva djeca prežive, vraća se **ista instanca** — bez suvišne kopije.
5. EV03 — `walk_controls()` daje dubinski niz potomaka, bez same kontrole.
6. EV04 — `accept` posjeti sebe, pa svako dijete rekurzivno.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `id` prazan | `ValueError: Control.id must not be empty` |
| `title` prazan | `ValueError: Control <id> has no title` |
| `title` nedostaje u dokumentu | `KeyError: 'title'` |
| izmjena polja | `FrozenInstanceError` |
| `hash(control)` | `TypeError: unhashable type: 'list'` — vidi §11 t.1 |
| dva djeteta s istim `id` | prolazi; kontrola ne provjerava jedinstvenost |

## 7. Rezultat

Stablo kontrola je izgrađeno i nepromjenjivo; orezivanje daje **novo** stablo, a izvorno ostaje
dijeljivo. Neispravna kontrola pada pri konstrukciji, ne pri uporabi.

## 8. Kriteriji prihvaćanja

1. `id` i `title` obvezni i neprazni; poruka za `title` imenuje kontrolu. ✅
2. Odsutni popisi daju prazne liste, nikad `None`. ✅
3. `pruned` zadržava kontrolu kad je odabrana **ili** kad joj je potomak odabran. ✅
4. Odabrana kontrola ulazi u skup pronađenih; puki omotač ne ulazi. ✅
5. Kad ništa ne otpada, vraća se ista instanca (bez kopije). ✅
6. `walk_controls` daje potomke dubinski, bez same kontrole. ✅
7. Instanca je nepromjenjiva. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | negativni slučajevi | `Control.id must not be empty`; `Control x has no title`; `KeyError: 'title'` |
| 2 | učitavanje ISM kataloga | 1130 kontrola; sve nose `props` i `parts`; nijedna nema `params`, `links` ni djecu |
| 3, 4 | sintetičko stablo `parent(child)` | `pruned({"child"}) → ("parent", ["child"], {"child"})`; `pruned({"parent"}) → ("parent", [], {"parent"})`; `pruned({"zzz"}) → (None, ∅)` |
| 5 | `pruned({"parent","child"})[0] is parent` | `True` |
| 6 | `parent.walk_controls()` | `["child"]` |
| 7 | pokušaj pridruživanja | `FrozenInstanceError` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–7; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt ISM 2026-03-24.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (test framework nije odabran,
> `CLAUDE.md` §4). Kriteriji 3–6 provjereni su **sintetičkim** stablima jer isporučeni ISM
> katalog nema nijednu ugniježđenu kontrolu; širu potvrdu daje usporedba s prethodnom izvedbom
> orezivanja nad 281 nasumičnim stablom (`FRQ-OSCAL-14.9` §9).

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-ORG-05` | `pruned` i `walk_controls` su članovi klase; samoreferenca ide kroz `self.prune_all`, ne kroz tvrdo kodirano ime |
| `NFRQ-ORG-08` | pravilo orezivanja postoji na jednom mjestu; `Group` i `resolve()` ga ne prepisuju |
| `NFRQ-SEC-02` | kontrola ne izlaže postavljače ni interne popise za izmjenu |

## 11. Otvoreno

1. **`Control` je `frozen`, ali nije hashabilan** — `hash()` puca na popisima (`TypeError`).
   Nepromjenjiva vrijednost koja ne može u `set` ni biti ključ je proturječje ugovora; danas
   prolazi jer registar indeksira po `id`. Rješenje (`tuple` polja ili `eq=False`) mijenja javni
   tip polja — traži odluku.
2. **Dijelovi modela koje isporučeni ISM nikad ne koristi:** `params`, `links` i ugniježđene
   kontrole imaju 0 pojava. Zadržati radi vjernosti OSCAL modelu ili označiti kao neprovjereno
   područje — otvoreno.
3. **Jedinstvenost `id` unutar jednog kataloga se ne provjerava** (vidi `FRQ-OSCAL-14.1` §11 t.3).
