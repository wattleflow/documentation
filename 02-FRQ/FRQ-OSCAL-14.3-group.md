# FR-OSCAL-14.3 — Grupa kontrola

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Group(SelectableElement)` — imenovani dio kataloga koji drži kontrole i podgrupe |
| **Sestrinski** | [`14.1`](FRQ-OSCAL-14.1-catalog.md) katalog · [`14.2`](FRQ-OSCAL-14.2-control.md) kontrola · [`14.9`](FRQ-OSCAL-14.9-resolver.md) resolver |
| **Izvedba** | `src/wattleflow/oscal/models.py` |

## 1. Predmet

Grupa je **struktura, ne subjekt provjere**: nosi naslov pod kojim ISM organizira kontrole
(„Guidelines for system hardening"), ali nijedan profil ne bira grupu — bira kontrole, a grupa
preživi ako je u njoj ostalo išta.

| polje | obveza | uloga |
|---|---|---|
| `title` | **obvezno** | naslov dijela kataloga |
| `id` | opcionalno | ISM ga ne postavlja na grupama |
| `groups`, `controls` | popisi | djeca |
| `params`, `props`, `links`, `parts` | popisi | sadržaj, u ISM izdanju neiskorišten |

Asimetrija je namjerna i vrijedi je zapisati: `Control.id` je obvezan a `title` također, dok je
`Group.id` neobvezan — jer se grupa ne referira izvana.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `Catalog` / nadređena `Group` | strukturni kontekst |
| **A2** | `resolve()` | skup odabranih `id`-eva kontrola |
| **A3** | Posjetitelj (`IVisitor`) | obilazak |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | roditelj gradi stablo → `Group.from_dict(payload)` |
| **EV02** | A2 orezuje → `pruned(selected)` |
| **EV03** | potrošač traži ravni niz → `walk_controls()` |
| **EV04** | A3 obilazi → `accept(visitor)` |

## 4. Preduvjeti

1. `title` postoji u dokumentu; njegov izostanak je greška, ne prazna vrijednost.
2. Djeca (podgrupe, kontrole) zadovoljavaju vlastite preduvjete (`FR-OSCAL-14.2` §4).

## 5. Normalan tok

1. EV01 — normalizacija ključeva, rekurzivna izgradnja podgrupa i kontrola.
2. EV02 — `pruned(selected)` orezuje **podgrupe pa kontrole**; grupa u kojoj ne preživi ništa
   vraća `(None, ∅)` i nestaje iz rezultata. Grupa koja preživi uvijek je **nova instanca**
   (za razliku od `Control`, koji vraća sebe kad ništa ne otpada).
3. Skup pronađenih `id`-eva je unija onoga što su vratila djeca; grupa sama ne doprinosi
   nijedan `id` — nije subjekt izbora.
4. EV03 — `walk_controls()` daje **kontrole prije podgrupa**, svaku dubinski.
5. EV04 — `accept` posjeti sebe, pa **podgrupe pa kontrole**.

> Poredak u t.4 i t.5 **nije isti** — vidi §11 t.1.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `title` nedostaje | `KeyError: 'title'` |
| `id` izostane | `None`; valjano stanje |
| grupa bez djece | `pruned` je odbacuje; `walk_controls` daje prazan niz |
| grupa čije je jedino dijete prazna podgrupa | odbacuje se, jer preživljavanje ovisi o **preživjeloj** djeci, ne o polaznoj |
| izmjena polja | `FrozenInstanceError` |

## 7. Rezultat

Hijerarhija kataloga preživljava razrješavanje profila (`as-is` spajanje): kontrole zadržavaju
mjesto u kojem su objavljene, a prazne grane nestaju.

## 8. Kriteriji prihvaćanja

1. `title` je obvezan; `id` nije. ✅
2. `pruned` odbacuje grupu bez preživjele djece, a inače vraća novu instancu s orezanom djecom. ✅
3. Grupa ne doprinosi vlastiti `id` skupu pronađenih. ✅
4. `walk_controls` daje kontrole prije podgrupa, dubinski. ✅
5. `accept` posjeti grupu prije djece. ✅
6. Instanca je nepromjenjiva. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `Group.from_dict({"id": "g"})`; `Group(title="t").id` | `KeyError: 'title'`; `None` |
| 2 | `outer(inner(child)).pruned({"child"})` | `("outer", 1 podgrupa, {"child"})`; `Group("e").pruned({"x"}) → (None, ∅)` |
| 3 | isti poziv | skup je `{"child"}` — naslov grupe se ne pojavljuje |
| 4 | grupa s izravnom kontrolom i podgrupom | `["direct", "nested"]` |
| 5 | posjetitelj koji bilježi tipove | `["Group", "Group", "Control"]` |
| 6 | ISM katalog | 25 grupa na vrhu, 564 ukupno, sve nepromjenjive |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt ISM 2026-03-24.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4).

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFR-ORG-05` | `pruned` koristi naslijeđeni `self.prune_all` za podgrupe i `Control.prune_all` za kontrole — bez prepisane petlje |
| `NFR-ORG-08` | pravilo „prazna grana nestaje" postoji na jednom mjestu |

## 11. Otvoreno

1. **Poredak obilaska nije dosljedan.** `Group.accept` posjećuje **podgrupe pa kontrole**, dok
   `Group.walk_controls` daje **kontrole pa podgrupe**; `Catalog` je u oba slučaja
   `kontrole → grupe`. Dva potrošača istog stabla dobivaju različit redoslijed, a nijedan
   dokument ne kaže koji je ispravan. Nalaz zatiče postojeći kod; ispravak je izmjena ponašanja
   i traži odluku.
2. **`params`, `props`, `links`, `parts` na grupi nemaju nijednu pojavu u ISM izdanju** — isto
   pitanje kao `FR-OSCAL-14.2` §11 t.2.
3. **`Group` ne provjerava jedinstvenost naslova ni `id`-a** među braćom.
