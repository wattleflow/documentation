<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-15 — Data-plane operation authorisation is declared configuration, not a call-time flag

| | |
|---|---|
| **Version** | v0.0.5 |
| **Quality (25010)** | Security — access control, accountability, non-repudiation |
| **Enforcement** | **not measured** — `wem_lint` does not cover this class; declared blind spot (D-11) |
| **Reference frame** | Saltzer & Schroeder, *least privilege*, *fail-safe defaults* [23]; NIST SP 800-53 `AC-3` (access enforcement), `AC-6` (least privilege) — same control family as the OSCAL `ac-3` candidate already carried by `DriverPostgres`/`DriverSpark` (`HLRQ-14`) |
| **Raised by** | `HLRQ-21` `BR-01…BR-03` — Spark write authorisation entering the register |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A component that can perform a **data-plane operation with a lasting effect on a shared platform**
— writing to a Spark server being the case in hand — treats permission for that operation as
**declared, categorised configuration**, never a boolean or argument accepted at the call site.
Absence of a prohibition is not permission (same principle as
[`NFRQ-SEC-07`](NFRQ-SEC-07-emission-authorisation.md) c.3, transplanted from emission to
write).

This entry exists because the flag-shaped gate already in the register (`allow_raw_sql`/`unsafe`
on `DriverPostgres`, `allow_raw_sql`/`safe_mode` on `DriverSpark`) is fragile by **construction**,
not merely by an implementation slip: any caller may pass the override at the call site, and
nothing in configuration shows, before the call, that the override is possible
(`FRQ-DRV-01.28` §11 t.5). The two drivers were not even
consistent with each other in how strong that gate was — evidence that the shape itself, not a
single driver's logic, is what needs replacing.

## 2. Acceptance criteria

1. **Declared authorisation.** A component names what it is permitted to do through categorised,
   static configuration — never a boolean such as `allow_raw_sql: true` or an `unsafe=True`
   argument accepted at the point of call.
2. **Default is refusal.** An operation whose target, format or mode falls outside the declared
   categories does not execute. The refusal names *which* category was not satisfied, without
   disclosing anything an attacker could use to probe the boundary
   ([`NFRQ-SEC-06`](NFRQ-SEC-06-audit-confidentiality.md)).
3. **Structural validation and authorisation are never the same gate.** The component that checks
   a query's or a table name's *form* (the driver) is not the component that decides whether the
   operation is *permitted* (the processor and the connection) — merging them into one flag
   reproduces the fragility this entry exists to remove
   (`HLRQ-21` `BR-03`).
4. **Accountability.** Every authorised operation and every refusal produces an audit record
   naming the category that granted or refused it, under the record contract of
   [`NFRQ-OBS-02`](NFRQ-OBS-02-audit-fields.md).
5. **One mechanism, not one per driver.** The declared-authorisation check is written once, at the
   processor/connection layer, and reused — not reinvented per driver as another `safe_mode`-shaped
   pair ([`NFRQ-ORG-08`](NFRQ-ORG-08-deduplication.md) c.4). This is the exact duplication this
   entry's raising finding (`FRQ-DRV-01.28` §11 t.5) exposed.
6. **`[M]`** Count of write call sites reachable without a categorised-authorisation check —
   **target 0**; a **gate candidate**, not yet a criterion. Promotion to a gate needs a documented change
   ([`NFRQ-DEF-02`](NFRQ-DEF-02-measurement-charter.md)).

## 3. Verification

**Nothing is verified today: no code exists.** `FRQ-CON-21.1`
and `FRQ-PRC-21.2` are proposals with empty
§9. The criteria above are an input to that implementation, not a report (D-05). Machine
measurement of c.6 is **not implemented** — declared blind spot (D-11), not coverage.

## 4. Rationale

A call-time flag is the failure mode with precedent in this register: `allow_raw_sql`/`unsafe`
exists on two sibling drivers, was copied from one to the other without checking it matched the
target's actual behaviour, and ended up in two different strengths
(`FRQ-DRV-01.28` §11 t.5). The flag's danger is not that either implementation is wrong — both do
what they say — but that **nothing in configuration shows, before the call, that an override
exists**. Anyone who can call `read`/`write` can also supply the override; declaring permission
ahead of time, above the driver, removes the class of mistake instead of tightening one driver's
threshold (the stopgap would otherwise invite).

## 5. Open

1. **Whether this norm generalises** beyond Spark write to the existing flag-shaped gates already
   in code (`DriverPostgres`, `DriverSpark` read path, `FRQ-DRV-01.28` §11 t.5) — not decided.
   Written narrowly on its single witness (`HLRQ-21`), same discipline as
   [`NFRQ-SEC-07`](NFRQ-SEC-07-emission-authorisation.md) §5 t.3: a norm claimed wider than what
   it has been checked against is an aspiration (D-05).
2. **Whether c.6 becomes a gate**, and what the lint would have to read to measure it — undecided.
3. **Exact form of the declared category** (`allowed_destinations`/`allowed_formats`/`allowed_modes`,
   `FRQ-PRC-21.2` §1) belongs with that FRQ,
   not decided here.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-27) — **the identifier is provisional**; entry into the register requires a documented change (D-03) |
