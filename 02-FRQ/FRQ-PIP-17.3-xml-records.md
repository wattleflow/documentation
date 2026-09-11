# FRQ-PIP-17.3 — Čitanje XML dokumenta u zapise

| | |
|---|---|
| **Status** | Provedeno u kodu 2026-09-11 — zapis nastao uz kod, ne obrnutim inženjerstvom |
| **Roditelj** | [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) |
| **Odluka** | [`DR-PRC-004`](../04-DR/DR-PRC-004-pipeline-subjects-and-record-formatters.md) (prijedlog) |
| **Norme** | `NFRQ-ORG-02` · `NFRQ-ORG-07` · `NFRQ-ORG-08` · `NFRQ-ORG-09` · `NFRQ-OBS-01…03` |
| **Standard** | XML 1.0; Namespaces in XML 1.0 |

## 1. Opseg

Iz jednog XML dokumenta sustav mora proizvesti **ravne zapise** — po jedan za svaki element koji
konfiguracija imenuje kao zapis, ili, bez toga, za svako dijete korijena — i predati ih strani
zapisa s ciljnim formatom. Rječnik izvora nije unaprijed poznat: oblik zapisa proizlazi iz samog
dokumenta.

RSS je poseban slučaj ovog zahtjeva s poznatim rječnikom ([`FRQ-PIP-17.1`](FRQ-PIP-17.1-feed-records.md)):
dijeli s njim cijeli tok osim koraka čitanja.

## 2. Akteri

| oznaka | tko / što | unosi u proces |
|---|---|---|
| **A1** | onaj tko pokreće prolaz nad izvorišnom lokacijom | opseg (uzorak, putanja) i trenutak pokretanja |
| **A2** | izdavač dokumenta, **izvan sustava** | rječnik, koji sustav ne poznaje unaprijed |
| **A3** | okvir strane izvora (parser) | otvaranje i zatvaranje izvora, reviziju pokušaja, jedinstveni razred kvara |
| **A4** | konfiguracija pipelinea | ciljni format i ime elementa zapisa |

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | konfiguracija je učitana; pipeline se gradi s ciljnim formatom |
| **EV02** | A1 predaje XML dokument |

## 4. Preduvjeti

Isti kao [`FRQ-PIP-17.1`](FRQ-PIP-17.1-feed-records.md) §4.

## 5. Normalan tok

| korak | radnja |
|---|---|
| 1 | A1 predaje dokument (`EV02`) |
| 2 | Izvor se čita uz ograničenja proširenja entiteta (`BR-07`) |
| 3 | Biraju se elementi zapisa: svaki element imena iz konfiguracije, na bilo kojoj dubini; bez imena — djeca korijena |
| 4 | Svaki element svodi se na ravni zapis: atributi i potomci postaju polja, ugniježđena putanja spaja se točkom, prostor imena se odbacuje, a ponovljeni element spaja u jednu vrijednost (`BR-05`) |
| 5 | Zapisi i ciljni format stavljaju se na dokument; dokument se objavljuje radnoj ploči |

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| nepoznat ciljni format | izgradnja pipelinea pada (`BR-08`) | workflow ne počinje s pogrešnom konfiguracijom |
| XML nije dobro oblikovan ili proširenje prelazi ograničenje | kvar čitanja se prijavljuje, dokument se preskače (`BR-06`, `BR-07`) | prolaz se nastavlja |
| nijedan element zapisa | upozorenje, dokument se preskače | nema praznog izlaza |
| element bez teksta i bez potomaka | polje postoji, prazno | prazna vrijednost se ne gubi |
| isto ime u atributu i u potomku | spaja se u jednu vrijednost | nijedna se ne prepisuje |

## 7. Poslovna pravila

Nasljeđuje `BR-01`, `BR-05`, `BR-06`, `BR-07` i `BR-08` iz [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) §4.
**`BR-03` ne vrijedi:** rječnik nije poznat, pa zapisi istog dokumenta mogu nositi različita polja;
CSV zaglavlje je njihova unija ([`FRQ-STR-17.2`](FRQ-STR-17.2-record-formats.md) §5).

## 8. Rezultat

Dokument nosi zapise i ciljni format i objavljen je radnoj ploči; izvornik je nepromijenjen
(`BR-01`).

## 9. Kriteriji prihvaćanja i verifikacija

| | kriterij | verifikacija |
|---|---|---|
| 1 | Djeca korijena i imenovani element daju zapise | **izvedeno** 2026-09-11 — smoke test |
| 2 | Atributi (i s prostorom imena), ugniježđene putanje, tekst i atribut lista postaju polja | **izvedeno** |
| 3 | Ponovljeni element spaja se u jednu vrijednost | **izvedeno** |
| 4 | Zapisi → XML → zapisi čuvaju vrijednosti; atributi se vraćaju kao elementi | **izvedeno** |
| 5 | Ime koje nije valjano XML ime odbija se pri pisanju | **izvedeno** |
| 6 | Prolaz s procesorom, radnom pločom i repozitorijem daje zapise u JSON-u, CSV-u i XML-u iz jednog dokumenta | **izvedeno** end-to-end |
| 7 | RSS i XML dijele tok; RSS se razlikuje samo korakom čitanja | pregled; RSS prolaz ponovljen nakon preuređenja — **izvedeno** |

**Testova nema** — deklarirana rupa (`DR-WFL-023`, D-11).

## 10. Dijagram

Tok je [`FRQ-PIP-17.1-feed-records.puml`](FRQ-PIP-17.1-feed-records.puml) uz korak čitanja iz §5
t.3–4; zaseban dijagram nije pisan. Struktura klasa: [dekompozicija](../01-HLRQ/HLRQ-17-decomposition.puml).

## 11. Otvoreno

- **Obilazak nije vjeran izvoru:** atributi se pri pisanju u XML vraćaju kao elementi.
- **Dokument se čita u memoriji**; strujanje velikih dokumenata nije podržano.
- **Prepoznavanje po deklaraciji:** dokument bez `<?xml` deklaracije repozitorij ne prepoznaje kao
  XML, iako ga pipeline čita — pipeline ne pita za tip, repozitorij pita.
- **Ime elementa zapisa** ne dopire do strane čitanja repozitorija: pohranjeni XML čita se po
  djeci korijena.
- **Mješoviti sadržaj:** tekst elementa koji ima i potomke ide pod ime elementa, uz polja potomaka.

## 12. Sljedivost prema izvedbi

Izvedba: `blackwattle/src/wattleflow/pipelines/xml/extract.py`, `helpers/converters/xml.py`,
`helpers/parsers/text.py`, `helpers/formatters/text.py`, `strategies/documents/xml.py`.
