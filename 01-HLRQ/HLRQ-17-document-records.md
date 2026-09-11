# HLRQ-17 — Strukturirani dokumenti u zapise

| | |
|---|---|
| **Status** | U kodu (2026-09-11) — RSS i XML; XLSX, ORC i Avro neprovjereni |
| **Odluka** | [`DR-PRC-004`](../04-DR/DR-PRC-004-pipeline-subjects-and-record-formatters.md) (prijedlog) — subjekti paketa, formateri primaju zapise |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `blackwattle` |
| **Djeca** | [`FRQ-PIP-17.1`](../02-FRQ/FRQ-PIP-17.1-feed-records.md) · [`FRQ-STR-17.2`](../02-FRQ/FRQ-STR-17.2-record-formats.md) · [`FRQ-PIP-17.3`](../02-FRQ/FRQ-PIP-17.3-xml-records.md) |
| **Dijagram** | [dekompozicija](HLRQ-17-decomposition.puml) — pogled, ne izvor istine (D-13) |
| **Sljedivost** | `NFRQ-ORG-02` · `NFRQ-ORG-04` · `NFRQ-ORG-08` · `NFRQ-ORG-09` · `NFRQ-SEC-03` · [`FRQ-PRC-15.22`](../02-FRQ/FRQ-PRC-15.22-document-flow.md) (tok perzistencije) |

> **Odluke autora od 2026-09-11 proturječe ovom zapisu** (D-02 — nalaz, ne izmjena zahtjeva):
> sadržaj se ne drži u metapodacima ([`FRQ-DOC-15.8`](../02-FRQ/FRQ-DOC-15.8-document.md) §1), a za
> strukturirane podatke koristi se DataFrame **gdje god je moguće**. Oblik razmjene iz §1 („lista ravnih
> zapisa") i predaja zapisa preko metapodataka (`FRQ-PIP-17.1` korak 6, `DR-PRC-004` t.5) usklađuju se
> kroz DR (D-03).

## 1. Narativ

Izvori objavljuju sadržaj u strukturiranim XML rječnicima — RSS je najrašireniji — a potrošači
podataka (analitika, arhiv, tablični alati, druge obrade) traže **zapise u standardnom formatu**:
JSON, CSV, XLSX, ORC, Avro. Sustav mora iz izvornog dokumenta proizvesti zapise u formatu koji
konfiguracija izabere, i pohraniti ih.

**Zašto.** Klasa po kombinaciji izvora i formata množi kod brojem izvora puta brojem formata.
Razdvajanjem na tri brige umnožak postaje zbroj: **izvor zna svoj rječnik, format zna svoj zapis,
a između njih putuje jedan oblik — lista ravnih zapisa.** Novi izvor ne traži novi format, novi
format ne traži novi izvor.

**Zatečeno (2026-09-11).** Paketi za RSS i XML pipelineove bili su prazna rezervirana mjesta.
Formateri tabličnih formata primali su samo tablicu, a driver spremišta po vlastitom ugovoru odbija
sve što nije već serijalizirano. Zapis iz strukturiranog dokumenta nije imao tko serijalizirati.

## 2. Mjesto u dekompoziciji

| sloj | briga | uloga | kategorija FR-a |
|---|---|---|---|
| **čitanje formata** | rječnik izvora → ravni zapisi; siguran XML | helper: konverter i parser | unutar `PIP` |
| **transformacija** | jedan dokument → zapisi i ciljni format | `Pipeline` | `PIP` |
| **perzistencija** | zapisi → standardni format → spremište | `Strategy` repozitorija, formater, `Driver` | `STR` |

Parseri i formateri su postojeća obitelj s tvornicom po tipu datoteke — sposobnost, ne novi
primitiv (`NFRQ-ORG-04`). Driver se ne mijenja: serijalizacija je posao formatera (`BR-09`).

## 3. Opseg

| oznaka | predmet | dokument |
|---|---|---|
| `FRQ-PIP-17.1` | RSS dokument (0.9x/2.0 i 1.0) u zapise, uz ciljni format | [FRQ-PIP-17.1](../02-FRQ/FRQ-PIP-17.1-feed-records.md) ✅ |
| `FRQ-STR-17.2` | Zapisi u standardnom formatu prema spremištu; pohranjeni RSS i XML natrag u zapise | [FRQ-STR-17.2](../02-FRQ/FRQ-STR-17.2-record-formats.md) ✅ |
| `FRQ-PIP-17.3` | XML dokument u zapise; RSS je njegova specijalizacija s poznatim rječnikom | [FRQ-PIP-17.3](../02-FRQ/FRQ-PIP-17.3-xml-records.md) ✅ |

## 4. Poslovna pravila

Vrijede za cijelu sposobnost; djeca ih nasljeđuju i pozivaju se na oznaku.

| oznaka | pravilo |
|---|---|
| **BR-01** | Izvornik se ne mijenja; zapisi nastaju kao nova datoteka. |
| **BR-02** | Vrsta izvora prepoznaje se po sadržaju kad naziv ne odlučuje (feed pohranjen kao `.xml`). |
| **BR-03** | Polja standarda prisutna su u svakom zapisu i kad su prazna; polja atributa (privitak) pojavljuju se samo kad ih izvor nosi. |
| **BR-04** | Datum se svodi na ISO 8601; vrijednost koja se ne da pročitati kao datum ostaje doslovno — ne gubi se i ne izmišlja. |
| **BR-05** | Ponovljeni element spaja se u jednu vrijednost, a pri pisanju u izvorni format ponovno se razdvaja. |
| **BR-06** | Dokument koji se ne da pročitati, ili nije traženog rječnika, prijavljuje se i preskače; prolaz se nastavlja. |
| **BR-07** | XML se čita samo uz ograničenje proširenja entiteta; vanjski entiteti i DTD se ne dohvaćaju. |
| **BR-08** | Ciljni format bira konfiguracija, ne kod. Nepoznat format zaustavlja izgradnju pipelinea; format bez formatera je kvar pisanja. |
| **BR-09** | Driver prima gotov sadržaj; format zapisa je stvar formatera. |

## 5. Nefunkcionalni zahtjevi i OSCAL

| NFR | posljedica za ovu sposobnost |
|---|---|
| `NFRQ-SEC-03` lokalnost | konverzija RSS-a i XML-a i serijalizacija u JSON, CSV, RSS i XML koriste samo standardnu knjižnicu; XLSX, ORC i Avro vuku svoje knjižnice lijeno. **Zatečeno:** tvornica formatera pri uvozu učitava cijelu obitelj, pa strategija zapisa nosi i ovisnosti tuđih formata (Pillow) |
| `NFRQ-SEC-02` napadna površina | javna površina deklarirana kroz `__all__` i lijeni agregat paketa (`DR-WFL-007`) |
| `NFRQ-ORG-01` lokalnost helpera | predaju zapisa dijele pipeline i strategija — dvije domene — pa živi u dijeljenom `helpers/` |
| `NFRQ-ORG-02` nomenklatura | pipeline nosi subjekt paketa; kriterij 5 za paket mjeren je od kriterija 0.9.0 (`DR-PRC-004`) |
| `NFRQ-ORG-08` deduplikacija | jedna strategija zapisa i jedna baza čitanja za svaki izvor; RSS pipeline je specijalizacija XML pipelinea, ne kopija |
| `NFRQ-ORG-09` vanjski standard | XML 1.0 i Namespaces in XML, RSS 2.0 i RSS 1.0, RFC 822 i W3C-DTF za datume, RFC 4180 (CSV), RFC 8259 (JSON) |
| `NFRQ-OBS-01…03` audit | pipeline i strategije pišu na `DEBUG`, `WARNING` i `ERROR`; `INFO` po dokumentu pripada procesoru |

**OSCAL.** Sposobnost ne uvodi konekciju, driver ni procesor, pa ne nosi OSCAL dekorator
(`CLAUDE.md` §6.1).

## 6. Otvoreno

1. **Format bira pipeline, a ne strana zapisa.** `write_context` nema implementatora, pa je
   konfiguracija pipelinea jedini kanal koji danas radi. Izbor pripada strani zapisa.
2. **Avro traži shemu**, a zapisi iz feeda je nemaju; izvođenje sheme iz polja nije odlučeno.
3. **XLSX, ORC i Avro nisu provjereni** — knjižnice nisu instalirane u okolini provjere.
4. **Atom (RFC 4287)** nije RSS i nije u opsegu; ulazi kao zaseban zahtjev ako zatreba.
5. **Broj 17 je sljedeći slobodni redni broj registra**, ne broj primjera (usp. `HLRQ-13` §7 t.2);
   primjer workflowa u `examples/` ne postoji.
6. **XML se čita u memoriji**; strujanje velikih dokumenata nije podržano.
7. Razred `HLRQ` nije u registru zahtjeva ([`HLRQ-000-EN.md`](HLRQ-000-EN.md) §Open).
