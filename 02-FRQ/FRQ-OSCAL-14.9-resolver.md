# FRQ-OSCAL-14.9 — Razrješavanje profila u katalog

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `resolve(profile, source, *, catalog_uuid=None, strict=True) -> Catalog` |
| **Sestrinski** | [`14.4`](FRQ-OSCAL-14.4-profile.md) profil · [`14.2`](FRQ-OSCAL-14.2-control.md) kontrola · [`14.3`](FRQ-OSCAL-14.3-group.md) grupa |
| **Izvedba** | `src/wattleflow/oscal/resolver.py` |

## 1. Predmet

Razrješavanje pretvara **selektor u katalog**: profil kaže koje `id`-eve hoće, izvorni katalog
kaže što ti `id`-evi jesu, a rezultat je novi katalog koji sadrži samo odabrano — s očuvanom
hijerarhijom grupa (`as-is` spajanje) i bez praznih grana.

Podjela odgovornosti je namjerna i vrijedi je zapisati: **orezivanje pripada čvorovima**
(`Control.pruned`, `Group.pruned`), a resolveru pripada profilna strana — koji su `id`-evi
odabrani, koliko strogo se sudi nedostatku, i od čega se sastavlja rezultat.

Neprovedeno **namjerno**: `modify` (alters/sets), uzorci, parametarske preinake i `include-all`
— ASD ISM profili ih ne koriste (`FRQ-OSCAL-14.4` §11 t.1).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Pozivatelj | profil, izvorni katalog, `strict`, `catalog_uuid` |
| **A2** | `Profile` | `control_ids()`, `excluded_control_ids()`, `metadata`, `back_matter` |
| **A3** | `Catalog` (izvor) | `controls`, `groups`, `params`, `back_matter` |
| **A4** | `Control` / `Group` | `prune_all` / `pruned` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 pozove `resolve(profile, source)` |
| **EV02** | resolver traži izbor od A2 |
| **EV03** | resolver orezuje stablo kroz A4 |
| **EV04** | resolver sastavlja rezultat |

## 4. Preduvjeti

1. Profil bira **barem jedan** `id`.
2. Izvorni katalog odgovara profilu — provjeru `imports[*].href` resolver **ne radi**
   (`FRQ-OSCAL-14.4` §11 t.2).

## 5. Normalan tok

1. EV02 — odabrani `id`-evi su skup iz `control_ids()`; isključeni se oduzimaju
   (`excluded_control_ids()`). Isključenje **pobjeđuje** nad uključenjem.
2. EV03 — `Control.prune_all` nad kontrolama izvora, `Group.prune_all` nad grupama. Svaki poziv
   vraća preživjele čvorove **i** skup stvarno pronađenih `id`-eva.
3. Ako je `strict` (zadano), razlika traženo − pronađeno mora biti prazna; inače `KeyError` s
   brojem nedostajućih i prvih deset `id`-eva.
4. EV04 — rezultat je novi `Catalog`:

   | polje | odakle |
   |---|---|
   | `uuid` | `catalog_uuid`, inače novi UUID4 |
   | `metadata` | **iz profila** — rezultat nosi identitet baselinea, ne izvora |
   | `params` | iz izvora |
   | `controls`, `groups` | preživjeli čvorovi |
   | `back_matter` | iz profila ako postoji, inače iz izvora |

5. Izvorni katalog ostaje **netaknut**; rezultat je zaseban objekt.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| profil ne bira ništa | `ValueError: Profile <uuid> selects no controls` |
| `id` iz profila nema u izvoru, `strict=True` | `KeyError` s brojem i uzorkom nedostajućih |
| isto, `strict=False` | rezultat sadrži samo pronađeno; **nema traga o izgubljenom** (§11 t.1) |
| `id` i uključen i isključen | isključen |
| grupa ostane prazna | nestaje iz rezultata |
| profil bez `back_matter` | preuzima se izvorov |
| pozvan s pogrešnim izvorom | tiho pogrešan rezultat ili `KeyError`, ovisno o preklapanju |

## 7. Rezultat

Baseline postaje katalog koji se dalje ponaša kao svaki drugi: može se registrirati, obići,
ponovno razriješiti. ASD E8 ML1 daje 46 kontrola u 3 grupe iz izvora od 1130 kontrola u 564 grupe.

## 8. Kriteriji prihvaćanja

1. Rezultat sadrži točno odabrane kontrole, minus isključene. ✅
2. Hijerarhija grupa je očuvana; prazne grupe nestaju. ✅
3. `strict=True` prijavljuje nedostajuće `id`-eve; `strict=False` nastavlja. ✅
4. Prazan izbor je greška, ne prazan katalog. ✅
5. `metadata` dolazi iz profila, `params` iz izvora, `back_matter` iz profila s uzmakom na izvor. ✅
6. Izvorni katalog ostaje nepromijenjen. ✅
7. Bez `catalog_uuid` rezultat dobiva novi UUID4. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | ML1 profil nad ISM katalogom | 46 kontrola — jednako `len(profile.control_ids())` |
| 1 | sintetički profil s isključenjem | `["keep"]`, `"drop"` uklonjen |
| 2 | isti poziv | 3 grupe od 564; ML2 i ML3 daju 87 i 123 kontrole |
| 3 | profil koji traži `ghost` | `KeyError: Profile p2 references 1 control(s) absent from source catalog s: ['ghost']`; uz `strict=False` → `["keep"]` |
| 4 | profil s praznim `with_ids` | `ValueError: Profile p3 selects no controls` |
| 5 | ML1 rezultat | `metadata is profile.metadata`, `params == source.params`, `back_matter is profile.back_matter` |
| 6 | izvor nakon razrješavanja | 1130 kontrola |
| 7 | poziv bez `catalog_uuid` | UUID duljine 36 |
| ekvivalencija | **diferencijalni test** protiv prethodne izvedbe orezivanja | 281 nasumično stablo (sjeme 0–299), struktura i skup pronađenih `id`-eva identični u svakom slučaju |

**Trojka (D-10):** alat = `fr_evidence.py` i `prune_oracle.py`; kriterij = §8 t.1–7 i strukturna
jednakost s prethodnom izvedbom; platforma = Python 3.11.15, Linux (WSL2), radno stablo `oscal`
2026-08-21, artefakti ASD ISM E8 ML1–ML3 (izdanje 2026-03-24).

> **Slijepa pjega (D-11).** Nijedna od dvije skripte nije u repozitoriju (`CLAUDE.md` §4).
> Diferencijalni test dokazuje **jednakost s prethodnom izvedbom**, ne ispravnost prema OSCAL
> specifikaciji — obje bi mogle biti jednako krive.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-ORG-05` | resolver ne drži nijednu pomoćnu funkciju modula; orezivanje je na čvorovima |
| `NFRQ-ORG-08` | pravilo preživljavanja postoji jednom — resolver ga ne prepisuje |
| `NFRQ-SEC-01` | rezultat je nov objekt; kvar u razrješavanju ne može oštetiti izvorni katalog |
| `NFRQ-SEC-02` | javna površina modula je jedno ime (`resolve`) |

## 11. Otvoreno

1. **`strict=False` gubi informaciju bez traga.** Nedostajući `id`-evi ne završe ni u rezultatu
   ni u zapisu; pozivatelj ne može saznati što je izgubljeno osim ponovnom usporedbom. Kandidat:
   vratiti ih uz rezultat ili zapisati.
2. **Rezultat ne bilježi podrijetlo.** Razriješeni katalog ne nosi trag koji ga profil i koji
   izvor tvore; `metadata` je profilova, ali izvor nestaje. Za audit trag to je rupa.
3. **`href` se ne provjerava** (`FRQ-OSCAL-14.4` §11 t.2) — razrješavanje protiv pogrešnog kataloga
   prolazi kad se `id`-evi slučajno preklope.
