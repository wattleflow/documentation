# DR-001: Zero-trust paketiranje frameworka

**Status:** Prihvaćeno (2026-05-29)
**Sloj:** Arhitektonska odluka — paketiranje, sigurnost, ovisnosti
**Vezano uz:** `Standard §7`, `docs/index.md` Sloj 4

---

## 1. Zašto?

### 1.1 Poslovna motivacija

Korisnik wattleflow frameworka može biti organizacija s strogim sigurnosnim zahtjevima — vladin sektor, financije, kritična infrastruktura, obrambena industrija. Za takve korisnike, **lanac opskrbe softvera (supply chain)** predstavlja konkretan rizik:

- Kompromitirana third-party biblioteka može uvesti malicious code u runtime
- Tranzitivne ovisnosti šire napadnu površinu izvan korisnikove kontrole
- Compliance režimi (npr. ASD ISM, NIST 800-161) eksplicitno traže evidenciju i kontrolu lanca opskrbe

### 1.2 Specifični problem

Trenutni `wattleflow-workflow` paket sadrži specijalizacije (`connections/`, `drivers/`, `processors/`, `strategies/`, `pipelines/`, `documents/`) koje povlače 10+ third-party biblioteka: `psycopg2`, `sqlalchemy`, `kafka-python-ng`, `paramiko`, `pysolr`, `elasticsearch`, `opensearch-py`, `pyspark`, `pdfplumber`, `pillow`, i druge.

Korisnik koji koristi samo jednu konekciju (npr. Postgres) **trenutno mora imati** sve te ovisnosti instalirane ili dostupne. Svaka od njih može biti vektor napada.

### 1.3 Princip zero-trust

> Framework core ne smije implicitno vjerovati nijednoj third-party biblioteci. Korisnik svjesno **opt-in** instalira točno one komponente koje koristi i preuzima odgovornost za njihov audit.

---

## 2. Što? (zahtjevi)

### 2.1 Funkcionalni

| ID | Zahtjev |
|---|---|
| F-ZT-1 | `wattleflow` core paket ovisi samo o Python standardnoj biblioteci |
| F-ZT-2 | `wattleflow-workflow` core paket ovisi o `wattleflow` + `cryptography` (samo OSCAL/audit potrebe) |
| F-ZT-3 | Specijalizacije se distribuiraju kao **zaseban projekt** (radni naziv: `wattleflow-examples`) |
| F-ZT-4 | Svaka specijalizacija u izdvojenom projektu deklarira vlastite third-party ovisnosti eksplicitno |
| F-ZT-5 | Sloj 3 (cross-cutting: `oscal/`, `audit/`, `constants/`, `helpers/`, `decorators/`) ostaje u core jer je sigurnosno integralan |

### 2.2 Nefunkcionalni

| ID | Zahtjev |
|---|---|
| N-ZT-1 | Core paket bez specijalizacija mora se moći importati i izvoditi u Python 3.10+ okruženju bez ijedne dodatne instalacije |
| N-ZT-2 | Izdvojeni projekt mora navesti sigurnosno upozorenje u README-u |
| N-ZT-3 | Tranzitivne ovisnosti svake specijalizacije moraju biti dokumentirane (SBOM-friendly) |
| N-ZT-4 | Korisnik mora moći zamijeniti svaku specijalizaciju vlastitom implementacijom bez modifikacije core paketa |

---

## 3. Kako? (dizajn)

### 3.1 Granica paketa

```
wattleflow            (stdlib only)
  └── core/           sučelja
  └── concrete/       generičke implementacije
  └── helpers/        utilities
  └── constants/      enumi
  └── decorators/     PresetDecorator, ...

wattleflow-workflow   (+ cryptography)
  └── oscal/          OSCAL standard
  └── audit/          SIEM forwarding (TODO)
  └── api/            FastAPI MVC (opcionalno)

wattleflow-examples   (radni naziv; opt-in po komponenti)
  └── connections/    psycopg2, sqlalchemy, kafka, paramiko, ...
  └── drivers/        avro, claude, elasticsearch, grafana, kafka, ...
  └── processors/     postgres-read, kafka-write, ...
  └── strategies/     specifične strategije
  └── pipelines/      pdf, text, png, dataframe, ...
  └── documents/      avro, dataframe, opensearch, protobuf, solr, ...
```

### 3.2 Pravilo ovisnosti

```
core → ničemu ne ovisi
concrete → core
helpers/constants/decorators → core
oscal/audit → core + concrete
specijalizacije → core + concrete + (third-party)
```

Suprotno **nije** dopušteno. Core paket nikad ne importa iz specijalizacija.

### 3.3 Migracijski put

1. **Faza A** — verifikacija: grep da nijedan modul u `core/`, `concrete/`, `helpers/`, `constants/`, `decorators/`, `oscal/` ne importa iz `connections/`, `drivers/`, `processors/`, `strategies/`, `pipelines/`, `documents/`.
2. **Faza B** — premještanje: stvaranje `wattleflow-examples` repozitorija, prijenos foldera, prilagodba `pyproject.toml` (jedan paket po specijalizaciji ili namespace package).
3. **Faza C** — dokumentacija: svaki podpaket dobiva README s ovisnostima, kontrolama (OSCAL) i Docker primjerom.
4. **Faza D** — verzioniranje: `wattleflow-examples` ima vlastiti version cycle.

### 3.4 Distribucijski model (predloženo)

- **Namespace package:** `wattleflow_examples.connections`, `wattleflow_examples.drivers`, ...
- **Extras u pyproject.toml:** `pip install wattleflow-examples[postgres,kafka]` instalira samo te dvije sa svojim ovisnostima
- **Default install:** samo metadata + dokumentacija; ništa se ne povlači bez ekstra

### 3.5 Veza s OSCAL

Svaka specijalizacija u izdvojenom paketu **mora** deklarirati `OSCAL_CONTROLS: ClassVar[Tuple[str, ...]]`. `@oscal_policy` dekorater (TODO, u core paketu) provjerava ih u runtime-u. Tako je compliance gating dio core obveze, ali popis kontroli specifične komponente je u njenom paketu.

---

## 4. Otvoreno

- Konačno ime izdvojenog projekta (radni: `wattleflow-examples`)
- Distribucijski model (jedan paket s extras vs. više neovisnih paketa)
- Strategija verzioniranja (semver per-component vs. monorepo verzija)
- Repository struktura (monorepo s više pyproject.toml vs. više repozitorija)

---

## 5. Reference

- `Standard.md §7` — sažeti standard
- `docs/index.md` — kontekst slojeva
- NIST SP 800-161 — Cybersecurity Supply Chain Risk Management
- ASD ISM — Information Security Manual (kontrolni katalog koji koristi OSCAL modul)
