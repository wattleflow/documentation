# FRQ-PIP-17.1 — Čitanje feeda u zapise

| | |
|---|---|
| **Status** | Provedeno u kodu 2026-09-11 — zapis nastao uz kod, ne obrnutim inženjerstvom |
| **Roditelj** | [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) |
| **Odluka** | [`DR-PRC-004`](../04-DR/DR-PRC-004-pipeline-subjects-and-record-formatters.md) (prijedlog) |
| **Norme** | `NFRQ-ORG-02` · `NFRQ-ORG-07` · `NFRQ-ORG-09` · `NFRQ-OBS-01…03` |
| **Standard** | RSS 2.0 (RSS Advisory Board); RSS 1.0 (RDF Site Summary) s Dublin Core; RFC 822 i W3C-DTF za datume; ISO 8601 |

## 1. Opseg

Iz jednog RSS dokumenta sustav mora učiniti dostupnim **kanal i njegove stavke kao ravne zapise**
i predati ih strani zapisa zajedno s **ciljnim formatom** koji je konfiguracija izabrala. Obje
verzije rječnika — 0.9x/2.0 i 1.0 (RDF) — daju isti oblik zapisa.

U koji se format zapisi serijaliziraju i kamo se pohranjuju **nije** predmet ovog zahtjeva
([`FRQ-STR-17.2`](FRQ-STR-17.2-record-formats.md)).

## 2. Akteri

Nijedan nije definiran ovim zahtjevom, a bez svakoga proces ne može početi:

| oznaka | tko / što | unosi u proces |
|---|---|---|
| **A1** | onaj tko pokreće prolaz nad izvorišnom lokacijom | opseg (uzorak, putanja) i trenutak pokretanja |
| **A2** | izdavač feeda, **izvan sustava** | rječnik, verziju i stupanj usklađenosti; sustav ne smije pretpostaviti da je potpun |
| **A3** | okvir strane izvora (parser) | otvaranje i zatvaranje izvora, reviziju pokušaja, jedinstveni razred kvara |
| **A4** | konfiguracija pipelinea | ciljni format zapisa |

## 3. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | konfiguracija je učitana; pipeline se gradi s ciljnim formatom |
| **EV02** | A1 predaje dokument RSS 0.9x/2.0 — korijen `rss`, stavke unutar kanala |
| **EV03** | A1 predaje dokument RSS 1.0 — RDF, stavke uz kanal, autor i datum u Dublin Core |

## 4. Preduvjeti

1. Ciljni format imenuje poznati tip datoteke (`BR-08`); inače se pipeline ne gradi.
2. XML čitač ima ograničenja proširenja entiteta (`BR-07`); inače se ne čita ništa.

## 5. Normalan tok

| korak | radnja |
|---|---|
| 1 | A1 predaje dokument (`EV02` ili `EV03`) |
| 2 | Izvor se čita i provjerava: dobro oblikovan XML, RSS korijen, postoji kanal |
| 3 | Kanal i svaka stavka svode se na ravni zapis po poljima standarda (`BR-03`); autor i datum iz Dublin Core preslikavaju se na polja RSS-a 2.0 |
| 4 | Datumi se svode na ISO 8601 (`BR-04`); ponovljeni element spaja se u jednu vrijednost (`BR-05`) |
| 5 | Za ciljni format RSS predaje se feed s kanalom; za svaki drugi — stavke kao zapisi |
| 6 | Zapisi i ciljni format stavljaju se na dokument; dokument se objavljuje radnoj ploči |

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| nepoznat ciljni format | izgradnja pipelinea pada (`BR-08`) | workflow ne počinje s pogrešnom konfiguracijom |
| XML nije dobro oblikovan, korijen nije RSS ili nema kanala | kvar čitanja se prijavljuje, dokument se preskače (`BR-06`) | prolaz se nastavlja |
| proširenje entiteta prelazi ograničenje | odbija se kao kvar čitanja (`BR-07`) | napad ne troši memoriju procesa |
| feed bez stavki | upozorenje, dokument se preskače | nema praznog izlaza |
| datum se ne da pročitati | ostaje doslovno (`BR-04`) | vrijednost se ne gubi |
| feed pohranjen pod drugim nazivom | pripadnost odlučuje korijen dokumenta (`BR-02`) | naziv ne odlučuje |

## 7. Poslovna pravila

Nasljeđuje `BR-01…BR-08` iz [`HLRQ-17`](../01-HLRQ/HLRQ-17-document-records.md) §4.

## 8. Rezultat

Dokument nosi zapise i ciljni format i objavljen je radnoj ploči; izvornik je nepromijenjen
(`BR-01`). Neuspjeh jednog dokumenta ne prekida prolaz.

## 9. Kriteriji prihvaćanja i verifikacija

| | kriterij | verifikacija |
|---|---|---|
| 1 | RSS 2.0 i RSS 1.0 daju isti oblik zapisa | **izvedeno** 2026-09-11 — smoke test nad oba uzorka |
| 2 | Polja standarda prisutna su u svakom zapisu i kad su prazna (`BR-03`) | **izvedeno** — JSON izlaz nosi `null` |
| 3 | RFC 822 i W3C-DTF svode se na ISO 8601; nečitljiv datum ostaje (`BR-04`) | **izvedeno** |
| 4 | Ponovljena kategorija se spaja, a u RSS izlazu ponovno razdvaja (`BR-05`) | **izvedeno** — obilazak RSS → RSS 2.0 → RSS daje iste zapise |
| 5 | Neispravan XML, tuđi korijen i *billion laughs* odbijeni su kao jedinstveni razred kvara (`BR-06`, `BR-07`) | **izvedeno** |
| 6 | Ime pipelinea zadovoljava gramatiku, a kriterij 5 za njegov paket je mjeren | **izvedeno** — `wem_lint` bez nalaza; fixture s tuđim subjektom daje ERROR |
| 7 | Pipeline ne piše na razini `INFO` (`NFRQ-OBS-03`) | pregled — `wem_lint` pipeline ne svrstava među korake, pa ovo ne mjeri |
| 8 | Prolaz s procesorom, radnom pločom i repozitorijem daje zapise u tri formata iz jednog dokumenta | **izvedeno** end-to-end — JSON, CSV i RSS |

**Testova nema** — `DR-WFL-023` odlučio je okvir, a testovi za `src/` ne postoje. Smoke test i
end-to-end prolaz izvedeni su izvan repozitorija; deklarirana rupa (D-11).

## 10. Dijagram

[`FRQ-PIP-17.1-feed-records.puml`](FRQ-PIP-17.1-feed-records.puml) — activity; gledište: put
jednog dokumenta od izvora do zapisa na dokumentu; publika: analitičar i implementator. Slika nije
generirana. Pogled, ne izvor istine (D-13).

## 11. Otvoreno

- **Ciljni format bira pipeline**, jer `write_context` nema implementatora (`HLRQ-17` §6 t.1).
- **Izvor se čita po imenu dokumenta, ne kroz driver** — kanal pipeline → driver ne postoji
  ([`HLRQ-13`](../01-HLRQ/HLRQ-13-llm-models.md) §7 t.3).
- **Atom (RFC 4287)** nije u opsegu.

## 12. Sljedivost prema izvedbi

Za strukturu klasa vrijedi [dekompozicija](../01-HLRQ/HLRQ-17-decomposition.puml), ne ovaj zapis.
Izvedba: `blackwattle/src/wattleflow/pipelines/rss/extract.py` (specijalizacija `pipelines/xml/extract.py`),
`helpers/converters/{xml,rss}.py`,
`helpers/parsers/text.py`, `helpers/records.py`, `enums/filetype.py`.
