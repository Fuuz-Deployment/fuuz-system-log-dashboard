# Fuuz System Log Dashboard — Reference Guide

> **Audience:** Developers, Managers, and Administrators  
> **Version:** 0.0.9 · Deploy Date: 2026-06-14  
> **URL:** `https://build.mfgx.fuuz.app/system/testing/systemLogDashboard`  
> **Last scanned:** Sun Jun 14 2026 at 12:45 PM

---

## Overview

The **System Log Dashboard** is a multi-tab observability and governance screen built in Fuuz. It provides a unified view of the health, activity, and risk posture of every major artifact type in a Fuuz environment — users, screens, data flows, data models, documents, saved scripts, saved APIs — plus two special-purpose tools (an embedded HTML governance report and an interactive ERD).

Each tab follows the same structure:

1. **Stat cards (KPI bar)** — headline metrics at a glance, color-coded by urgency (blue = informational, green = healthy, orange = warning, red = critical).
2. **Chart panels** — interactive charts that expand to full-screen via the ⟺ icon in the top-right corner of each card.
3. **Detail tables** (on some tabs) — filterable row-level lists of the specific items that contribute to a flagged KPI.

---

## Navigation

The tab bar at the top of the screen contains nine tabs:

| # | Tab | What it monitors |
|---|-----|-----------------|
| 1 | **USERS** | Login activity and data change behavior per user and domain |
| 2 | **SCREENS** | Inventory, deployment, risk, and authorship of all Fuuz screens |
| 3 | **DATA FLOWS** | Flow deployment, type mix, integrity issues, and risk signals |
| 4 | **DATA MODELS** | Model inventory, execution type, field distribution, and risk |
| 5 | **DOCUMENTS** | Document deployment, render status, and risk tier |
| 6 | **SAVED SCRIPTS** | Script deployment, language mix, and authorship |
| 7 | **SAVED APIS** | Saved query quality, parameter hygiene, and risk signals |
| 8 | **SCREEN HTML** | Embedded full-featured Screen Governance HTML report |
| 9 | **FUUZ ERD** | Interactive entity-relationship diagram across all data models |

The footer bar shows: **Last Authenticated At**, **Screen Version**, and **Deploy Date** for quick version confirmation.

---

## Tab 1 — USERS

**Purpose:** Detect unusual authentication patterns, identify the most active users, and understand how data changes are distributed across domains and model types.

### Charts

| Chart | Type | What to look for |
|-------|------|-----------------|
| **Authentication Events by Week** | Stacked bar + avg line | Spikes well above the Avg 35 line may indicate bulk logins, scripted access, or onboarding bursts. A bar near zero (e.g., the 2-event week of 06/14) may indicate a maintenance window or outage. Color segments distinguish domains (Fuuz Team, 40solutions.com, fanucamerica.com). |
| **Authentication Trend by Domain** | Multi-series line + avg line | Watch for new domains appearing or existing ones dropping to zero — both indicate permission or integration changes. The Fuuz Team line dominates; any sustained orange (fanucamerica.com) or green (40solutions.com) spike warrants review. |
| **Change Engagement by Model Type** | Dual-line (Master vs Transactional) | A crossover where Transactional overtakes Master may indicate system maturation or a shift in app usage patterns. Widening gaps in either direction suggest imbalanced write load. |
| **Change Type Over Time** | Stacked bar (Create / Update / Delete) | Healthy systems show far more Creates and Updates than Deletes. A Delete-heavy week should be investigated. |
| **Change Significance** | Stacked bar (Significant vs Insignificant) | Significant changes include field edits that affect business logic. A sudden rise in insignificant changes (e.g., whitespace, label tweaks) may indicate automated scripting or bulk migrations. |
| **Most Active Users** | Horizontal bar (Significant vs Insignificant) | The top two users are `cscott@mfgx.io` (1,178 changes) and `dataflows-noreply@mfgx.io` (822). The `dataflows-noreply` service account appearing prominently is normal if automated flows are writing records; an unexpectedly high count warrants checking which flows are triggering changes. |

### What to flag
- Any domain not recognized in the authentication charts.
- Sudden drop in weekly auth events without a known maintenance window.
- Service account (`dataflows-noreply`, `system-noreply`) change volumes exceeding human users without a known automation reason.

---

## Tab 2 — SCREENS

**Purpose:** Monitor the health of the screen inventory — which screens are deployed, which carry risk, and where element hygiene needs attention.

### Stat Cards (KPI Bar)

| Card | Value | Description | Concern threshold |
|------|-------|-------------|-------------------|
| TOTAL SCREENS | 182 | All screens in inventory | Baseline reference |
| DEPLOYMENT RATE | 32% | 59 of 182 deployed; 123 never deployed | <50% warrants review of stale development screens |
| TIER 1 RISK | 0 | Screens containing inline scripts | Any value >0 is critical |
| HYGIENE ISSUES | 280 | Total element-level hygiene violations | Reduce toward zero over time |
| NO PARENT | 0 | Orphaned screens with no parent route | Any value >0 is a render risk |
| STALE 90D+ | 175 | Screens not touched in 90+ days | High value — consider archiving |
| ACTIVE UNDEPLOYED | 121 | Screens marked active but never shipped | Should be resolved or demoted |
| UPDATED 7D | 1 | Screens touched in the last 7 days (3 in 30d) | Low activity — expected in stable phase |

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Active vs Inactive** (donut) | 98% Active (178/182). 2.2% inactive screens may be legacy artifacts. |
| **Deployment Status** (donut) | 32% Deployed (59/182). The 68% gap means 123 screens exist only in development. Coordinate with owners before any cleanup. |
| **Edge vs Standard** (donut) | Only 1 Edge screen of 182 — Edge screens run client-side and have stricter performance requirements. |
| **Risk Tier Distribution** (donut) | 9 at risk: 0% T1 (Scripts), 2.75% T2 (Heavy elements), 2.2% T3 (Hygiene), 95.05% No Risk. Focus first on the T2 screens. |
| **Element Density Distribution** (histogram) | Most screens fall in the 1–10 and 26–50 element buckets. 20 screens have 100+ elements — review these for performance. |
| **Top Bloated Screens** (horizontal bar) | UpdatedEntAdminPage (544 elements) and xEntDash1 (449) are outliers. Bloated screens increase load time and maintenance cost. |
| **Screen Activity Trend** (line) | Created vs. updated per week. Avg created = 3/week. Spikes in creation (e.g., 01/04/2026: 35 created) indicate development sprints. |
| **Screen Complexity by Module** (bar) | System module dominates with 6,770 elements. Machine Telemetry (883) and Quality Tracking (311) are next. |
| **Deployment Health** (stacked bar) | System module has the most undeployed screens (144 total, few deployed). Machine Telemetry and Human Capital Management show better deployment ratios. |
| **Authorship by Domain** (bar) | Fuuz Team has authored 353 screen versions and created 182 screens — all authorship concentrated in one domain, expected for a single-org environment. |
| **Update Activity (Last 30 Days)** (area) | Only 3 screens updated in 30 days. Low activity is expected in a stable system; a zero-update period of 60+ days may indicate stagnation. |
| **Top Action-Heavy Screens** (bar) | entAdminHomepageDev3 has 19 action elements — the only screen above the 10-action threshold. High action counts increase flow invocation risk. |
| **Top Authors** (horizontal bar) | cscott@mfgx.io (108), rbrooks@mfgx.io (47). |
| **Top Editors** (horizontal bar) | cscott@mfgx.io (106), rbrooks@mfgx.io (49). |

### Detail Tables

**RISK-BEARING SCREENS** (9 rows)
Columns: Tier · Screen · Module · Risk Signals · Elements · Active

| Tier | Screen | Key Risk Signals |
|------|--------|-----------------|
| T2 | xEntDash1 | noDataPath (37), notInForm (37), themeUnsafe (40) |
| T2 | entAdminHomepageDev3 | actions (19), noDataPath (17), noDesc (3), notInForm (17), themeUnsafe (32) |
| T2 | appAdminDev4 | noDataPath (12), noDesc (12), notInForm (12), themeUnsafe (12) |
| T2 | myProfileTest1 | noDataPath (9), notInForm (1), themeUnsafe (9) |
| T2 | testAlarm1 | noDataPath (6), noDesc (6), notInForm (6), themeUnsafe (5) |
| T3 | testing9768 | themeUnsafe (2) |
| T3 | historyTest | themeUnsafe (2) |
| T3 | test9992 | themeUnsafe (2) |
| T3 | ewqeqwewqeqweqwe | noDesc (1) |

**SCRIPT-HEAVY SCREENS (Tier 1):** No screens match ✓ — This is a green signal; no inline scripts detected.

**ACTION-HEAVY SCREENS (>10 Actions)** (1 row): entAdminHomepageDev3 — 19 action elements, 203 total elements.

**SCREENS WITH ELEMENT HYGIENE ISSUES** (9 rows)
Columns: Screen · Module · Issues · Detail — Lists the same screens as Risk-Bearing with human-readable issue descriptions (e.g., "37 no dataPath, 37 not in form, 40 theme-unsafe").

**BLOATED SCREENS (>50 Elements)** (25 rows): UpdatedEntAdminPage (544), xEntDash1 (449), System Log Dashboard (251), Gateway Dashboard (248), entAdminHomepageDev3 (203)…

### Risk signal glossary
| Signal | Meaning |
|--------|---------|
| `noDataPath` | Element has no bound data path — will render empty |
| `notInForm` | Input element is not inside a form container |
| `noDesc` | Element lacks a description/label |
| `themeUnsafe` | Element uses hardcoded colors/styles instead of theme tokens |
| `actions` | Count of action-type elements (buttons, triggers) |

---

## Tab 3 — DATA FLOWS

**Purpose:** Audit the operational health and risk posture of all data flows — their deployment state, node composition, concurrency patterns, and which flows carry elevated risk.

### Stat Cards

| Card | Value | Description | Concern threshold |
|------|-------|-------------|-------------------|
| TOTAL FLOWS | 40 | All flows in inventory | Baseline reference |
| DEPLOYMENT RATE | 80% | 32 deployed; 8 never deployed | Good — 8 undeployed flows need review |
| TIER 1 RISK | 1 | Confirmed destructive operation | Any value >0 requires immediate review |
| HARDCODED CREDS | 0 | Secrets in flow definition | Any value >0 is critical |
| ORPHAN REQUESTS | 3 | Request nodes with no response path | Fix to avoid hung executions |
| MUTEX ISSUES | 3 | Lock/unlock errors or retry loops | Fix to prevent deadlocks |
| STALE 90D+ | 38 | Flows not updated in 90+ days | 95% of inventory — very high |
| UPDATED 7D | 1 | Recently touched flows (2 in 30d) | Low activity |

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Active vs Inactive** (donut) | 100% Active — all 40 flows are active. No flows have been deactivated. |
| **Deployment Status** (donut) | 80% Deployed (32/40). 8 undeployed flows may be in development or abandoned. |
| **Flow Type** (donut) | 67.5% System, 20% Screen, 5% Edge, 5% Integration, 2.5% Document. The System majority is expected for a platform-level environment. |
| **Node Type Mix** (horizontal bar) | 409 total nodes. Most common: transform (88), query (46), response (41), javascriptTransform (40), mutate (38), setContext (33). High `mutate` counts increase write risk. |
| **Flows by Module** (horizontal bar) | Machine Data App (19), Testing (2), Configuration (2), Orchestration (1)… Machine Data App dominates — review its flows for the Tier 1 risk. |
| **Integrity & Concurrency Issues** (horizontal bar) | Fork/Combine Mismatch (5) is the highest single issue type. Orphan Request (3), Ambiguous Response (3), Unhandled Errors (3), Mutex Retries=-1 (3). Each of these can cause silent failures in production. |
| **Flows by Module Group** | Machine Telemetry Site Application (19 flows, 70.4% of total, 17 deployed at 89%, 19 active, 1 T1 risk, 14 T2 risk, 19 stale >90d). |
| **Top Authors** | cscott@mfgx.io (24), system-noreply@mfgx.io (8), lmenard@mfgx.io (4). |
| **Top Editors** | cscott@mfgx.io (24), rhamilton@mfgx.io (6), lmenard@mfgx.io (4). |
| **Update Activity (Last 30 Days)** | 2 flows updated this month — very low. |

### Detail Tables

**RISK-BEARING FLOWS** (30 rows)
Columns: Tier · Sec · Flow · Module · Signals · Used By · Active

Key high-risk entries:

| Tier | Flow | Signals | Used By |
|------|------|---------|---------|
| T1 | Alarms - Delete All Alarms | `graphqlDelete` | 2 |
| T2 | Screen Generator | `broadcast, executesFlow, graphqlWrite` | 70 |
| T2 | Connect to Confluence page | `graphqlWrite, httpWrite` | 42 |
| T2 | WorkcenterHistoryScheduler | `graphqlWrite, jsCode, mutexBadRetries` | 21 |
| T2 | Telemetry Generator Scheduler IIoT Raw Data | `broadcast, graphqlWrite, jsCode` | 20 |

> **Screen Generator** is used by 70 screens — any failure or change here has wide blast radius.

---

## Tab 4 — DATA MODELS

**Purpose:** Review the schema inventory, identify models with hard-delete risk or missing audit trails, and understand the distribution of model types across modules.

### Stat Cards

| Card | Value | Description | Concern threshold |
|------|-------|-------------|-------------------|
| TOTAL MODELS | 62 | All models in inventory | Baseline |
| DEPLOYMENT RATE | 97% | 60 deployed; 2 never deployed | Excellent |
| TIER 1 RISK | 2 | Models with hard-delete exposed | Review immediately |
| NO AUDIT TRAIL | 0 | Models without DCC (data change capture) | Any value >0 is a compliance concern |
| DESC MISMATCHES | 0 | Copy-paste errors in field descriptions | Clean |
| UNINDEXED DT | 0 | DateTime fields missing index (sort/filter cost) | Clean |
| STALE 90D+ | 60 | Models not touched in 90 days | 97% of inventory — expected for stable schema |
| UPDATED 7D | 0 | No models updated this week | Expected in stable phase |

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Model Kind** (donut) | 100% Reference — all models are Reference type. No Transactional-only models in this set. |
| **Model Execution Type** (donut) | 50% Unknown, 19.35% Setup, 17.74% Transactional, 12.9% Master. The 50% Unknown is worth resolving for better governance. |
| **Deployment Status** (donut) | 97% Deployed (60/62). 2 models never deployed — confirm whether they are in active development or orphaned. |
| **Risk Tier Distribution** (donut) | 2 at risk (3.23% Tier 1 / Hard Delete). These two models allow permanent record deletion without soft-delete protection. |
| **Field Type Distribution** | String (2), Relation (2) — shown for the small subset of custom-typed fields. |
| **Models by Module** | Testing (6), Learning (6), Orchestration (3), Manufacturing (2), Session 1 Stock Prices (2). |
| **Models by Module Group** | Machine Telemetry Site Application (24), Applications (9), System (9), Quality Tracking (8), Human Capital Management (6). |
| **Largest Models (by field count)** | Diff (7 fields), DiffGroup (5 fields) — these are internal diff/versioning models with minimal fields. |
| **Top Authors** | cscott@mfgx.io (31), lmenard@mfgx.io (10). |

---

## Tab 5 — DOCUMENTS

**Purpose:** Track the health of Fuuz document templates (Stimulsoft .mrt files). This environment has only one document currently.

### Stat Cards

| Card | Value | Description | Concern threshold |
|------|-------|-------------|-------------------|
| TOTAL DOCUMENTS | 1 | Single document in inventory | Baseline |
| DEPLOYMENT RATE | 0% | Never deployed | **Action required** |
| TIER 1 RISK | 0 | Broken documents that won't generate | Clean |
| FLOW SUBSTITUTED | 0 | Documents pointing to deprecated flows | Clean |
| NEVER RENDERED | 0 | Deployed but never rendered | N/A (not deployed) |
| ACTIVE UNDEPLOYED | 1 | Marked live, not shipped | **Action required** |
| STALE 90D+ | 1 | Not touched in 90+ days | Monitor |
| UPDATED 7D | 0 | No updates this week | Expected |

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Active vs Inactive** | 100% Active (1/1). |
| **Deployment Status** | 0% Deployed — the one document has never been deployed. If it is meant for production, deploy it or change its status to inactive. |
| **Risk Tier Distribution** | 1 at risk — classified as Tier 3 (Hygiene). Not critical but should be reviewed. |
| **Render Volume Distribution** | Documents bucketed by lifetime render count — with 0 deployments, render count is 0. |

---

## Tab 6 — SAVED SCRIPTS

**Purpose:** Audit reusable JSONata/JavaScript scripts used across screens and flows.

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Active vs Inactive** | 100% active — no retired scripts. |
| **Deployment Status** | 73.3% deployed (26.67% never deployed). 4 undeployed scripts need attention. |
| **Script Language** | 93.33% JSONata, 6.67% JavaScript. JavaScript scripts have more risk surface (arbitrary execution); review them for safety. |
| **Authorship by Domain** | 100% Fuuz Team — all scripts authored internally. |
| **Top Authors** | cscott@mfgx.io (10), system-noreply@mfgx.io (4), rhamilton@mfgx.io (1). |
| **Top Editors** | cscott@mfgx.io (11), system-noreply@mfgx.io (3), rhamilton@mfgx.io (1). |

---

## Tab 7 — SAVED APIS

**Purpose:** Review the quality and integrity of saved GraphQL queries used across screens and flows.

### Stat Cards

| Card | Value | Description | Concern threshold |
|------|-------|-------------|-------------------|
| TOTAL QUERIES | 11 | All saved queries | Baseline |
| MUTATIONS | 0 | Write operations in saved queries | Any mutations should be reviewed |
| EMPTY QUERIES | 2 | Queries with no queryText | Fix — will return no data |
| HARDCODED LIMITS | 1 | `first: N` not parameterized | Fix — prevents dynamic pagination |
| PAGINATION BROKEN | 1 | Query missing `total` field | Fix — paginators won't know page count |
| MISSING DESC | 10 | 91% undocumented | Add descriptions for maintainability |
| STALE 90D+ | 2 | Not updated in 90+ days | Monitor |
| UPDATED 7D | 1 | Recently active (9 in 30d) | Healthy activity |

### Charts

| Chart | What to look for |
|-------|-----------------|
| **Operation Type Mix** | 100% Query — no mutations or subscriptions saved. |
| **Risk Tier Distribution** | 4 at risk: 18.18% T1 (Broken — won't execute), 18.18% T2 (Quality issues), 63.64% No Risk. Two broken queries need immediate fix. |
| **Iteration Pattern** | 5 queries have only 1 commit (written once, never revised). 3 have 2–3 commits. High iteration suggests refinement; single commits suggest set-and-forget. |
| **Parameter Count Distribution** | Most queries have 0 parameters — they are hardcoded, making them fragile to data changes. |
| **Quality Signals Breakdown** | Missing Description is the dominant issue (10 of 11 queries). |

---

## Tab 8 — SCREEN HTML

**Purpose:** An embedded standalone HTML governance report ("Screen Governance") rendered as a full interactive application within the dashboard. It provides a richer, filter-driven view of the same screen inventory data.

### Layout

The HTML dashboard has two panels:

**Left sidebar:**
- Header: `FUUZ / Screen Governance` — 182 screens scanned (as of scan timestamp)
- **Sections** (navigation links):
  - Overview — summary metrics
  - Risk (9) — screens with risk signals
  - Element Hygiene (280) — all hygiene violations
  - Activity — update/creation activity
  - Inventory — full screen list
  - Drift (298) — screens that have diverged from expected structure
- **Filter Tables:** All · My screens · Recent (7d) · At risk · Active+Undeployed
- **Layout toggle:** Force | Layered (for ERD/graph views within the report)

**Main content area:** Mirrors the SCREENS tab KPI bar and chart panels but in a richer HTML format with better typography. The "Drift" count of 298 is notable — it represents elements or configurations that have drifted from baseline expectations and is not directly surfaced on the SCREENS tab chart.

### When to use vs the SCREENS tab
Use the **SCREENS tab** for a quick at-a-glance review. Use **SCREEN HTML** when you need to filter by ownership (`My screens`), investigate recent changes, or dig into drift details — it provides the same data with more interactive filtering.

---

## Tab 9 — FUUZ ERD

**Purpose:** Explore the complete data model relationship graph for the environment. Useful for understanding schema dependencies before making model changes.

### Interface

| Element | Description |
|---------|-------------|
| **Header stats** | 350 models · 571 relations · 28 inferred relations |
| **Search box** | Search models by name, description, or field name |
| **Model sidebar** | 346 models listed alphabetically — click any to show its 1-hop relation diagram |
| **Canvas** | Interactive graph area — scroll to zoom, drag to pan, Enter to jump to first search match |
| **Layout toggle** | Force (physics-based layout) / Layered (hierarchical layout) |
| **Zoom control** | 100% default with zoom-in/zoom-out buttons and a full-screen expand |
| **"View complete ERD"** button | Renders all 350 models and 571 relations simultaneously — use with caution in large environments |

### When to use
- Before adding a field to a model, check its relations to understand downstream impact.
- When debugging a query that joins multiple models, use the 1-hop view to confirm relation paths.
- For onboarding new developers — the complete ERD provides a map of the full schema.

> **Note:** 28 inferred relations means the platform detected relationships (e.g., from field naming conventions) that are not explicitly declared in the schema. These should be reviewed and formalized or suppressed.

---

## Common Concern Patterns

The following patterns across tabs should trigger a coordinated review:

| Pattern | Tabs | Action |
|---------|------|--------|
| Low deployment rate + high stale count | SCREENS, DATA FLOWS, DATA MODELS, DOCUMENTS | Run a cleanup sprint — archive or deploy stale artifacts |
| T1 risk signals (scripts, hard-delete, destructive flows) | SCREENS, DATA FLOWS, DATA MODELS | Immediate review with the owning developer |
| High "active but undeployed" counts | SCREENS, DOCUMENTS, SAVED SCRIPTS | Align with product team on release schedule |
| Missing descriptions / documentation | SAVED APIS, DATA MODELS | Add descriptions; set a governance rule requiring descriptions on new artifacts |
| Service account dominating change logs | USERS | Confirm automation is intentional and scoped correctly |
| Fork/Combine mismatches in flows | DATA FLOWS | Fix flow graph structure to prevent hung executions |
| Drift count (298) | SCREEN HTML | Investigate using the Drift section in the HTML report |

---

## Footer Reference

The footer bar (visible on all tabs) shows:
- **Last Authenticated At:** `06/12/2026 7:50 AM` — the last time the current user's session was renewed
- **Screen Version:** `0.0.9` — the version of this dashboard screen definition
- **Deploy Date:** `06/14/2026 8:30 AM` — when this version was deployed to this environment
- **Role context:** Supervisor Manager · Fuuz Overflow · Build

If the deploy date is stale relative to known screen changes, the screen may not yet be deployed in this environment.

---

## Screenshots

Screenshots for each tab are in the `/screenshots` directory of this repository:

| File | Tab |
|------|-----|
| `01-users-top.png` | USERS — top (auth charts) |
| `02-users-bottom.png` | USERS — bottom (change charts) |
| `03-screens-top.png` | SCREENS — KPI bar + donut charts |
| `04-screens-mid.png` | SCREENS — density + bloated + trend |
| `05-screens-deployment.png` | SCREENS — deployment health + complexity |
| `06-screens-authors.png` | SCREENS — activity + action-heavy + authors |
| `07-screens-risk-table.png` | SCREENS — risk-bearing table |
| `08-screens-hygiene-table.png` | SCREENS — hygiene + bloated tables |
| `09-dataflows-top.png` | DATA FLOWS — KPI bar + donut charts |
| `10-dataflows-nodes.png` | DATA FLOWS — node mix + modules + integrity |
| `11-dataflows-authors.png` | DATA FLOWS — module group + authors + activity |
| `12-dataflows-risk-table.png` | DATA FLOWS — risk-bearing flows table |
| `13-datamodels-top.png` | DATA MODELS — KPI bar + donut charts |
| `14-datamodels-bottom.png` | DATA MODELS — field + module + author charts |
| `15-documents.png` | DOCUMENTS — full tab |
| `16-savedscripts.png` | SAVED SCRIPTS — full tab |
| `17-savedapis-top.png` | SAVED APIS — KPI bar + charts |
| `18-screenhtml-top.png` | SCREEN HTML — embedded governance report |
| `19-screenhtml-sidebar.png` | SCREEN HTML — sidebar + overview detail |
| `20-fuuzerd.png` | FUUZ ERD — interactive model diagram |
