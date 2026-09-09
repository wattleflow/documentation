# NFRQ-SEC-07 — Emission is an authorised act, not a configuration key

| | |
|---|---|
| **Status** | Proposal (Draft, 2026-09-09) — **the identifier is provisional**; entry into the register requires a DR (D-03) |
| **Quality (25010)** | Security — integrity, accountability, non-repudiation; Functional suitability — appropriateness |
| **Enforcement** | **not measured** — `wem_lint` does not cover this class; declared blind spot (D-11) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-EN.md#language)) |
| **Raised by** | [`HLRQ-16`](../01-HLRQ/HLRQ-16-sdr-capture.md) `BR-12` — a transceiver enters the register |

## 1. Statement

A component that can cause a **physical emission** — radio transmission being the case in hand —
treats emission as an **act requiring declared authorisation**, not as a value a configuration file
may switch on. Absence of a prohibition is not permission.

This is the one place in the register where the constraint is not about the software's own
integrity: an emission leaves the machine, is irreversible, and is regulated by a legal regime
distinct from the one governing reception (`HLRQ-16` `BR-10`, `BR-12`).

> **Scope note.** This entry does **not** interpret any statute and states no legal conclusion. It
> requires that an authorisation be *declared, recorded and checked* — **what** authorisation is
> lawful is settled outside this register, by the operator (`HLRQ-16` §7 t.6).

## 2. Acceptance criteria

1. **Structural unavailability.** Where the device does not report the capability, the emitting
   path does not exist for the caller: it fails by **type**, not by a runtime flag read from
   configuration.
2. **Declared authorisation.** An emitting component names its authorisation as an explicit,
   resolvable reference — never a boolean such as `transmit: true`, whose default could make
   emission the quiet consequence of an incomplete file.
3. **Default is refusal.** A component whose authorisation is absent, unresolved or expired does
   not emit. The refusal names *which* authorisation is missing, without disclosing its content
   ([`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality-EN.md)).
4. **Accountability.** Every emission produces an audit record identifying the authorisation
   under which it occurred, the device, the frequency and the duration — under the record
   contract of [`NFRQ-OBS-02`](NFRQ-OBS-02-audit-fields-EN.md) and
   [`DR-WFL-008`](../04-DR/DR-WFL-008-audit-record-content.md). An emission with no record is a
   defect of the same class as an emission with no authorisation.
5. **Not merged with the receive path.** The emitting path is a separate contract and is not
   factored together with reception for resemblance
   ([`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication-EN.md) c.4).
6. **`[M]`** Count of emitting call sites reachable without an authorisation reference —
   **target 0**, and unlike most `[M]` entries this one is a **gate candidate**, not a diagnostic.
   Promotion to a gate goes through a DR ([`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter-EN.md)).

## 3. Verification

Review of the emitting component's public surface (c.1, c.5), of its configuration surface
(c.2, c.3), and of one recorded emission (c.4). **Nothing is verified today: no transceiver is
supported and no emitting code exists** — a check on 2026-09-09 found no SDR component at all in
`blackwattle/src/`. The criteria are an input to implementation, not a report (D-05).

Machine measurement of c.6 is **not implemented** — declared blind spot (D-11), not coverage.

## 4. Rationale

Reception is recoverable: a wrong frequency yields a useless capture. Emission is not — it reaches
other people's equipment and cannot be withdrawn. The asymmetry justifies an asymmetric default:
everywhere else in this framework configuration expresses a value, and here it may not express a
permission.

Criterion 1 exists because a boolean is the failure mode with precedent: a key that defaults to
the safe value in the code and to the unsafe one in a copied example file. Making the path absent
rather than disabled removes the class of mistake instead of documenting it.

## 5. Open

1. **What form the authorisation reference takes** — a licence identifier, an operator credential,
   an environment-resolved token, or a signed grant. Not decided; the shape belongs with the
   transceiver's FR, which is not written (`HLRQ-16` §4, `FRQ-PRC-16.6` candidate).
2. **Whether c.6 becomes a gate**, and what the lint would have to read to measure it.
3. **Whether this entry generalises** beyond radio to any actuator the framework may drive. Written
   narrowly on purpose: a norm claimed wider than its single witness is an aspiration (D-05).
