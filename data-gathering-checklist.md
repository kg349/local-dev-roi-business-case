# Data-Gathering Checklist

## TL;DR

This checklist replaces the placeholder numbers in `cost-model.xlsx` with real numbers from Azure DevOps, Jira, HR, and Finance. **Budget about 4-6 hours total of analyst/manager time, plus 1 week of passive developer time-tracking, to produce a fully calibrated model.**

The order below is roughly highest-leverage to lowest. If you can only do the first three, the model is already credible.

---

## 1. Defect distribution from Azure Boards (HIGHEST PRIORITY)

This is the input that drives Pillar 1 (shift-left rework cost) and is the single most important data point.

### Required: a `Found In` field on Bug work items

Most Azure DevOps Bug templates have a "Found In" field — confirm yours does. If not, ask the process admin to add a custom field with values matching our four environments: `Local`, `Development`, `Staging`, `Production`. Going forward devs/QA fill it in when filing bugs.

> Note: we deliberately do **not** include "CI" as a separate `Found In` value. Our CI pipeline runs but does not gate deployment, so a CI-flagged bug operationally reaches the Development environment where it's actually caught/fixed. Counting it as "Development" matches the cost it actually incurs.

### WIQL query — defects by detection stage (last 180 days)

In Azure DevOps: *Boards → Queries → New Query → Editor → switch to WIQL editor*.

```sql
SELECT
    [System.Id],
    [System.Title],
    [Microsoft.VSTS.Common.FoundIn],
    [System.CreatedDate],
    [System.AreaPath]
FROM WorkItems
WHERE
    [System.WorkItemType] = 'Bug'
    AND [System.CreatedDate] >= @Today - 180
    AND [System.AreaPath] UNDER 'YourProject\YourTeamArea'
ORDER BY [System.CreatedDate] DESC
```

### Pivot in Excel (or use the Analytics view directly)

```
Stage          Count    %
Local          ___      ___
Development    ___      ___
Staging        ___      ___
Production     ___      ___
TOTAL          ___      100%
```

Plug these % values into the **Defect-Distribution-PCE** tab of `cost-model.xlsx`.

### Fallback if `Found In` isn't populated

Look at:

- **Tags** (`#caught-in-staging`, `#prod-incident`)
- **Linked work items** — Bugs linked to deployment work items hint at stage
- **State transition dates** — Bugs created within hours of a release suggest production discovery

Or, sample 30 random bugs and manually classify them — extrapolate from the sample.

---

## 2. PR cycle time for bug-fix PRs

Drives Pillar 2 (development-fix workflow cost).

### KQL query (Azure DevOps Analytics)

In Azure DevOps: *Project Settings → Analytics → Open in Power BI* (or query Analytics OData).

OData query:

```
https://analytics.dev.azure.com/{org}/{project}/_odata/v3.0-preview/PullRequests
?$filter=
    Status eq 'completed' and
    CompletedDate ge 2025-10-01Z and
    contains(Title, 'fix') or contains(Title, 'bug')
&$select=
    PullRequestId,
    Title,
    CreatedDate,
    CompletedDate,
    SourceBranch,
    TargetBranch
```

In Power BI / Excel, compute:

```
cycle_time_hours = (CompletedDate - CreatedDate) in hours
```

Then take the median (p50), p75, and p90.

```
p50 cycle time: ___ hours
p75 cycle time: ___ hours
p90 cycle time: ___ hours
```

Plug into Inputs tab.

### Alternative: az CLI

```bash
az repos pr list --status completed --top 200 \
   --query "[?contains(title, 'fix') || contains(title, 'bug')].[pullRequestId, title, creationDate, closedDate]" \
   -o tsv > prs.tsv
```

Then process in Excel/Python.

---

## 3. Pipeline duration, failure rate, queue wait

Drives Pillar 4 (Azure DevOps pipeline cost).

### From Azure DevOps Analytics

Power BI report or OData query for `PipelineRuns`:

```
https://analytics.dev.azure.com/{org}/{project}/_odata/v3.0-preview/PipelineRuns
?$filter=
    CompletedDate ge 2025-10-01Z
&$select=
    PipelineRunId,
    PipelineId,
    Result,
    QueuedDate,
    StartedDate,
    CompletedDate
```

Compute:

```
queue_wait_min = (StartedDate - QueuedDate) in minutes
duration_min = (CompletedDate - StartedDate) in minutes
failure_rate = count(Result = 'failed') / count(*)
```

Aggregate by pipeline:

```
Pipeline name              Median duration   p95 queue wait   Failure rate
build-and-test             ___ min            ___ min          ___%
nightly-regression         ___ min            ___ min          ___%
```

### From Azure Portal

Azure DevOps → Pipelines → Analytics → "Pipeline pass rate" and "Pipeline duration" reports give these directly.

### Parallel job count and cost

Org admin → Billing → Azure DevOps services. Look for "Parallel jobs" line items. Note the number of:

- Microsoft-hosted parallel jobs (after the free 1)
- Self-hosted parallel jobs (after the free 1)

```
Microsoft-hosted parallel jobs: ___ × $40 = $___ /month
Self-hosted parallel jobs:      ___ × $15 = $___ /month
```

---

## 4. Loaded hourly rate from Finance

Drives every dollar conversion in the model. **The single highest-leverage number to confirm with Finance.**

Ask Finance:

> *"What's our fully-loaded developer hourly rate, including base salary, benefits, employer taxes, equipment, software, and allocated overhead? We need this to model a process improvement ROI."*

Typical components:

| Component | Typical % of base |
|---|---|
| Base salary | 100% |
| Benefits (health, retirement, PTO) | +25-35% |
| Employer taxes | +7-9% |
| Equipment + software | +3-5% |
| Allocated overhead (real estate, IT, mgmt) | +15-25% |
| **Loaded multiplier** | **1.5x - 1.8x base** |

```
Confirmed loaded hourly rate: $___ /hour
```

If Finance won't disclose, derive: `(median_dev_total_comp × 1.4) / 2080`

---

## 5. Headcount, sprint cadence

Trivial but easy to get wrong.

```
Active developers on this team: ___
Sprint length (days): ___
Working days per sprint per dev: ___ (typically 9 for 2-week sprints)
Sprints per year: ___ (typically 24-26)
```

---

## 6. Bug volume and severity mix

```
Total bugs created in last 180 days: ___
Annualised: ___ × (365/180) = ___ bugs/year
% Sev 1 (critical): ___%
% Sev 2 (high): ___%
% Sev 3 (medium): ___%
% Sev 4 (low): ___%
```

The model uses average defect cost; if your team has very different fix-time profiles by severity, edit the **Bug-Cost-by-Stage** tab to weight them separately.

---

## 7. The 1-week developer time-tracking template

This is the highest-quality data we can produce — direct measurement of inner-loop cycle time. Worth one developer-week of effort.

### Instructions to give the team

> *"For one week, every time you make a code change and want to verify it (run tests, check behaviour, debug), record:*
>
> 1. *Timestamp when you initiate the verify step*
> 2. *Timestamp when you have the result*
> 3. *Where the verification happened: Local / Development env / Staging*
> 4. *Outcome: Worked / Broke / Inconclusive*
>
> *Use a simple spreadsheet or even a Slack thread. Don't overthink — count anything that's a 'verify the change' moment."*

### Template

| Date/time start | Date/time end | Cycle min | Where | Outcome | Notes |
|---|---|---|---|---|---|
| 2025-04-21 09:14 | 09:38 | 24 | Development | Worked | Auth fix |
| 2025-04-21 10:02 | 10:06 | 4 | Local (just unit test) | Worked | |
| ... | ... | ... | ... | ... | ... |

### Compute at end of week

```
Total cycle-min logged: ___
Number of verify events: ___
Median cycle time: ___ min
% verifications that happened in Development or beyond: ___%
% in Local or unit-test only: ___%
```

The median cycle time is the input to the **Cycle-Time-Sprint** tab.

---

## 8. Code review effort (often surveyed, not measured)

There's no Azure DevOps query for "how long did it take a reviewer to review a PR" — the platform doesn't capture this. Survey 5-10 senior developers:

> *"For a typical bug-fix PR (small change, ~50-200 LOC):*
>
> 1. *How long does it take you to review it (focused, uninterrupted time)?*
> 2. *On average, how many rounds of feedback before approval?*
> 3. *On average, how long after a PR is opened do you actually pick it up?"*

```
Median reviewer review time per round: ___ min
Median feedback rounds per bug-fix PR: ___
Median wait time for first review: ___ hours
```

---

## 9. Workaround self-assessment (from the team)

Run [`developer-workarounds.md`](developer-workarounds.md)'s checklist as a 30-minute team meeting.

Result: count of workarounds in use + total hours/week impact + total annual cost.

```
Workarounds we currently rely on: ___ of 14
Total hours/week impact: ___
Annual cost: $___
```

---

## 10. Defect-to-prod incident impact (qualitative)

Not strictly required, but if you have any of:

- A SEV-1 from the last year that escaped to production
- A customer-facing incident
- A security incident

...then a single bullet on the deck saying *"this incident was caused by a defect that should have been caught locally"* is worth more than ten slides of math. Write 1-2 sentences for each.

---

## Priority order

If you can do only:

| # | Effort | What you get |
|---|---|---|
| **1 hour** | Run query #1 (defect distribution) | A real PCE %; the model becomes defensible |
| **2 hours** | Add #4 (hourly rate) and #5 (headcount) | Real dollar numbers, not placeholder ones |
| **4 hours** | Add #2 (PR cycle time) and #3 (pipeline) | The development-fix workflow tab and the pipeline tab use real numbers |
| **1 week** | Add #7 (developer time-tracking) | The cycle-time number is *measured*, not estimated |
| **6-8 hours** | Add #8, #9, #10 | The deck has texture and credibility |

---

## A note on querying Azure DevOps

If you don't have analyst access, two paths:

1. Ask a team-admin to run the queries and share the results.
2. Use the built-in Azure DevOps Analytics Views in Boards/Pipelines — most of the queries above are also available as one-click reports.

For Jira-backed teams: use JQL equivalents. The substantive data is the same; the syntax differs. Atlassian Analytics or eazyBI plugins both expose the necessary fields.
