# WBS — Routing sposobnost (dovršetak)

| | |
|---|---|
| **Status** | Aktivan (planiranje) |
| **Datum** | 2026-07-03 |
| **Veže se uz** | `DR-WFL-001` (odluka), `NFR-ORG-04` (`03-NFRQ/`), memorija `routing-capability-dr` |
| **Opseg** | dovršetak routing sposobnosti (`helpers/routing.py`, `helpers/files.py`) i njena integracija |
| **Izvan opsega** | korekcije zatečenih `wem_lint` 9E/6W (zasebna zadaća); test-framework odluka |

---

## 1. Svrha i opseg

Raščlamba (ŠTO, ne KADA/KAKO) preostalog posla nakon isporučene jezgre routinga
(klasifikacija po imenu + rezolucija odredišta + `FileSourceScanner`). Svaki work
package (WP) je zaokružen, ima isporuku, ovisnost i gate (blokadu/uvjet).

## 2. Struktura raščlambe

### WP1 — Dovršiti `03_pii_complex_workflow.yaml` (konkretna potreba)
- **Opis:** zamijeniti demonstracijski `format {label}` per-label `match` glob-ovima prema stvarnoj konvenciji imenovanja.
- **Isporuka:** izvršiv YAML s ispravnim preslikavanjem datum/uzorak → kategorija.
- **Ovisnost:** — · **Gate:** ⛔ tvoja preslikavanja (uzorak imena → kategorija/target).
- **Prihvaćanje:** svaka ulazna datoteka rutira se u točan `<kategorija>/`; nerutabilne → WARNING.

### WP2 — Kompletnost create-time žigosanja
- **Opis:** ostale Create strategije (`avro`, `json`, `text`, `word`, `youtube`, …) žigošu proslijeđene k-v u metadata (sad samo `file`/`pdf`).
- **Isporuka:** dosljedan create-time provenijencijski kanal ILI jedan hook u `StrategyCreate.create()` bazi.
- **Ovisnost:** — · **Gate:** ⚠️ univerzalna varijanta dira `concrete/` → **§2.5 odluka**.
- **Prihvaćanje:** proslijeđeni k-v završe u `document.metadata` neovisno o Create strategiji; bez regresije.

### WP3 — Per-transport routeri
- **Opis:** `KafkaDestinationRouter` (`topic`/`key`), `ElasticsearchDestinationRouter`/`OpenSearch` (`index`), `SparkDestinationRouter` (`partition_by`).
- **Isporuka:** konkretni routeri iza `DestinationRouter` apstrakcije; `route` → native parametar.
- **Ovisnost:** WP2 (obrazac k-v) · **Gate:** odluka bindinga (driver kao Information Expert vs config).
- **Prihvaćanje:** dokument s `route` labelom perzistira na točan topic/index/particiju; postojeći filesystem put nepromijenjen.

### WP4 — Driver-layer: `subdir` bez mutacije stanja
- **Opis:** `DriverLocalStorage` `subdir` postaje čisti per-call parametar (ukloniti `current_path` leak i workaround reset u write strategijama).
- **Isporuka:** stateless `subdir`; uklonjen `driver.current_path = driver.write_path` iz `WriteRouted*`.
- **Ovisnost:** — · **Gate:** dira driver (mehanizam — prihvatljivo, bez `concrete/`).
- **Prihvaćanje:** dvije sibling repozitorije na istom driveru pišu u ispravne subdir-ove bez curenja između dokumenata; test potvrđuje.

### WP5 — NFR-ORG-01 promocija (per-class fan-in)
- **Opis:** kad druga domena (npr. `drivers` kroz WP3) usvoji router, verificirati fan-in ≥2 i ukloniti WARN tolerancije za routing klase.
- **Isporuka:** `wem_lint` bez WARN-a za routing per-class fan-in.
- **Ovisnost:** WP3 · **Gate:** —.
- **Prihvaćanje:** `wem_lint` ne prijavljuje routing module kao single-domain.

### WP6 — Automatizacija NFR-ORG-04 lint kriterija
- **Opis:** proširiti `wem_lint.py` za kriterij 1 (sposobnost ne nasljeđuje domenski primitiv) i kriterij 4 (transport-termin vokabular).
- **Isporuka:** nova provjera + unosi u `naming_registry.yaml`.
- **Ovisnost:** — · **Gate:** —.
- **Prihvaćanje:** namjerno prekršena sposobnost (npr. `class FooRouter(StrategyWrite)`) pada lint.

### WP7 — Testovi
- **Opis:** jedinični (`RoutingRule`/`PatternSpec`/`route_target`/`FileSourceScanner`) + integracijski (procesor→create→write routing).
- **Isporuka:** test suite u odabranom frameworku.
- **Ovisnost:** — · **Gate:** ⛔ odluka test-frameworka (CLAUDE.md otvoreno).
- **Prihvaćanje:** testovi prolaze i pokrivaju matcher putanje (glob/regex/format), dvoskok dohvat, name-only.

## 3. Kritični put (preporučeni redoslijed)

```
WP4  (brzo, čisti leak, bez ovisnosti)
  └─> WP2  (create-time žigosanje — obrazac za transporte)
        └─> WP3  (per-transport routeri)
              └─> WP5  (fan-in promocija)
WP1  paralelno — čim stignu preslikavanja (Gate)
WP6  paralelno — tooling
WP7  paralelno — čim se odluči framework (Gate)
```

## 4. Izvan opsega (zasebno)

- Korekcije zatečenih `wem_lint` **9 ERROR + 6 WARN** (concrete↔helpers ciklus, §7.1 procurivanja) — **zadaća 5**, nakon ovog WBS-a.
- Deployment na github (samo lokalni snapshot/merge tijek — `make dev-snapshot`/`push-snapshot`).
