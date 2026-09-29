# KPI definitions – Northgate Industrial ITSM (P1)

> **Transparency notice:** Northgate Industrial is a fictional company; all data is synthetic ([sn-synthetic-itsm-data](https://github.com/nimish-mathur/sn-synthetic-itsm-data) v0.1.0).

**Status:** draft v0.1 (P1.2) – KPI set proposed from the standard content inventory (P1.1b, 2026-09-29).
Items marked **verify (P1.3)** depend on indicator internals not yet inspected (formula, field, conditions).

## Principles
- **Start from standard content, clone and adapt** (BP 0001078 / 0001199). New indicators only where no standard one fits.
- **Names** follow BP 0001078: subject-code prefix + colon (`INC:`, `SLA:`, `CHG:`, `DQ:`).
- **Leading and lagging** (BP 0001078 "Leading/Lagging cascading"): every lagging outcome has at least one leading indicator a team can act on.
- **Three dashboard layers** (BP 0001078): *Awareness* (leadership), *Assessment* (process owners), *Action* (team leads).
- **SLA outcomes come from `task_sla.has_breached`**, never `incident.made_sla` (P0 finding).
- **Durations in business hours** for P2–P4 (NGI schedule), consistent with the SLA definitions.

## KPI catalogue

| ID | KPI (indicator name) | Business question | Definition / formula | Built from standard content | Type | Layer | Target | Owner |
|---|---|---|---|---|---|---|---|---|
| K01 | `SLA: Resolution SLA attainment %` | Do we keep our fix promises? | Completed resolution SLAs not breached ÷ completed resolution SLAs, by completion date | Incident SLA content pack sources/indicators – **verify (P1.3)**; fallback: new indicators on `task_sla` | Lagging | Awareness | ≥ 90 % | Incident Process Owner |
| K02 | `SLA: Response SLA attainment %` | Do we pick tickets up in time? | Same as K01 for response SLAs | As K01 | Lagging | Awareness | ≥ 95 % | Service Desk Manager |
| K03 | `INC: MTTR (business hours)` | How long does a fix take? | Σ `business_stc` of incidents resolved that day ÷ number resolved (hours) | Clone *Summed duration of resolved incidents* (field **verify**: likely calendar time → clone on `business_stc`) + *Number of resolved incidents* → formula | Lagging | Awareness | P3 ≤ 6 bh | Incident Process Owner |
| K04 | `INC: First-contact resolution %` | How much does the desk solve alone? | Resolved by *NGI Service Desk* with `reassignment_count = 0` ÷ resolved | Clone *Number of incidents resolved on first assignment* + Service Desk condition (**decision D-P1-2**) | Lagging | Awareness | ≥ 70 % | Service Desk Manager |
| K05 | `CHG: Change success rate %` | Do our changes land safely? | 1 − unsuccessful ÷ closed changes (by close date) | *Number of unsuccessful changes*, *Number of closed changes* → formula | Lagging | Awareness | ≥ 95 % | Change Manager |
| K06 | `INC: Open backlog` | How much work is waiting? | Incidents opened on/before the day and not resolved by it, excl. cancelled | *Number of open incidents* (source *Incidents.Open* is **date-based** – correct under historic collection) | Leading | Assessment | trend ↓ | Incident Process Owner |
| K07 | `INC: Backlog older than 30 days` | Is old work piling up? | K06 where opened more than 30 days before the collection date | New indicator on *Incidents.Open* (relative date condition) | Leading | Action | ≤ 5 % of K06 | Team leads |
| K08 | `INC: Stale open incidents (> 5 days no update)` | What is nobody working on? | Open incidents with no update for 5 days | *Number of open incidents not updated in last 5 days* – **historic values approximate** (uses current `sys_updated_on`) | Leading | Action | trend ↓ | Team leads |
| K09 | `INC: New incidents` | What is the demand? | Incidents opened that day | *Number of new incidents* | Leading | Assessment | – (volume) | Service Desk Manager |
| K10 | `INC: Reassignment rate` | How often do tickets bounce? | Σ reassignments of resolved incidents ÷ resolved | *Total count of reassignment of incidents* + *Number of resolved incidents* → formula | Leading | Assessment | ≤ 0.5 | Incident Process Owner |
| K11 | `INC: Reopen rate %` | Do fixes stick? | Resolved incidents with `reopen_count > 0` ÷ resolved | Clone of *Benchmark: incidents closed that were re-opened* on *Incidents.Resolved* | Leading | Assessment | ≤ 3 % | Incident Process Owner |
| K12 | `DQ: Incidents without category` | Can we trust the breakdowns? | New incidents with empty category (plus legacy value `pc_issue`) ÷ new | New indicators on *Incidents.New* | Governance | Assessment | ≤ 1 % | Analytics owner |

## Breakdowns (to confirm against the 101 existing breakdowns in P1.3)
| Breakdown | Applies to | Why |
|---|---|---|
| Priority | K01–K04, K06–K11 | SLA targets differ by priority |
| Category | K03, K06, K09, K12 | Demand and data quality by type |
| Assignment group | K01, K03, K04, K06–K08, K10 | Team accountability (NGI groups only) |
| Location (site) | K06, K09 | Plant-level view (Lyon HQ + 5 plants) |
| Change type | K05 | Standard / normal / emergency risk profile |

## Known constraints (from P0 / P1.1b)
- **No incident → change or incident → problem link on the Brazil PDI** (`caused_by`, `rfc`, `problem_id` absent). Change-induced incident KPIs are out of scope for P1; the standard source *Incident.Problem.Open* has an invalid condition and is excluded.
- Active SLAs of open incidents are recalculated live by the SLA engine; attainment KPIs use completed SLAs.
- Historic values of "not updated in X days" indicators are approximate (current `sys_updated_on`).
- French public holidays are not yet in the NGI schedule (historic data already excludes them).

## Open decisions
| ID | Decision | Options |
|---|---|---|
| D-P1-1 | Collection jobs | (a) own jobs `ITSM.Daily` / `ITSM.Historic` with only the NGI KPI set (BP naming) – recommended; (b) activate standard `[PA Incident]`, `[PA Incident SLA]`, `[PA Change]` jobs |
| D-P1-2 | FCR definition | (a) NGI: Service Desk + no reassignment – recommended; (b) standard "resolved on first assignment" (any group) |
| D-P1-3 | Incident → change/problem link | (a) out of scope for P1, documented – recommended; (b) custom field; (c) task relationship |
