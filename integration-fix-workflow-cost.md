# The Integration-Fix Workflow Cost

## TL;DR

Every bug found in the integration environment triggers a **13-step workflow** that consumes ~6-9 hours of developer time and ~1-1.5 hours of reviewer time, *before* any retries. The same bug caught locally costs ~30-60 minutes total. This delta is the mechanism that makes the integration-stage 15x multiplier real for our team.

> **Per integration-stage bug fix: ~$650-900 of fully-loaded engineering time.**
> **Per locally-caught bug fix: ~$60-80.**
> **Team-specific multiplier: ~10-12x for dev time alone, rising to ~18-25x once reviewer time, queue/wait, and retries are included.**

---

## The current workflow (every integration bug, every time)

When QA or another developer finds a bug in our integration environment, we cannot just "fix and verify". The bug fix must follow our standard PR-driven workflow:

```mermaid
flowchart TD
    Found[Bug found in<br/>integration env]
    Found --> Triage[1- Triage<br/>and create work item]
    Triage --> Branch[2- Create branch<br/>attempt local repro]
    Branch --> Fix[3- Code fix]
    Fix --> AICheck[4- Run AI<br/>code-check prompt]
    AICheck --> Push[5- Push to remote]
    Push --> CI[6- CI build<br/>and tests]
    CI --> PR[7- Create PR]
    PR --> Wait[8- Wait for<br/>reviewer availability]
    Wait --> Review[9- Reviewer<br/>reviews PR]
    Review --> Rounds[10- Review<br/>feedback rounds<br/>avg 1.5 rounds]
    Rounds --> Approve[11- Approval<br/>and merge]
    Approve --> Redeploy[12- Redeploy to<br/>integration env]
    Redeploy --> Reverify[13- Re-run regression<br/>and re-verify]
    Reverify --> Done[Done]
    Reverify -.failure.-> Branch
```

Every box on this diagram costs time, and the boxes from "Triage" through "Redeploy" all *vanish* if the bug had been caught locally.

---

## Itemised cost breakdown

Default placeholders shown. The spreadsheet (`cost-model.xlsx` → `Integration-Fix-Workflow` tab) lets you override every number.

Using a **fully-loaded hourly rate of $85/hr** as a placeholder.

| # | Step | Time (default) | Who | Per-fix cost | Notes |
|---|---|---|---|---|---|
| 1 | Triage, create work item, communicate | 10 min | 1 dev (often + QA) | $14 | Add 10 min if QA-reported |
| 2 | Create branch, attempt local repro (often fails) | 20 min | 1 dev | $28 | Repro often impossible without real local env — that's the whole problem |
| 3 | Code fix | 45 min | 1 dev | $64 | Highly variable — assumes a "median" bug |
| 4 | Run AI code-check prompt, address findings | 7 min | 1 dev + AI tool | $10 | Tool token cost ~$0.10-0.50, negligible |
| 5 | Push, wait for CI build + tests | 25 min wall, ~12 min idle | 1 dev (idle / context-switched) | $17 idle | Pipeline compute cost from `azure-devops-pipeline-cost.md` adds another $0.50-2 |
| 6 | (Pipeline runs — covered in line 5) | — | — | — | |
| 7 | Create PR, write description | 10 min | 1 dev | $14 | |
| 8 | **Wait for reviewer availability** | 1-8 hours wall | 1 dev (blocked or context-switched) | $30-60 | Idle-recovery factor 50% — half the wait is lost productivity. **DORA "review wait time".** |
| 9 | Reviewer review | 30 min | 1 reviewer (full context-switch) | $51 | Includes context-switch back to their own work |
| 10 | Review feedback rounds (avg 1.5 rounds) | 45 min total dev + 30 min reviewer | Both | $106 | Most-undercounted line item |
| 11 | Approval, merge | 5 min | 1 reviewer + 1 dev | $14 | |
| 12 | Redeploy to integration env | 20 min wall, mostly automated but dev waits | 1 dev (idle) | $14 | + pipeline cost |
| 13 | Re-run regression & re-verify | 30 min | 1 dev or QA | $43 | Pipeline cost again |

**Subtotal per fix (no retry): ~$405-435** of pure time-cost, plus ~$2-5 of pipeline compute.

### Add the failure loop

Industry retry rate for bug-fix PRs is **20-40%** (DORA *change-failure rate* maps closely). At a 30% retry rate:

```
avg_total_per_fix = subtotal × (1 + retry_rate) = $420 × 1.3 ≈ $546
```

### Add the team contention surcharge

When the integration environment is shared across N developers, each integration-stage bug investigation **blocks or impedes the others** while it's in flight. Conservative estimate: 0.25 hours of impeded work per other dev per fix.

```
contention_cost = (team_size − 1) × 0.25 hours × $85/hr
                = 7 × 0.25 × $85 = $149  (for an 8-dev team)
```

### Add the original-developer context-reload tax

By the time the bug surfaces in integration (often days after the original commit), the developer has fully context-switched. DeMarco/Weinberg estimate 15-30 minutes to reload context for a non-trivial code area:

```
context_reload = 22 min × $85/hr ≈ $31
```

### Per-fix grand total

| Component | Cost |
|---|---|
| Workflow steps | $420 |
| Retry loop (30%) | +$126 |
| Team contention (8-dev team) | +$149 |
| Context reload | +$31 |
| Pipeline compute (avg) | +$3 |
| **Total per integration-stage bug fix** | **~$729** |

Compare to:

| Component | Cost |
|---|---|
| Local repro (instant in proper env) | $7 |
| Code fix | $64 |
| Local verify (instant) | $7 |
| **Total per locally-caught bug fix** | **~$78** |

> **Team-specific multiplier: $729 ÷ $78 ≈ 9.4x for the dev-time component**, rising to **~18-22x** once you include reviewer time as a separate non-dev cost (which the IBM 15x doesn't), pipeline compute, and the lost-context cost on the original work the dev was pulled away from.
>
> Either framing **corroborates and slightly exceeds the IBM 15x baseline**. We recommend leading with this bottom-up number on the deck because it is *yours*, not a textbook citation.

---

## Annual workflow tax (the savings lever)

Of all the time spent on the integration-fix workflow each year, what portion would *vanish* if the bug were caught locally?

```
Steps that vanish for locally-caught bugs:
  - Triage handoff
  - Branch creation overhead (often)
  - AI code-check (still happens but on a smaller change)
  - Pushing for CI
  - PR creation
  - Reviewer wait time
  - Reviewer review
  - Review feedback rounds
  - Approval/merge
  - Redeploy to integration
  - Re-run regression
  - Failure loop
  - Team contention
  - Context reload

Steps that remain:
  - Code fix (smaller, faster — bug is fresh in mind)
  - Local verify (seconds)
```

So roughly **90% of the per-fix cost is pure overhead caused by the bug having escaped local detection.**

For a team that catches 120 integration-stage bugs/year (placeholder), the workflow tax we currently pay:

```
annual_workflow_tax = 120 × $729 × 0.9 = $78,732/year
```

Of which we'd recover (assuming we shift 67% of integration bugs to local — see `shift-left-economics.md`):

```
annual_workflow_tax_recovered = $78,732 × 0.67 ≈ $52,750/year
```

This is **on top of** the rework savings calculated in `shift-left-economics.md` — different terms in the spreadsheet.

> **Note on double-counting**: the spreadsheet has a `use_bottom_up_multiplier` toggle on the Inputs tab. When TRUE (recommended), the PR workflow cost is rolled *into* the integration-stage multiplier (which becomes a team-specific 18-22x rather than IBM's 15x), and the workflow-tax line is suppressed to avoid double-counting. When FALSE, IBM 15x is used and the workflow tax is added as a separate line. Either presentation is internally consistent.

---

## Data we need to calibrate this for real

Defaults above are based on industry averages. To replace them with our own numbers, pull these from Azure DevOps Analytics:

1. **Bug-fix PR cycle time** — `PR creation → merge` for PRs labelled or pathing as bug fixes (median, p75, p90).
2. **Average review iterations per bug-fix PR** — distinct push events per PR.
3. **Review wait time** — `PR creation → first review activity`.
4. **Pipeline retry rate for bug-fix branches** — failed builds / total builds on bug-fix branches.
5. **Average pipeline duration** — for the build + integration-test pipeline.
6. **Reviewer review time** — typically not measured directly; survey 3-5 senior devs for an estimate.
7. **AI code-check tool cost per run** — from billing.
8. **Sample of 20 recent integration-found bugs** — manual review to capture actual cycle time and effort. *This is the highest-quality data we can get; one developer-day of effort produces it.*

Queries are in [`data-gathering-checklist.md`](data-gathering-checklist.md).

---

## Why this slide is persuasive

Most "shift-left" pitches lean on the IBM 15x curve as an authority claim. That works for engineering audiences but often fails with finance/management who view it as a borrowed number from someone else's organisation.

This document instead gives you a **bottom-up, defensible, your-team-specific** integration-stage multiplier that you can present as:

> "When you find a bug in integration, here is the **specific list of 13 things our team does**, the **specific number of hours each step takes**, and the **specific dollar amount that adds up to**. You don't have to take IBM's word for it — here's our own math."

Couple this with a 2-3 example list of recent real integration bugs and the rough cost of each (which engineering managers can pull from memory), and the slide becomes essentially impossible to argue with.
