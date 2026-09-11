# Montra Fleet Maintenance, Breakdown & Uptime Management System

A working implementation of the Audree Infotech **FINAL v3.0** requirements baseline
for Montra Electric — preventive maintenance, breakdown service, job cards,
quality control and vehicle uptime, with VIN-level traceability throughout.

This is a running application, not a prototype: the business rules in Document 02
are implemented as executable code, and the screens are driven by a real database.

---

## What is implemented

| Area | Requirements | Notes |
|---|---|---|
| Organisation & data scope | FR-ORG-001..003 | Customer › Region › Depot › Sub-fleet and Montra › Zone › Service Centre. Every list, dashboard and API response is filtered by the signed-in user's scope, server-side. |
| Vehicle & component master | FR-VEH-001..005, CMP-001..003 | VIN as the lifecycle key; serialised component fitment history with old/new serial traceability. |
| Preventive maintenance | FR-PM-001..013, FR-PMO-001/002 | Persisted PM **obligations**, one per VIN / level / cycle, never overwritten. Km, calendar and operating-hour triggers; earliest reached wins. |
| Checklists | CHK-001..015 | Version-controlled, frozen onto the job at execution. OK / Not OK / Advisory-Observation / N/A / Numeric / Text / Photo. |
| Defects & concessions | FR-DEF-001, FR-CON-001 | Defects are VIN-owned and survive job closure; deferred items auto-attach to later service. Concessions carry expiry and follow-up, and block release once expired. |
| Appointments & capacity | FR-APT-001..004, CAP-001 | Bay, shift, technician roster and SRT-based duration. Double-booking prevented; no-show returns the obligation to the due queue. |
| Breakdown / RSA | FR-BD-001..010 | Intake, mobility-driven priority, triage, dispatch, roadside-to-workshop handover as a second job card under one service event. |
| Service execution | FR-SE-001, FR-JC-001..010 | Service Event › Job Card › Work Item spine with the canonical 19-state job card machine. |
| Parts | SPM-001..020, BL-SP-001..011 | Applicability validated before issue; serialised parts require serial capture and close the previous fitment. |
| Quality & release | QC-001..003, BL-QC-001..008 | Server-evaluated release gate with explicit, readable blockers. Segregation of duties enforced. |
| Uptime & downtime | FR-AVL-001..005 | Vehicle Status Ledger with non-overlapping intervals, attribution, source precedence and audited corrections. |
| SLA & clocks | FR-SLA-001, FR-CLK-001 | Explicit pause intervals — status alone never pauses a clock. Gross and net time both retained. |
| Reliability | BL-BD-014..016 | Repeat-failure matching and First-Time-Fix derived from persisted match rows, not report logic. |
| Telematics | FR-TEL-001, BL-USG-001..007 | Idempotent ingestion, monotonicity and plausibility validation, quarantine, staleness and projected usage. |
| Audit | FR-AUD-001, BL-SEC-004..010 | Append-only audit and status history; business records are cancelled or amended, never deleted. |

Phase 2 and 3 scope (predictive maintenance, battery-swap operations, deep ERP
automation, supplier analytics) is deliberately **not** implemented.

---

## Running it locally

Requirements: **Node.js 22.6 or newer** and **PostgreSQL 14+**. Nothing else —
the application has **zero third-party dependencies**.

```bash
# 1. Point at your database
export DATABASE_URL="postgres://user:password@localhost:5432/montra"

# 2. Create the schema and reference master data
node src/scripts/reset-db.ts

# 3. Load the demonstration fleet (optional but recommended)
node src/scripts/seed-demo.ts

# 4. Start
node src/index.ts
```

Then open <http://localhost:3000>.

### Demo accounts

Every account uses the password `montra123`.

| Username | Role | What they see |
|---|---|---|
| `ravi.kumar` | Service Centre Manager | Chennai workshop: scheduling, job cards, capacity |
| `kumar.s` | Technician | Assigned jobs and the checklist runner |
| `priya.n` | QC Supervisor | The release gate, concessions, releases |
| `vijay.r` | Service Coordinator | Breakdown queue, triage, off-hire |
| `meena.s` | Parts / Stores | Parts issue and VOR |
| `fleet.abc` | Fleet Customer | Only ABC Logistics vehicles |
| `admin` | System Administrator | Configuration, integrations, audit |

Signing in as different users is the quickest way to see row-level scoping:
`ravi.kumar` sees the Chennai fleet, `fleet.abc` sees only their own vehicles.

---

## Deploying to Render

1. Push this repository to GitHub.
2. In Render: **New → Blueprint**, and select the repository.
3. Render reads [`render.yaml`](render.yaml) and creates a PostgreSQL database
   and the web service, wiring `DATABASE_URL` automatically.
4. First boot applies the schema, reference data and demo fleet.

For a real deployment set `SEED_DEMO_DATA=false` and leave `MIGRATE_ON_START=true`
only for the initial release.

---

## How the code is organised

```
db/
  01_schema.sql              12 schemas, 86 tables — implements Document 04 §3
  02_seed_reference.sql      Montra master data: models, parts, SLAs, roles
src/
  index.ts                   HTTP server, routing, error envelope
  workers.ts                 PM re-evaluation, SLA clocks, housekeeping
  lib/
    pgclient.ts              PostgreSQL wire-protocol client (see note below)
    db.ts                    Query helpers, audit, status history, notifications
    http.ts                  Router, JSON handling, static serving
    auth.ts                  Passwords, tokens, roles, row-level scope
  domain/                    The business rules — the heart of the system
    pm.ts                    Due calculation, obligations, next-PM ladder
    availability.ts          Vehicle Status Ledger and transitions
    jobcard.ts               Canonical state machine and handovers
    checklist.ts             Execution, defect generation, deferred defects
    qc.ts                    Release gate and concessions
    reliability.ts           Repeat failure and First-Time-Fix
    usage.ts                 Telematics ingestion and data quality
  routes/                    REST API per Document 04 §25
  scripts/                   Migration, seed and a headless browser test driver
web/                         Single-page client (no framework, no build step)
docs/                        Traceability and deviations from the baseline
```

Every rule implementation carries the rule identifier from Document 02 in a
comment, so `BL-QC-003` in the specification is greppable in the source.

---

## Two deliberate deviations from Document 03

Document 03 locks the stack to **Angular + ASP.NET Core + Microsoft SQL Server**.
This build differs in two ways, both forced by the environment it was written in:

**1. PostgreSQL instead of SQL Server.** Render offers PostgreSQL only. The
schema is a direct translation — same tables, same columns, same constraints,
snake_case applied. See [`docs/DEVIATIONS.md`](docs/DEVIATIONS.md) for the
table-by-table mapping.

**2. Node.js instead of .NET, and a dependency-free web client instead of
Angular.** The build environment had no access to the .NET SDK or to the npm
registry, so the application is written against the Node and browser standard
libraries alone. One consequence is worth keeping: **the system has no
third-party dependencies at all**, which removes an entire class of supply-chain
review from your security assessment.

The domain model, entity names, status values, rule identifiers, API paths and
business logic all follow the baseline exactly, so this remains a faithful
reference implementation for the production .NET build — and a working system
in its own right in the meantime.

---

## Verification

`docs/ACCEPTANCE.md` maps the Document 05 acceptance scenarios (`AT-*`) to what
has been demonstrated against this build, including the negative paths: invalid
state transitions, release blocked by a critical defect, part applicability
rejection, segregation of duties, telematics quarantine and cross-scope denial.

---

*Prepared for Montra Electric · Audree Infotech Pvt. Ltd. · FINAL v3.0 baseline*
