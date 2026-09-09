# DR-WFL-004 — Pisanje akronima u identifikatorima koda

| | |
|---|---|
| **Status** | **Otvoren** (nije odlučeno) |
| **Datum** | 2026-07-28 (formaliziran; pitanje otvoreno od 2026-07-09) |
| **Verzija** | 1 (2026-07-28) |
| **Prethodna oznaka** | `DR-015` — referenciran u `METHODOLOGY.md` §9, `dictionary.yaml` i `RIJECNIK.md` prije nego je zapis postojao |
| **Realizira** | NFRQ-ORG-02 kriterij 4, NFRQ-ORG-03 kriterij 4 |
| **Provodi** | `tools/wem_lint.py` (ORG-02, ORG-03), preko `dictionary.yaml → acronym_identifier_casing` |

## Kontekst

PEP 8 propisuje da se akronim u imenu piše velikim slovima (`CreatePDFDocument`).
Zatečeni kod nosi miješani oblik (`CreatePdfDocument`). Oba su obranjiva: PEP 8 je
priznati standard i pod-metoda po `METHODOLOGY.md` §8, ali `PDFHTMLParser` je čitljiviji
kao `PdfHtmlParser` kad se akronimi lance.

Pitanje je **otvoreno** i to je jedini razlog zašto ovaj zapis postoji prije odluke:
kriterij je bio referenciran (`DR-015`) iz tri doktrinarna dokumenta a da zapis nije
postojao — referenca na nepostojeći zapis nije sljedivost.

## Odluka

**Nije donesena.** Do odluke vrijedi režim:

* lint prijavljuje odstupanje kao **WARNING s deklariranim waiverom**, nikad ERROR;
* upozorenja su **empirijska građa** za odluku, ne kazna;
* provođenje jedne strane neodlučenog pitanja kršilo bi kaskadu — metoda ne propisuje
  mimo odluke (`METHODOLOGY.md` §9).

Režim je **strojno čitljiv**, ne komentar u kodu:

```yaml
acronym_identifier_casing:
  status: neodluceno          # usvojen -> lint diže na ERROR
  decision_pending: DR-WFL-004
```

`wem_lint` od v1.4.0 čita `status` i iz njega izvodi strogost. Kad odluka padne,
mijenja se registar — alat ostaje netaknut.

## Ugovor

Ne lomi ništa dok je status `neodluceno`. Prelazak na `usvojen` pretvara postojeća
upozorenja u ERROR i ruši build dok se imena ne migriraju (hard rename, NFRQ-ORG-02
implementacijske napomene).

## Cijena

Odgoda nosi trošak: svaka nova klasa dodaje se u zatečenom obliku i povećava opseg
buduće migracije. To je svjesno prihvaćeno — trošak pogrešne odluke veći je od troška
čekanja, jer je migracija imena hard rename bez aliasa.

## Svjedočanstvo

Aspiracija (PEP 8 konformnost) naspram zatečenog stanja. **Nema mjerenja** koliko je
identifikatora zahvaćeno — to je prvi ulaz koji odluka treba, a lint ga proizvodi kao
`ORG-02`/`ORG-03` WARNING vektor.

## Registar

`dictionary.yaml`:
* `acronym_identifier_casing.status` — jedini prekidač strogosti
* `code.identifier_acronyms` — popis tokena na koje se pravilo primjenjuje

Napomena: `code.identifier_acronyms` (JSON, URI, SQL…) **nije** isti vokabular kao
`acronyms` na vrhu rječnika (DQI, NFR, PDSA…). Vidi [DR-WFL-005](DR-WFL-005-dictionary-absorbs-code-vocabulary.md).

## Povijest zapisa

Povijest **zapisa** (artefakta), odvojena od povijesti odluke: odluka je
nepromjenjiva, zapis se revidira (`dictionary.yaml`: `odluka` / `zapis-odluke`).

| v | datum | izmjena |
|---|---|---|
| 1 | 2026-07-28 | prvi zapis; formalizira pitanje referencirano kao `DR-015` iz tri dokumenta a nikad zapisano |
