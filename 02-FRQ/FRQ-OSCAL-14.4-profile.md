# FRQ-OSCAL-14.4 — Profil (selektor baselinea)

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `Profile(OSCALElement)` te njegovi dijelovi `Import`, `IncludeControls`, `Merge` |
| **Sestrinski** | [`14.9`](FRQ-OSCAL-14.9-resolver.md) resolver · [`14.11`](FRQ-OSCAL-14.11-policy.md) policy · [`14.1`](FRQ-OSCAL-14.1-catalog.md) katalog |
| **Izvedba** | `src/wattleflow/oscal/models.py`; artefakti `resources/asd/ISM_E8_ML{1,2,3}-baseline_profile.json` |

## 1. Predmet

Profil je **popis onoga što vrijedi**, ne popis kontrola: ne sadrži nijednu kontrolu, nego ih
navodi po `id`-u iz izvornog kataloga. U ASD ISM izdanju to su tri baselinea — Essential Eight
Maturity Level 1, 2 i 3.

| dio | uloga |
|---|---|
| `Import` | jedan izvor (`href`) i njegovi selektori |
| `IncludeControls` | `with_ids` — što ulazi; ASD ISM koristi isključivo ovaj oblik |
| `Merge` | kako se spaja (`as-is` u ISM profilima) |
| `modify` | preinake — **prenosi se kao sirovi rječnik i ništa ga ne čita** |

Profil **ne razrješava sam sebe**: pretvaranje u katalog je posao resolvera (`FRQ-OSCAL-14.9`), a
provjera komponente posao policyja (`FRQ-OSCAL-14.11`).

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Artefakt `ISM_E8_ML*-baseline_profile.json` | OSCAL profil dokument |
| **A2** | `ASDOSCALProfileLoader` | putanja |
| **A3** | `Profile` — predmet | selektori |
| **A4** | `resolve()` | traži odabrane i isključene `id`-eve |
| **A5** | `OSCALPolicy` | uzima `control_ids()` kao baseline |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | A2 pročita dokument → `Profile.from_dict(payload)` |
| **EV02** | A4 razrješava → `control_ids()` + `excluded_control_ids()` |
| **EV03** | A5 gradi gate → `control_ids()` |
| **EV04** | posjetitelj → `accept(visitor)` |

## 4. Preduvjeti

1. Dokument je OSCAL **profil** — omotan (`{"profile": {…}}`) ili gol; katalog odbija loader.
2. `uuid` neprazan i **barem jedan** `import`; profil bez izvora nije profil.
3. `metadata` prisutna (isto kao katalog).

## 5. Normalan tok

1. EV01 — odmotavanje, normalizacija ključeva, izgradnja `imports`, `merge`, `back_matter`;
   `modify` ostaje sirovi rječnik.
2. `__post_init__` odbija prazan `uuid`, pa profil bez `imports` (poruka nosi `uuid`).
3. EV02 — `control_ids()` spljošti `imports[*].include_controls[*].with_ids` **redom pojave**,
   bez uklanjanja duplikata; `excluded_control_ids()` isto za `exclude_controls`.
4. EV03 — A5 od tog popisa gradi `frozenset` baselinea.
5. EV04 — `accept` posjeti **samo profil**; `imports` nisu elementi i ne obilaze se.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| `uuid` prazan | `ValueError: Profile.uuid must not be empty` |
| bez `imports` | `ValueError: Profile <uuid> has no imports` |
| dokument je katalog | odbija ga loader prije `from_dict` |
| `include_controls` prazan | profil se konstruira; prazan izbor pada tek u resolveru (`FRQ-OSCAL-14.9` §6) |
| isti `id` u dva importa | duplikat ostaje u popisu; posljedice nosi potrošač (skup ga svede) |
| `id` i u `include` i u `exclude` | isključenje pobjeđuje — razlika skupova u resolveru |
| `modify` prisutan | prenosi se, ne primjenjuje; **tiho neprovedena namjera** (§11 t.1) |

## 7. Rezultat

Baseline je izražen kao skup `id`-eva, odvojen od kataloga u kojem ti `id`-evi žive. Ista tri ASD
profila hrane i resolver i policy, bez ijedne kopije popisa kontrola.

## 8. Kriteriji prihvaćanja

1. `from_dict` prihvaća omotani i goli oblik. ✅
2. Prazan `uuid` i izostanak `imports` zaustavljaju konstrukciju, s `uuid` u poruci. ✅
3. `control_ids()` spljošti sve importe redom pojave. ✅
4. `excluded_control_ids()` postoji simetrično i vraća prazan popis kad isključenja nema. ✅
5. `merge` i `back_matter` se grade, `modify` prenosi neizmijenjen. ✅
6. `accept` posjeti samo profil. ✅

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | `Profile.from_dict({"profile": {…}})` | konstruiran, `uuid = p` |
| 2 | negativni slučajevi | `Profile.uuid must not be empty`; `Profile p has no imports` |
| 3 | ML1 profil | 46 `id`-eva iz 1 importa (ML2: 87, ML3: 123 — `FRQ-OSCAL-14.9` §9) |
| 4 | ML1 profil; sintetički import s isključenjem | `[]`; `["b"]` isključen iz `["a","b"]` |
| 5 | ML1 profil | `Merge(as_is=True, …)`, `modify` je `None`, `back_matter` prisutan |
| 6 | posjetitelj koji bilježi tipove | `["Profile"]` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakti ASD ISM E8 ML1–ML3 (izdanje 2026-03-24).

> **Slijepa pjega (D-11).** Skripta nije u repozitoriju (`CLAUDE.md` §4). Oblici selektora koje
> ASD ISM ne koristi (`matching`, `with_child_controls`, `include_all`) provjereni su samo
> sintetički: model ih **prihvaća i pohranjuje**, ali ih nijedan potrošač ne čita.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-ORG-08` | popis odabranih i isključenih `id`-eva izvodi se na jednom mjestu (`Import` → `Profile`), a ne u resolveru |
| `NFRQ-SEC-02` | profil ne izlaže mutatore; baseline se ne može proširiti nakon konstrukcije |
| `NFRQ-SEC-01` | baseline je vezan uz `uuid` profila; poruka o prekršaju imenuje profil, pa se opseg kvara vidi |

## 11. Otvoreno

1. **`modify`, `matching`, `with_child_controls` i `include_all` se prihvaćaju, ali se ne
   primjenjuju.** Profil koji ih koristi bit će **tiho** krivo razriješen — model ne prijavlja da
   je namjeru ignorirao. ASD ISM ih ne koristi, pa danas nema posljedice. Kandidat: odbiti
   dokument koji ih sadrži, ili ih zapisati kao nepodržane u rezultatu. Traži odluku.
2. **`href` se ne provjerava.** Resolver prima izvorni katalog od pozivatelja i **ne uspoređuje**
   ga s `imports[*].href` (`resolve()` docstring to izrijekom prepušta pozivatelju). Profil se
   time može razriješiti protiv pogrešnog kataloga.
3. **Duplikati u `with_ids`** prolaze do potrošača; benigno dok su potrošači skupovi.
