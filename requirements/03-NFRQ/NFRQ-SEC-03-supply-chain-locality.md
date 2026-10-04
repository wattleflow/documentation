<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-03 — Supply-chain trust and distribution locality

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Maintainability (modularity) · Security (integrity); NIST OSCAL for SBOM evidence |
| **Enforcement** | `wem_lint` — `clean_core_imports` (**error**), `distribution_manifest` (`warning`) |
| **Policy** | `ARCHITECTURE.md` §7 |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

> **A distribution boundary IS a security compartment.** This requirement extends
> [`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality.md) (dependency locality) **across the
> distribution boundary** and supersedes the withdrawn `NFRQ-ORG-06`.

## 1. Statement

A module's home distribution is determined by its **import closure, not by its role**.

A module belongs to a clean-core distribution (`wattleflow`, `wattleflow-workflow`) **only if**
its entire transitive closure fits within that distribution's permitted tier:

> **clean-core tier = stdlib ∪ `wattleflow` ∪ allowlist** (`tools/dictionary.json` →
> `scope.core_libraries`)

A module that references a third-party package — **eagerly or lazily** — belongs to a non-core
distribution (`blackwattle`, `wattleflow-cad`). **Deferred loading reduces import-time
cost but does not change the home distribution.**

Every distribution declares a **manifest** of the sub-trees it owns; **no `wattleflow.*` sub-tree
has two owners**.

## 2. Acceptance criteria *(partly machine-checkable)*

1. **Locality.** Every module in a distribution has a closure ⊆ that distribution's tier; a
   module with a third-party reference is not in a clean-core distribution.
   → *Machine:* `wem_lint` `clean_core_imports` plus the per-distribution manifest.

   > **Exception — guarded optional dependency**. A reference protected by a
   > `try/except ImportError` branch whose fallback is **functionally complete** and within the
   > distribution's tier **does not relocate the module**: its effective closure is the tier. The
   > criterion is the **completeness of the fallback**, not the presence of a `try/except` — a
   > branch that raises, returns `None` or degrades the function **is not** a guarded dependency.
   > Verification is a **masking test** (mask the package → the module must still import and
   > work), not a code review.
   > **Holders: none** (`scope.guarded_optional` is empty). The exception remains in force as a
   > mechanism; whether to keep or withdraw it is an open decision.

2. **Sole ownership.** Every `wattleflow.*` sub-tree is packaged by **exactly one** distribution
   (no import shadowing in the namespace merge).
   → *Machine:* manifest comparison.
3. **No symlinks in packaging.** A distribution does not package someone else's `__init__.py`;
   development uses **editable installs** (PEP 420/660), not symlinks.
   → *Review* plus the `packages.find` allowlist.
4. **Supply chain.** Every non-core distribution ships a **hash-pinned lock and an SBOM**;
   third-party code violating core policy (package / licence / known-bad) raises a warning.
   → *Machine:* an SBOM validator driven by core policy. **Not implemented** (D-11).
5. **Self-integrity of own modules.** The integrity of shipped modules rests on the **wheel
   `RECORD`** (per-file `sha256`) — it is **verified, not re-implemented**. A "distribution
   digest" (Merkle root) is the hash of the `RECORD` / the SBOM root; meaningful **only for built
   artefacts**.
   → For development/editable trees the `RECORD` is empty (it holds only a `.pth`), so there a
   `wem_lint` digest scan (`FileDigest`) takes over self-integrity and **simultaneously detects
   namespace shadowing/collision** (c.2). **Not implemented** (D-11).
6. **`[M]` Correlated breach.** `Blast` ([`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md)) is
   computed over the **transitive** closure; modules sharing a third-party dependency have
   **correlated** breaches — nominal isolation without supply-chain isolation is an illusion
   (log4j). **Ratio** scale; **diagnostic**.

## 2a. Evidence — blackwattle guard audit, 2026-09-27

Prompted by an author reminder while fixing `DriverSpark`/`SparkWriteProcessor`: an audit of every
`connections/*.py` and `drivers/*.py` module's top-level imports (grep for `try:`/`except`/
`ModuleNotFoundError` around each third-party reference) found **four** modules importing an
*optional* dependency (per `pyproject.toml` extras — `pyspark`/`pyarrow` in `streaming`, `pandas`
in `data`, both outside the base install) with **no guard at all**:

| module | unguarded import | fixed |
|---|---|---|
| `drivers/spark.py` | `from pyspark.sql import DataFrame, SparkSession` | wrapped in `try/except ImportError → ModuleNotFoundError` with install hint |
| `drivers/postgres.py` | `import pandas as pd` (the sibling `sqlalchemy` import in the same file *was* already guarded) | same |
| `drivers/proxy.py` | `import pandas as pd` | same |
| `drivers/http_proxy.py` | `import pandas as pd` | same |

`connections/spark.py` already guarded its own `pyspark` import correctly — the driver's own
sibling import was the gap. Verified by masking (D-10 style): running
`from wattleflow.drivers.spark import DriverSpark` and `from wattleflow.drivers.proxy import
DriverProxy` in the `core` conda env (genuinely lacking `pyspark`/`pandas`) now raises a clear
`ModuleNotFoundError` naming the package and the `pip install` command, instead of a raw traceback.
All 21 driver + 10 Spark connection/processor regression tests still pass; the pre-existing 65-error
baseline (`converters/word/*`, `tests.domains.legal.*`, one SDR test — all environmental, unrelated)
is unchanged.

**Not audited further:** every other module's top-level imports were checked by the same grep pass
(see table above for the only four gaps found); a full masking-test sweep across every connection
and driver (c.1's stated verification method, at module level) has not been run as an automated
`wem_lint` rule — this remains a manual, one-off audit, not continuous coverage (declared limit,
D-11).

## 3. Verification

| Criterion | Method | State |
|---|---|---|
| 1, 2 | `wem_lint` distribution-manifest gate | enforced |
| 3 | packaging review plus the `packages.find` allowlist | enforced |
| 4 | SBOM validator | **not implemented** (D-11) |
| 5 | `RECORD` verification / digest scan | **not implemented** (D-11) |
| 6 | metric over the SBOM | **not implemented** (D-11) |

**The masking test is nobody's machine criterion.** The procedure is carried by.3
(module level) and (package level), and c.1 names it as the verification method
**at module level**. The **package level** — *importability of a package under a masked optional
tier* — has no criterion, so `wem_lint` does not measure it. Open; requires a documented change (D-03).

## 4. Implementation notes

* **The tool must treat `foreign_imports` as a SEC-03 finding (a flag), not as an exclusion.**
  `wem_lint` currently uses a third-party edge to *exclude* a module from the ORG-01/02/03 scope —
  this requirement needs the same datum *reported*. The fix is a redesign: one graph, with `Rule`
  reading the edge for supply chain and `Metric` counting it for attack surface.
* Criterion 4 **does not re-implement** integrity: `pip --require-hashes` / `uv.lock` plus a
  CycloneDX/SPDX SBOM plus (optionally) Sigstore / PEP 740. `helpers/digest.py` (`FileDigest`)
  remains for the integrity of **our own** modules, not for third-party vetting.
* In a development tree the distribution digest is **not an identity token** (the source changes
  constantly) — it serves only as a drift/shadowing lint.
* `MANIFEST.in` is **the single control point for packaging**: a
  package that `packages.find` declares but `MANIFEST.in` omits installs **empty**.

## 5. Exception: `blackwattle` is not a zero-trust package

`blackwattle` is **deliberately outside** the scope of c.1 (`ARCHITECTURE.md` §7.4): it holds
specialisations resting on deferred third-party imports, plus the compliance layer. The package is an example implementation over existing open-source sub-systems;
**the user is responsible for auditing every installed dependency**, and that must be stated in
every description of it.

**Deferral must survive the aggregate**: lazy loading inside a module is worth
nothing if the package `__init__.py` undoes it with an eager re-export.

## 6. Justification

| Principle | Implication |
|---|---|
| Zero trust (supply chain) | clean core carries **zero** third-party code → a compromised library never reaches core users |
| Dependency Inversion / Stable Abstractions | a clean interface in core, the third-party adapter in a non-core distribution |
| Information Hiding (Parnas) | third-party specifics hidden behind the core abstraction |
| Occam / DRY | chain integrity rests on **standards** (SBOM / lock / attestations), not on a bespoke mechanism |
| PEP 420 / PEP 660 | namespace packages plus editable installs replace symlinks without losing live development |

Scientific basis: `workflow/Analiza.md` §5.5 (correlated breaches over transitive dependencies) — **the cited sections do not exist** in that file, which carries §1, §2 and §7 only (declared gap, D-11; checked 2026-09-19). Until they do, this criterion rests on a source the reader cannot open (D-05).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-07-09; moved from `NFRQ-ORG-06` and extended 2026-07-22) |
