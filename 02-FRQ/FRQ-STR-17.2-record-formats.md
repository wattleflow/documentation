# FRQ-STR-17.2 — Zapisi u standardnom formatu

| | |
|---|---|
| **Status** | Provedeno u kodu 2026-09-11 — zapis nastao uz kod, ne obrnutim inženjerstvom |
| **Roditelj** | [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) |
| **Odluka** | [`DR-PRC-004`](../04-DR/DR-PRC-004-pipeline-subjects-and-record-formatters.md) (prijedlog) |
| **Norme** | `NFRQ-ORG-08` · `NFRQ-ORG-09` · `NFRQ-SEC-03` · `NFRQ-OBS-01…03` |
| **Standard** | RFC 8259 (JSON); RFC 4180 (CSV); RSS 2.0; XLSX, ORC i Avro kroz vlastite knjižnice |

## 1. Opseg

Zapise koje je pipeline stavio na dokument strana zapisa mora serijalizirati u ciljni standardni
format i predati spremištu. **Ista strategija služi svakom izvoru zapisa** — ne zna je li zapis
došao iz RSS-a ili iz drugog rječnika. Repozitorij uz to mora pohranjeni dokument pročitati natrag
u isti oblik zapisa.

Kako zapisi nastaju nije predmet ovog zahtjeva ([`FRQ-PIP-17.1`](FRQ-PIP-17.1-feed-records.md)).

## 2. Akteri

| oznaka | tko / što | unosi u proces |
|---|---|---|
| **A1** | pipeline koji je stavio zapise i ciljni format ([`FRQ-PIP-17.1`](FRQ-PIP-17.1-feed-records.md)) | zapise i format na dokumentu |
| **A2** | radna ploča i repozitorij — tok perzistencije ([`FRQ-PRC-15.22`](FRQ-PRC-15.22-document-flow.md)) | trenutak pisanja i driver |
| **A3** | obitelj formatera s tvornicom po tipu datoteke | serijalizaciju zapisa |
| **A4** | driver spremišta | pohranu gotovog sadržaja; pri čitanju prepoznavanje tipa i parsirani sadržaj |

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | radna ploča predaje dokument repozitoriju — odmah ili pri pražnjenju |
| **EV02** | repozitorij traži pohranjeni dokument po identifikatoru |

## 4. Preduvjeti

1. Repozitorij ima driver; bez njega pisanje i čitanje su kvar.
2. Za ciljni format postoji registriran formater (`BR-08`).

## 5. Normalan tok — pisanje

| korak | radnja |
|---|---|
| 1 | Dokument stiže (`EV01`) |
| 2 | S dokumenta se čitaju zapisi i ciljni format |
| 3 | Tvornica daje formater za ciljni format |
| 4 | Formater serijalizira: JSON po RFC 8259; CSV po RFC 4180 — zaglavlje je unija polja u redoslijedu prvog pojavljivanja, redak završava s CRLF; XLSX preko tablice; ORC; RSS 2.0 s kanalom; XML kao niz elemenata zapisa |
| 5 | Driver pohranjuje sadržaj pod imenom izvora i sufiksom formata (`BR-09`) |
| 6 | Dokument bilježi izlaz, tko ga je zapisao i kada |

## 6. Normalan tok — čitanje

| korak | radnja |
|---|---|
| 1 | Repozitorij traži dokument po identifikatoru (`EV02`) |
| 2 | Driver prepoznaje tip po sufiksu ili, kad sufiks ne odlučuje, po sadržaju (`BR-02`) i vraća parsirani sadržaj |
| 3 | Sadržaj se provjerava protiv očekivanog oblika i stavlja na dokument s formatom — isti oblik koji daje pipeline |

## 7. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| dokument ne nosi zapise | upozorenje, ništa se ne piše | nema praznog izlaza |
| za format ne postoji formater | kvar pisanja (`BR-08`) | repozitorij ga prijavljuje s imenom strategije |
| Avro bez sheme | kvar pisanja | shema se ne izmišlja |
| pohranjeni dokument nije očekivanog oblika | kvar čitanja s tipom pročitanog sadržaja | tekst se ne predaje kao feed |

## 8. Poslovna pravila

Nasljeđuje `BR-01`, `BR-02`, `BR-08` i `BR-09` iz [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) §4.

## 9. Rezultat

Pohranjena datoteka u ciljnom formatu, a dokument zna gdje je. Pri čitanju: dokument sa sadržajem
u obliku zapisa i njegovim formatom.

## 10. Kriteriji prihvaćanja i verifikacija

| | kriterij | verifikacija |
|---|---|---|
| 1 | Isti dokument daje JSON, CSV i RSS iz jednog prolaza | **izvedeno** 2026-09-11 end-to-end |
| 2 | CSV nosi zaglavlje kao uniju polja i redove s CRLF | **izvedeno** |
| 3 | Postojeći putovi — tekst i tablica — nepromijenjeni | **izvedeno** |
| 4 | Driver ne serijalizira (`BR-09`) | pregled — driver nije mijenjan |
| 5 | Pohranjeni RSS i XML čitaju se po sufiksu ili sadržaju; tuđi sadržaj odbija se kao kvar | **izvedeno** kroz repozitorij s driverom: `.rss`, RSS 1.0 pohranjen kao `.xml`, XML s deklaracijom; tekst, i RSS pod XML strategijom, odbijeni |
| 6 | Dokument bez zapisa ne proizvodi datoteku | pregled — **nije izvedeno** |
| 7 | XLSX, ORC, Avro | **nije izvedeno** — knjižnice nisu instalirane u okolini provjere |

**Testova nema** — deklarirana rupa (`DR-WFL-023`, D-11).

## 11. Dijagram

[`FRQ-STR-17.2-record-formats.puml`](FRQ-STR-17.2-record-formats.puml) — sequence; gledište: pohrana
zapisa jednog dokumenta i čitanje pohranjenog natrag; publika: implementator i tester. Slika nije
generirana. Pogled, ne izvor istine (D-13).

## 12. Otvoreno

- **Ime izlaza je ime izvora bez sufiksa.** Dva izvora istog imena iz različitih direktorija pišu u
  isti izlaz.
- **Ime elementa zapisa ne dopire do strane čitanja**: pohranjeni XML čita se po djeci korijena.
- **Format bira pipeline** ([`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) §6 t.1).
- **Tvornica formatera učitava cijelu obitelj**, pa strategija nosi i ovisnosti tuđih formata
  ([`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) §5).

## 13. Sljedivost prema izvedbi

Za strukturu klasa vrijedi [dekompozicija](../01-HLRQ/HLRQ-17-decomposition.puml).
Izvedba: `blackwattle/src/wattleflow/strategies/documents/{records,rss,xml}.py`,
`helpers/formatters/{text,tabular,factory}.py`, `helpers/records.py`; driver nepromijenjen.
