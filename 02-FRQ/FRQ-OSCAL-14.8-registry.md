# FR-OSCAL-14.8 — Registar kataloga

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `OSCALCatalogRegistry(Wattleflow)` |
| **Sestrinski** | [`14.1`](FRQ-OSCAL-14.1-catalog.md) katalog · [`14.2`](FRQ-OSCAL-14.2-control.md) kontrola |
| **Izvedba** | `src/wattleflow/oscal/registry.py` |

## 1. Predmet

Registar je **ravni pogled na više kataloga**: katalozi su indeksirani po `uuid`, a sve njihove
kontrole — uključujući ugniježđene — po `id`. Bez njega bi svaki potrošač koji traži kontrolu po
`id`-u morao obilaziti stablo.

Registar **namjerno nije `IRepository`**: taj ugovor vraća `ITarget` i pretpostavlja facade-upise,
a ovdje trebaju tipizirani `Catalog` / `Control` pogledi. Obrazloženje stoji u docstringu klase i
ovdje se ne prepričava (D-13).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Pozivatelj (bootstrap workflowa) | učitani katalozi, **redom** |
| **A2** | `OSCALCatalogRegistry` — predmet | indeks |
| **A3** | Potrošač provjere (policy, izvještaj) | `id` kontrole |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A1 zove `register(catalog)` |
| **EV02** | A3 traži `get(id)` / `find(id)` / `id in registry` |
| **EV03** | A3 nabraja `iter_catalogs()` / `iter_controls()` |
| **EV04** | A1 zove `clear()` |

## 4. Preduvjeti

1. Katalog je već izgrađen i valjan (`FR-OSCAL-14.1`).
2. Redoslijed registracije je **odgovornost pozivatelja** — vidi §5 t.2.

## 5. Normalan tok

1. EV01 — `uuid` se upisuje u indeks kataloga; potom se **cijelo stablo** obilazi
   (`create_iterator()`) i svaka kontrola upisuje pod svoj `id`.
2. Sudar `id`-eva među katalozima razrješava se pravilom **„kasniji pobjeđuje"**; pravilo je
   svjesno i zapisano u kodu, a posljedica je da pozivatelj mora registrirati puni ISM prije E8
   baselinea.
3. EV02 — `get` diže `KeyError` s porukom koja imenuje traženi `id`; `find` vraća `None`;
   `in` daje `bool`. Tri odgovora na isto pitanje, za tri različita poziva.
4. EV03 — `iter_catalogs` / `iter_controls` daju iteratore nad **vrijednostima indeksa**, ne
   kopije.
5. EV04 — `clear()` prazni oba indeksa; registrirani katalozi ostaju živi kod svojih vlasnika.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| isti `uuid` dvaput | `ValueError: Catalog already registered: <uuid>` — indeks se ne dira |
| isti `id` u dva kataloga | drugi prepisuje prvog; **bez upozorenja** |
| `get` nepoznatog `id` | `KeyError: 'Unknown OSCAL control: <id>'` |
| `find` nepoznatog `id` | `None` |
| katalog bez kontrola | registrira se; `control_count` ostaje nepromijenjen |
| izmjena kataloga nakon registracije | nemoguća — katalog je nepromjenjiv |

## 7. Rezultat

Provjera „postoji li kontrola `ism-0445`" je pretraga rječnika, neovisna o tome u kojem je
katalogu i koliko duboko kontrola bila.

## 8. Kriteriji prihvaćanja

1. `register` indeksira katalog po `uuid` i **sve** kontrole po `id`. ✅
2. Ponovna registracija istog `uuid`-a je greška. ✅
3. Sudar `id`-eva razrješava se „kasniji pobjeđuje". ✅
4. `get` diže `KeyError` s imenom kontrole; `find` vraća `None`; `in` radi. ✅
5. `len(registry)` je broj kataloga, `control_count` broj kontrola — dvije različite mjere. ✅
6. `clear()` vraća registar u prazno stanje. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | registracija ISM kataloga | `len = 1`, `control_count = 1130`, `iter_controls` daje 1130 |
| 2 | ponovna registracija | `ValueError: Catalog already registered: abea4b79-…` |
| 3 | dva sintetička kataloga s istim `id` | `get("dup").title == "second"`; kataloga 2, kontrola 1 |
| 4 | nepoznati `id` | `KeyError: 'Unknown OSCAL control: nope'`; `find → None`; `in → False` |
| 5 | isti registar | `len = 1` uz `control_count = 1130` |
| 6 | `clear()` | `0`, `0` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt ISM 2026-03-24.

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4).

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFR-SEC-01` | registar je **u memoriji i po procesu**; ne dijeli stanje između procesa, pa nema zajedničke točke kvara |
| `NFR-SEC-02` | indeksi su privatni; javna površina su pogledi, ne rječnici |
| `NFR-SEC-03` | samo stdlib i `wattleflow` |

## 11. Otvoreno

1. **Sudar `id`-eva prolazi bez traga.** Pravilo „kasniji pobjeđuje" ovisi o redoslijedu koji
   nitko ne provjerava; pogrešan redoslijed daje **tiho drukčiji baseline**. Kandidat: zapis
   (`warning`) pri prepisivanju, ili obvezan redoslijed kao dio ugovora.
2. **`register` nije atomičan.** Katalog se upiše prije obilaska kontrola; iznimka usred
   obilaska ostavlja registar djelomično popunjen, a ponovni pokušaj pada na `ValueError`.
3. **`Wattleflow` baza donosi audit sposobnost koja se ne koristi** — registar ne zapisuje
   nijedan događaj, ni registraciju ni sudar. Vidi t.1.
