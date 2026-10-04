<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-13 — Network exposure and least privilege of an instance

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration; corollary: a credential that never reaches the repository still protects nothing if the service that checks it is exposed with defaults |
| **CIA** | **Confidentiality · Integrity** (attack surface of the instance) |
| **Quality (25010)** | Security — confidentiality, integrity |
| **Enforcement** | text checks over `dockers/<instance>/` (c.1–c.5); **not in `wem_lint`** — declared blind spot (D-11); extends [`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md) from the module boundary to the network boundary |
| **Reference frame** | Saltzer & Schroeder, *least privilege*, *fail-safe defaults* [23]; ISO/IEC 27002:2022 §8.20 *Networks security*, §8.22 *Segregation of networks* [61]; NIST SP 800-53 `AC-6`, `SC-7`, `CM-7` |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

An instance exposes **nothing beyond the loopback interface by default**; publishing a port to
another interface is a declared change with a named audience. Inside the instance, services reach
each other on the compose network only; every convenience default that widens access
(sign-up, anonymous access, telemetry to the vendor) is **off**, and no container runs with more
privilege than its image needs.

## 2. Acceptance criteria

1. **Ports bind to `127.0.0.1`** in the tracked compose file; a bare `"3000:3000"` is a finding.
   A wider binding lives in an override file (`docker-compose.override.yaml`, ignored) or a
   documented change. *(machine-checkable — text)*
2. **Access-widening defaults are off**: `GF_USERS_ALLOW_SIGN_UP=false`, anonymous access off,
   vendor analytics/reporting off. *(machine-checkable — text)*
3. **No privilege escalation**: no `privileged: true`, no Docker socket mount, no `network_mode:
   host`, no `cap_add` without a stated reason. *(machine-checkable — text)*
4. **Inter-service addresses are service names** on the compose network (`prometheus:9090`,
   `pushgateway:9091`), never host ports. *(machine-checkable — text)*
5. **The push target is configuration, not code**: `pushgateway_url` comes from `.env`
   (`runtime.pushgateway_url`); a run that leaves the host uses TLS and authentication to the
   gateway — **candidate**, not yet a criterion (§5). *(by review)*

## 3. Verification

`grep` over the compose file on demand; the monitoring instance of 2026-09-19 meets c.1–c.4.
The operating point of c.1 is subject to [`NFRQ-SEC-04`](NFRQ-SEC-04-detection-operating-point.md):
a loopback-only Grafana that nobody can reach from the team's browsers leads to a hand-edited
binding, which is the bypass the criterion exists to prevent — the override file is the sanctioned
path.

## 4. Justification

| Principle | Implication |
|---|---|
| Fail-safe defaults [23] | the default is the closed state; opening is an act |
| Attack surface ([`NFRQ-SEC-02`](NFRQ-SEC-02-attack-surface.md)) | each published port is a channel across a trust boundary; the loopback binding removes the channel rather than guarding it |
| Adversary model ([`NFRQ-SEC-05`](NFRQ-SEC-05-adversary-model.md)) | a laptop stack's adversary is the local network, not the internet; the criterion is sized to that, not to a data centre |
| Reference standard: ISO/IEC 25010 — Security (confidentiality, integrity) | |

## 5. Open

TLS and basic authentication on the Pushgateway when a run pushes across hosts; a reverse proxy
in front of Grafana; running the stack rootless. Each is a compose change first, a criterion
only with a documented change.

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
