# Base 34 — The Remote House: team infrastructure

**Avatar-Energy · Base 34 · August 22, 2026**

*Architect's decision (2026-08-22): the team's development pipeline runs on a self-hosted, always-on remote rakazo instance. This base specifies the design before the infrastructure — the Gate before the cent.*

---

## The problem

The team needs to talk to persistent bots 24/7. The alternative — running the stack on the architect's machine — fails for three verified reasons: (1) the machine sleeps, and routines do not fire; (2) exposing the architect's machine to the Internet also exposes his GitHub credentials with push power over 8 repositories; (3) rakazo is beta (born 2026-08-13) and the hosted product "Rakazo Cloud" does not exist — "migrate local↔remote at will" is not an operation the project supports.

The answer is one house, always remote: bots live in the cloud; the architect's machine is a client.

## The INF spec

| ID | Requirement | Origin |
|---|---|---| 
| INF-001 | **one house only**: a single rakazo instance, always remote and always-on; no parallel local team instance — the architect's machine is a client (browser + local Hermes for direct terminal work) | decision |
| INF-002 | **honest size**: VM 2 vCPU / 2–4 GB RAM / ≥ 20 GB disk (official doc: 2 GB suffices for API+worker+Postgres; local Docker sandbox stays out — E2B/Daytona providers take the bots' computers) | self-host.md |
| INF-003 | **sandbox outside the house**: bot computers on a dedicated provider (E2B or Daytona) — the VM runs only API/worker/Postgres; execution isolated, disposable, never on the application VM | self-host.md |
| INF-003b | **model credentials stay in the house**: LLM keys encrypted in the VM's Postgres under the instance's ENCRYPTION_KEY; export by default: **never** — same law as AVA-006 | AVA-006 |
| INF-004 | **closed registration**: `SIGNUP_ALLOWLIST` with the architect's team; no open signup | decision |
| INF-005 | **public HTTPS + single origin**: WEB_ORIGIN/BETTER_AUTH_URL/API_URL under HTTPS with own domain; no plain HTTP | self-host.md |
| INF-006 | **state backup**: daily Postgres dump + DATA_DIR (bot homes, browser profiles) retained 7 days; restore tested quarterly | operation |
| INF-007 | **declared cost**: target monthly budget ≤ US$ 20 (Fly.io 2GB ≈ US$ 10.70; Hetzner CX22 4GB ≈ US$ 4.59 + volumes; E2B/Daytona per use) — deviation > 20% triggers spec review, not silence | decision |
| INF-008 | **controlled upgrades**: pinned rakazo version; upgrade only after changelog review — beta with expected breaking changes | self-host.md |
| INF-009 | **Hermes untouched**: nothing in this base changes the architect's local Hermes CLI — the monthly D1 keeps running on Hermes until an equivalent routine exists and is verified in the remote house | decision |
| INF-009b | **credential divorce**: the remote house does not inherit the architect's gh session — bots that need GitHub use a fine-grained token (scoped to one fork's repo, or a read-only deploy key where possible), never a person's gho_ | decision |

## The design

```
   TEAM (browser/mobile, anywhere)
        │ HTTPS
        ▼
┌─────────────────────────────────────────┐
│ THE HOUSE (always-on VM, Fly.io or      │
│ Hetzner)                                │
│  API ─ worker ─ Postgres ─ DATA_DIR     │
│  model credentials: encrypted there     │
└────────────────┬────────────────────────┘
                 │ sandbox provider
                 ▼
┌─────────────────────────────────────────┐
│ COMPUTERS (E2B/Daytona)                 │
│  browser+shell per bot, ephemeral       │
└─────────────────────────────────────────┘

ARCHITECT'S MACHINE = client (browser) + local Hermes (own terminal)
```

## What this base is not

- Not the bridge to "Rakazo Cloud" (it does not exist); it is self-hosting of the open code
- It does not move the house's specification work: specs keep being born in the repositories, trilingual, under SDD
- It does not replace the architect's Hermes (INF-009)

## Roles

The architect decides provider and domain name, provisions model credentials in the house. The hands execute: provision the VM, raise the stack (self-host guide), configure the allowlist, validate HTTPS, test the backup. Same contract: draft → reviewed → verified.

## Verification (how this spec tests itself)

| ID | Criterion |
|---|---|
| INF-001 | a single instance listed at the provider; no second team instance |
| INF-002/003 | `docker ps`/systemd on the VM shows only API/worker/Postgres; computers run on the sandbox provider |
| INF-005 | curl HTTPS on the public origin: no HTTP downgrade; valid certificate |
| INF-006 | dump restored in a test environment becomes a functional instance |
| INF-007 | monthly bill ≤ US$ 20 or recorded review |

---

*Status: draft — awaiting the architect's review. Code AGPL-3.0-or-later · Content CC BY-SA 4.0. Architecture and authorship: Cleiton Moura Loura.*
