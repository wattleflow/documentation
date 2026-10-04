<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# NFRQ-SEC-11 — Integrity of a deployed instance

| | |
|---|---|
| **Version** | v0.0.5 |
| **Postulate** | **P-23** — a secret is not configuration; corollary: everything that is *not* a secret **is** in the repository, and the instance is rebuilt from it |
| **CIA** | **Integrity** |
| **Quality (25010)** | Security — integrity, authenticity |
| **Enforcement** | text checks over `dockers/<instance>/` (c.1, c.2, c.4); review for c.3, c.5; **not in `wem_lint`** — declared blind spot (D-11) |
| **Reference frame** | ISO/IEC 27002:2022 §8.9 *Configuration management*, §8.32 *Change management* [61]; NIST SP 800-53 `CM-2`, `CM-6`, `SI-7`; [`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md) (supply chain) |
| **Language** | EN only — no HR edition ([`0-NFRQ` §Language](NFRQ-000-INDEX.md#language)) |

## 1. Statement

A supporting instance (`HLRQ-20`) is **fully described** by the tracked files under
`blackwattle/dockers/<instance>/` plus the untracked `.env`. What runs is what the repository says:
images are **pinned**, provisioning is **read-only**, and state that is not in the repository is
either derived (a scrape) or declared as a volume.

## 2. Acceptance criteria

1. **Every image is pinned to a version**; `latest` and untagged references are findings. A
   digest (`@sha256:`) is the target form; a version tag is the tolerated form until digests are
   recorded. *(machine-checkable — text)*
2. **Provisioning mounts are read-only** (`:ro`) and the provisioned datasource is
   `editable: false`; a dashboard edited in the UI is not the source of truth (D-13) — the file is.
   *(machine-checkable — text)*
3. **A change to the instance is a change to the repository**, reviewed like code; no
   `docker exec` configuration survives a recreate. *(by review)*
4. **Pushed telemetry keeps the labels the run declared** (`honor_labels: true`), so a scrape
   cannot rewrite `job`/`instance`/`workflow`. *(machine-checkable — text)*
5. **Integrity of the image source** — registry and publisher are named in the compose header;
   signature verification (cosign / Docker Content Trust) is a **candidate**, not a criterion,
   until the toolchain is decided. *(declared blind spot)*

## 3. Verification

`grep` over the compose and provisioning files on demand; the monitoring instance of 2026-09-19
meets c.1 (version tags), c.2 and c.4; c.5 is open. No CI runs it.

## 4. Justification

| Principle | Implication |
|---|---|
| P-20 — a rendering is never the source of truth | the running container is a rendering of the compose file; drift between them is the same class of defect as documentation drift |
| P-21 — concatenation is not composition | an unpinned image concatenates the repository with whatever the registry serves today |
| Supply-chain locality ([`NFRQ-SEC-03`](NFRQ-SEC-03-supply-chain-locality.md)) | the image is a third-party dependency of the instance; the pin is its lock file |
| Reference standard: ISO/IEC 25010 — Security (integrity, authenticity) | |

## Change history

| Version | Date | Change |
|---|---|---|
| v0.0.5 | 2026-10-02 | `Version` replaces `Status`; previous status: Proposal (Draft, 2026-09-19) — introduced by  **the identifier is provisional** until a documented change admits it it (D-03) |
