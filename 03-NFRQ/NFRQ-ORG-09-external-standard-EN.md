# NFRQ-ORG-09 — A specialisation over an external standard exposes its model

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-08-28) — **the identifier is provisional** |
| **Quality (25010)** | Functional correctness (completeness) · Maintainability — modularity, analysability |
| **Enforcement** | **not measured** — review; c.5 is AST-observable but not implemented (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |

> **Entered without a DR** (decision of 2026-08-28), against the default in D-03. The register
> is held to be sufficient justification for a norm of this kind; the `DR` series is reserved
> for **exceptions** to it. The deviation is recorded here rather than left silent.

## 1. Statement

Where a domain the framework processes is fixed by an **external normative standard** and an
implementation of that standard is already available in the standard library or in a vetted
dependency, the wattleflow specialisation **wraps that implementation** and carries it to the
framework's primitives. At the I/O boundary the wrapper belongs to the `parser` family, or to its
write-side mirror the `formatter` family — never to `adapter`, whose contract is to expose an
`IAdaptee` and which this is not.

It re-derives nothing the implementation already yields, exposes the **standard's model** rather
than the current consumer's projection of it, and carries only the **residue** — what the
standard omits, or what the framework itself needs (identity, digest, audit, the serialisation
boundary to metadata).

## 2. Acceptance criteria

1. The module **names the standard and the implementation** it wraps; where behaviour turns on a
   particular clause, the clause is named at that point.
2. The return type is the **domain model**, not a tuple or dict shaped to a caller.
3. Every capability of the referenced implementation is **reachable** through the wrapper.
   Reachability is the test, not present usage.
4. Hand-written code covering what the implementation already covers is a **finding**; the
   finding names the clause it duplicates.
5. **`[M]`** Public methods of the wrapper having exactly **one** call site — **diagnostic, not a
   gate**. It is the earliest visible sign that the model is not exposed and the consumer's need
   is being met by accretion. Charter:
   [`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md).

## 3. Verification

Review, recorded in the change record. **c.5 is not implemented** — declared blind spot (D-11),
not coverage.

## 4. Boundary

The residue is legitimate and this requirement does not forbid it: an implementation may be
absent, may not cover a clause, or may sit outside the distribution's tier
([`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality-EN.md)). What the requirement forbids is
**undeclared** residue.

Equally, the wrapper is not the home for what the standard does not decide — output naming,
rendering and persistence are the consumer layer's
([`NFRQ-ORG-01`](NFRQ-ORG-01-helper-locality-EN.md),
[`NFRQ-ORG-04`](NFRQ-ORG-04-crosscutting-capability-EN.md)).

## 5. Witness (D-05)

The mail parser, measured 2026-08-28. The superseded reader hand-rolled address, date and HTML
handling that `email.policy.default` already performs, and returned a projection
(`tuple[headers, body, attachments]`) shaped to one pipeline. Three losses were measured on one
message: a multi-author `From` (RFC 5322 §3.6.2) collapsed to a single address, the second
author lost without a record; the `-0000` "zone unknown" distinction (RFC 2822 §4.3) flattened
into a naive value and mixed with aware ones in the same archive; `<script>` body text carried
into the extracted content, because markup was stripped but its content was not.

The projection then drove accretion: sixteen methods on the parser, plus filename composition,
each serving one caller — c.5 in the field.

Decisive for this register: that reader **passed every existing gate** — `ORG-01`, `ORG-02`,
`OBS-02`, `ORG-07`. The gap was the register's, not the code's.

## 6. Justification

| Principle | Implication |
|---|---|
| `IParser` / `IFormatter` — a Strategy specialisation at the I/O boundary | the contract belongs to the standard; the wrapper owns only the crossing. `IAdapter` is **not** this role: its contract is to expose an `IAdaptee`, and it has no implementations |
| Conformance is not re-implementation | a standard's clause re-derived by hand is a second, unversioned implementation of it |
| MDL mirror ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface-EN.md)) | a projection hides the model, so each new need adds surface instead of reading it |
| Reachability over usage | a pipeline's present need is not the domain's extent; the next need must not require a code change in the adapter |
