<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-MEM-01 — Instance memory: classes that inherit `Wattleflow` declare `__slots__`

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Performance efficiency — resource utilisation (memory) |
| **Enforcement** | **not measured** by `wem_lint`; a static check over the class tree is proposed, not implemented — declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |
| **Raised by** | [`HLRQ-01-GENERIC-LAYER`](../01-HLRQ/HLRQ-01-GENERIC-LAYER.md) `BR-PTN-07` · analysis `2026-10-01-slots-in-wattleflow-children` |

## 1. Statement

Every class that inherits `Wattleflow` declares `__slots__`, so that its instances carry no
`__dict__`. A class that holds no state declares `__slots__ = ()`. The rule holds for every class
of the generic layer (`workflow/src/wattleflow/concrete/`), including every child of the `Strategy`
family defined there, unless the class has a **declared exception**.

## 2. Acceptance criteria

1. Each class that inherits `Wattleflow` declares `__slots__`; the empty tuple stands for "no state".
2. `__slots__` names every instance attribute the class itself sets, and does not repeat a slot of its base.
3. Every base of such a class — each interface of `core` and each mixin it inherits — declares
   `__slots__ = ()` (or real slots). One base without `__slots__` gives every instance a `__dict__`,
   so criterion 1 then has no effect.
4. **Declared exception.** A class that cannot comply records the reason in its FRQ. A class without
   `__slots__` and without that record is a defect.
5. **`[M]`** The number of classes that carry a `__dict__` and have no recorded exception —
   **diagnostic, not a gate**. Charter: [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md).

## 3. Verification

Review of the class tree and of the FRQ of each class. A static check (class → `__slots__` declared,
every base in the MRO declares it) is **not implemented** — declared blind spot (D-11), not coverage.
Behaviour (`__dict__` present or absent) follows from the language rules and was **not** executed in
the analysis cited above.

## 4. Rationale

| Principle | Implication |
|---|---|
| a per-instance `__dict__` costs memory on every object | the cost is paid per item for classes created per item (pipelines, strategies, documents) |
| `__slots__` takes effect only along the whole MRO | the interfaces of `core` are part of the rule, not outside it (c.3) |
| `__slots__` removes `__weakref__` unless it is a slot | any registration by weak reference must be designed with this ([`NFRQ-PRF-02`](NFRQ-PRF-02-measurement-speed.md) §6) |

## 5. Open

| item | note |
|---|---|
| interfaces of `core` | 83 of 86 declare no `__slots__` (analysis §1); declaring `__slots__ = ()` in them is a `core` change |
| classes of `concrete/` without `__slots__` | `GenericPipeline`, `LazyIterator`, `LazyAsyncIterator`, `ThreadSafeObservable`, `Orchestrator`, the `Strategy` family, `RepositoryWithDriver`, `DummyReadDocument`, `WorkflowFactoryLogger` — no exception recorded |
| mixins outside the layer | for example `Serialise` (`IWattleflow`, no `__slots__`) used with three strategies |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-10-01) — **identifier and category `MEM` are provisional** (the same abbreviation names the Memento role on the FRQ axis); entry into the register requires a documented change (D-03) |
