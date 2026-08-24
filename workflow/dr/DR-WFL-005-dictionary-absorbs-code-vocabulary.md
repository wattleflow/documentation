# DR-WFL-005 — Rječnik apsorbira vokabular koda; `naming_registry.yaml` se ukida

| | |
|---|---|
| **Status** | **Prihvaćen** (2026-07-28) |
| **Datum** | 2026-07-28 |
| **Verzija** | 1 (2026-07-28) |
| **Realizira** | NFR-ORG-02, NFR-ORG-03 (dijele registar), `METHODOLOGY.md` §1 t.2 (verzionirani kriterij) |
| **Mijenja** | `dictionary.yaml` (registry_version 0.1.0 → 0.2.0), `tools/wem_lint.py` (1.3.0 → 1.4.0) |
| **Ukida** | `tools/naming_registry.yaml` |

## Kontekst

Vokabular je stajao u dvije datoteke koje su se same proglašavale komplementarnima:
`dictionary.yaml` (diskurs — natuknice, kratice doktrine) i `naming_registry.yaml`
(kod — domene, obitelji baza, uloge TypeVarova). Rječnik je u zaglavlju izrijekom
tvrdio *„kod → naming_registry.yaml"*.

Dvije posljedice:

1. **Akronimi na dva mjesta.** Oba registra imali su ključ `acronyms`, s različitim
   sadržajem i različitim značenjem. Isti znak, dvije uloge — točno ono što natuknica
   `semanticka-entropija` opisuje kao mjeru koju rječnik postoji da smanji.
2. **Kriterij bez jednog izvora.** `wem_lint` je čitao jedan registar, a doktrinarni
   dokumenti citirali drugi; nijedan nije bio nadređen.

## Odluka

Vokabular koda seli u blok **`code:`** rječnika. `naming_registry.yaml` se ukida.

Dva pod-pravila koja odluka fiksira:

**(a) Akronimi ostaju dva odvojena ključa.**

| ključ | sadržaj | čitatelj |
|---|---|---|
| `acronyms` (vrh) | kratice **doktrine** — DQI, NFR, PDSA, SBVR; nose `expansion` i reference | ljudi |
| `code.identifier_acronyms` | tokeni u **identifikatorima** — JSON, URI, SQL, OSCAL; nose samo pravilo pisanja | lint |

Presjek je **prazan** (mjereno 2026-07-28: 9 ∩ 27 = ∅) i mora ostati prazan. Spajanje
bi značilo da lint provjerava imena klasa prema „DQI"/„PDSA", a jedan ključ bi nosio
dvije uloge — spajanje registara ne smije spojiti vokabulare.

**(b) Verzioniranje je razdvojeno.**

* `registry_version` — verzija **rječnika** (diskurs)
* `code.criterion_version` — verzija **kriterija koda**

Izmjena natuknice ne smije obezvrijediti C-snimke konformnosti koda, ni obrnuto.
Trojka reproducibilnosti imenuje obje: `dictionary.code 0.3.0 (dictionary 0.2.0)`.

## Ugovor

**Lomi se:**

* zadana `--registry` putanja i **oblik** datoteke; `wem_lint` < 1.4.0 ne čita novi
  format, `wem_lint` ≥ 1.4.0 ne čita stari (odbija s uputom)
* svaka referenca na putanju `tools/naming_registry.yaml` — čitati kao
  `dictionary.yaml#code`

**Ne lomi se:** nijedan ključ ni vrijednost. Provjera ekvivalencije prije brisanja:
**11 od 11 ključeva preneseno**, jedine razlike su dvije namjerne
(`criterion_version` 0.2.0 → 0.3.0, `python_reference` 3.12 → 3.11 radi usklađenja s
`requires-python`).

## Cijena

`wem_lint` dobiva **cross-repo ovisnost**: rječnik živi u dokumentacijskom repozitoriju
i doseže se preko `tools/dictionary.yaml` → `../documentation/dictionary.yaml`. Sdist
kopira **sadržaj** (23 778 B), ne link, pa je distribucija samodostatna; razvojno stablo
bez dokumentacijskog repoa nema lint. Prihvaćeno: jedan izvor istine vrijedi više od
neovisnosti alata o dokumentaciji.

Kognitivna cijena negativna — jedan `--registry`, jedan blok, jedan verzijski par.

## Svjedočanstvo

* Empirijski: provjera ekvivalencije 11/11 ključeva; presjek akronimskih skupova ∅.
* Empirijski: lint pokrenut nakon migracije daje isti vektor kao prije
  (`ORG-01 misfiled-leaf ERROR 2`, `single-consumer-helper WARNING 4`).
* Deklaracija: `governance.change_mechanism: DR` u samom rječniku traži ovaj zapis.

## Registar

`dictionary.yaml`:
* zaglavlje, `related_registries` i natuknica `registar-imenovanja` ispravljeni — sva
  tri su pokazivala na ukinutu datoteku
* `registry_version` 0.1.0 → 0.2.0
* novi blok `code:` (`criterion_version`, `python_reference`, `blind_spots`, `rules`,
  `domains`, `shared_namespace`, `scope`, `bases`, `prohibited_standalone`,
  `identifier_acronyms`, `type_vars`)

`MANIFEST.in`: `include tools/naming_registry.yaml` → `include tools/dictionary.yaml`.

## Otvoreno

`code.domains` navodi `constants` kao domenu, a taj paket ne uvozi ništa iz wattleflowa
— čisti list. Lint ga zato prijavljuje kao 2 × `ORG-01 misfiled-leaf ERROR`. To je
greška **registra**, ne koda, i traži zaseban DR (uklanjanje `constants` iz `domains`).

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-28 | prvi zapis |

> **Izmijenjeno (2026-08-24).** `DR-WFL-020` vraća vokabular koda iz bloka `code:` u
> vlastiti kriterij po distribuciji (`tools/dictionary.json`); blok je uklonjen iz
> `dictionary.yaml`. Argument protiv **dvaju izvora istine** ostaje na snazi i vodi
> upravo na taj ishod: kopija koju lint ne čita zastarjela je do kriterija 0.8.0.
> Ukidanje `naming_registry.yaml` (t.2) ostaje netaknuto.
