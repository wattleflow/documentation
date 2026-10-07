
<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-17 — Third-party code is loaded at use, never at import

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Security — attack surface ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md)); Reliability — availability; Performance efficiency — time behaviour |
| **Enforcement** | **not measured** — no `wem_lint` rule checks *when* a library is imported; `SEC-03` measures *where* a `module` lives. Declared `blind spot` (D-11) |
| **`Policy`** | `ARCHITECTURE.md` §7.4· [`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md) §5 |
| **Raised by** | `HLRQ-26` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

In `lazy loading` for all third-party libraries - is imported **at the point of use**, inside the method that
needs it, and **only** when that use is reached. Importing a `wattleflow` module, or a package that
holds it, loads no third-party code. A library that is absent is discovered at use and **named**; it
never prevents the import of a module that does not use it.

## 2. Acceptance criteria

1. **Import at use.** No `src/` module has a module-level `import` or `from … import` of a
   third-party package; the statement sits in the function, method or property that uses it.
   *(machine-checkable — `AST`; not implemented)*
2. **Aggregates defer too.** A package that holds a module with a third-party import exposes its
   public names through `__getattr__` + `_EXPORTS`; importing one name loads one module.
   *(masking test)*
3. **Availability is not a parser's concern.** A parser or converter does not ask whether its
   library is present: the first use imports it, and an absent one raises the named error of
   criterion 4. Where a caller needs to know in advance (a test, a diagnosis), a helper class of
   the engine's own module answers it (`TikaApp.available()`), once — when the workflow or processor
   starts or the library is first instantiated — never on every call. *(unit test with the library masked)*
4. **A failed import is caught as it is, not as `ImportError`.** Loading a library may fail with any
   exception (a native library of the wrong version raises `AttributeError`, not `ImportError`); the
   guard covers what the library can raise, and the outcome is a **named** absence, not a bare
   traceback. *(unit test with a fake that raises)*
5. **An unselected engine is never loaded.** Where several libraries can serve one role, only the
   one selected in configuration is imported (`HLRQ-23` `BR-23-09`). *(masking test on the rest)*
6. **Masking test.** With every optional library masked, every module imports; a use that needs a
   masked library reports it absent (named error, or *unmeasured* for a measurement, `HLRQ-18`
   `BR-08`) and the process continues. *(unit test —.3)*
7. **No waiver.** The criterion carries no `declared` or `documented` waiver. *(by review)*

## 3. Verification

Masking test per module (remove the package from `sys.modules` and from the import path, import the
module, exercise the guarded use), plus a grep for module-level third-party imports until an AST rule
exists. **Blind spots (D-11):** a library that starts work in its own import (side effects at load)
is deferred by this criterion but not made cheap; a transitive import inside a third-party package
is out of reach.

## 4. Rationale

| `principle` | implication |
|---|---|
| Least functionality (`CM-7`) | code that is not used is not loaded, so it cannot be exploited |
| Blast radius ([`NFRQ-SEC-01`](NFRQ-SEC-01-blast-radius.md)) | a compromised or broken library harms only the path that calls it |
| Availability | one missing engine does not stop an unrelated workflow (23 libraries were needed to import one local driver) |
| Time behaviour | start-up cost is paid by the workflow that uses the library |

## 5. Open
1. **Current state is unmeasured.** The six libraries `cv2`, `faster_whisper`, `paddle`, `paddleocr`,
   `pynvml`, `rtlsdr` are imported inside methods; the other third-party imports have not been swept.
3. **Distribution effect.** Deferral does not move a module's home distribution;
   in a clean-core distribution the library is not a candidate for the tier ([`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md)).

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-06 | Criterion 3: a helper's check runs once (start or first instantiation), never per call; no parser carries an availability method (author's decision). |
| v0.0.5 | 2026-10-06 | Criterion 3: an advance answer, where one is needed, belongs to a helper class of the engine's module, never to the parser (author's decision). |
| v0.0.5 | 2026-10-06 | Criterion 3: `available()` withdrawn (author's decision: an advance check costs time and decides nothing the named failure at first use does not). |
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-30) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
