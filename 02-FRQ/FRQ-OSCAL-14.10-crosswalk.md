# FR-OSCAL-14.10 — Prijevod taksonomije kontrola (crosswalk)

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../workflow/dr/DR-WFL-013-oscal-requirement-category.md) (kategorija); **mapiranja čekaju compliance sign-off** (§1) |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Crosswalk` + artefakt `resources/crosswalk/nist-sp800-53_to_asd-ism.json` |
| **Sestrinski** | [`14.11`](FRQ-OSCAL-14.11-policy.md) policy — jedini potrošač |
| **Izvedba** | `src/wattleflow/oscal/crosswalk.py` |

## 1. Predmet

Komponenta može deklarirati kontrole u jednoj taksonomiji (NIST SP 800-53), dok aktivni baseline
govori drugom (ASD ISM). Crosswalk je **mehanizam prijevoda**, i ništa više: tablica je kurirani
compliance artefakt koji ovaj kod primjenjuje, ne stvara.

> **Status mapiranja: `proposed-requires-compliance-review`** (polje `status` u artefaktu, stanje
> 2026-05-31). Do sign-offa nijedan prijevod nije autoritativan; ovaj zapis dokumentira mehanizam,
> ne potvrđuje tablicu. Mapiranja se **ne izmišljaju** (`CLAUDE.md`, worklist „OSCAL crosswalk").

Ključno svojstvo: **nemapirani `id` prolazi nepromijenjen**. Komponenta koja već govori jezikom
baselinea ne treba crosswalk, a neprevediv `id` izlazi kao **prekršaj politike**, ne kao tiho
nestali zahtjev.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Artefakt `nist-sp800-53_to_asd-ism.json` | `name`, `mappings`, `status`, `provenance` |
| **A2** | `Crosswalk` — predmet | tablica u memoriji |
| **A3** | `OSCALPolicy` | deklarirani `id`-evi komponente |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A3 (ili bootstrap) gradi crosswalk — `from_file` / `from_dict` / konstruktor |
| **EV02** | A3 provjerava komponentu → `translate(declared)` |

## 4. Preduvjeti

1. Artefakt postoji i valjan je JSON s ključem `mappings` (izostanak daje **prazan** crosswalk,
   ne grešku).
2. Ciljni `id`-evi mapiranja pripadaju taksonomiji aktivnog baselinea.

## 5. Normalan tok

1. EV01 — svaki izvorni ključ se pohranjuje **malim slovima**, a odredište kao `frozenset`.
   `name` i `mappings` su jedino što se čita; `status`, `provenance`, `description` se **ne
   učitavaju** — postoje za ljudskog recenzenta.
2. EV02 — `translate` za svaki `id` traži mapiranje neosjetljivo na velika/mala slova:
   - postoji → u rezultat ulaze **svi** ciljni `id`-evi (1:n je konjunkcija: komponenta tvrdi da
     zadovoljava sve);
   - ne postoji → `id` ulazi nepromijenjen.
3. Rezultat je skup — duplikati nestaju, redoslijed nije zajamčen.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| datoteka ne postoji | `FileNotFoundError: Crosswalk JSON not found: <putanja>` |
| bez `mappings` | prazan crosswalk; `is_empty` je `True`, `translate` propušta sve |
| `id` različitog slovnog oblika (`AC-3`) | mapira se; usporedba je neosjetljiva |
| `id` bez mapiranja | prolazi nepromijenjen → past će na policyju ako nije u baselineu |
| mapiranje 1:n | **svi** ciljevi moraju biti u baselineu da `declared ⊆ baseline` vrijedi |
| prazan ulaz | prazan izlaz |

## 7. Rezultat

Deklaracija komponente izražena je jezikom baselinea, ili je ostala nepreveden `id` koji policy
odbija. Nema trećeg ishoda — prijevod ništa ne guta.

## 8. Kriteriji prihvaćanja

1. `from_file` učitava `name` i `mappings`; nedostatak datoteke je greška. ✅
2. Usporedba izvornog `id`-a je neosjetljiva na velika/mala slova. ✅
3. Nemapirani `id` prolazi nepromijenjen. ✅
4. Mapiranje 1:n daje sve ciljeve. ✅
5. Prazan crosswalk je valjano stanje (`is_empty`), a ne greška. ✅
6. Polja `status`, `provenance` i `description` ne utječu na ponašanje. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `from_file` nad isporučenim artefaktom | `name = nist-sp800-53_to_asd-ism`, 4 mapiranja |
| 1 | nepostojeća putanja | `FileNotFoundError: Crosswalk JSON not found: …/nope.json` |
| 2 | `"AC-3" in cw`; `translate(["ac-3","AC-3"])` | `True`; oba daju `ism-0445` |
| 3 | `translate(["unmapped"])` | `{"unmapped"}` |
| 4 | pregled tablice | sva četiri mapiranja su 1:1 (`ac-3→ism-0445`, `ia-5→ism-1401`, `sc-8→ism-0469`, `sc-13→ism-1080`); grana 1:n **nije** provjerena podacima |
| 5 | `Crosswalk()` | `is_empty = True`, `translate(["x"]) = {"x"}` |
| 6 | učitani objekt | `status` iz artefakta je `proposed-requires-compliance-review`; objekt ga ne izlaže |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakt `nist-sp800-53_to_asd-ism.json`.

> **Slijepa pjega (D-11), dvije.** (a) Skripta nije u repozitoriju (`CLAUDE.md` §4). (b) Grana
> 1:n je **neprovjerena podacima** — isporučena tablica nema nijedno takvo mapiranje; provjeren je
> samo kod.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFR-SEC-05` | prijevod ne smije skrivati prekršaj: nemapirani `id` pada, ne nestaje |
| `NFR-SEC-02` | `__slots__ = ("_name", "_map")`; tablica se nakon konstrukcije ne mijenja |
| `NFR-SEC-03` | samo `json` i `pathlib` iz stdlib-a |

## 11. Otvoreno

1. **Mapiranja nemaju compliance sign-off** (`status: proposed-requires-compliance-review`).
   `sc-8 → ism-0469` i `sc-13 → ism-1080` su **izvan Essential Eight opsega**, pa protiv E8
   baselinea ispravno padaju — što je lako pročitati kao kvar mehanizma. Do sign-offa se nijedan
   prijevod ne smije koristiti kao dokaz usklađenosti.
2. **`Crosswalk` nije `ModelBase`** iako ima `from_dict`. Namjerno — nije OSCAL model — ali dva
   `from_dict` ugovora u istom paketu, jedan pod bazom a drugi ne, traže zapis da se razlika ne
   čita kao propust.
3. **Nema provjere da su ciljni `id`-evi u baselineu.** Tablica koja mapira u kontrolu koje nema
   u aktivnom profilu proizvest će prekršaj koji izgleda kao krivnja komponente.
