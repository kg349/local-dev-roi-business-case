# Team Questionnaire — Calibrating the Cost Model

> **Purpose**: replace the placeholder defaults in `cost-model.xlsx` with answers grounded in *how your team actually works*. The questionnaire is grouped by section; each question is tagged with the spreadsheet cell it informs so plugging the answer back in is mechanical.
>
> **Audience**: distribute to your dev team (sections 2–4 + 6), engineering manager / tech lead (sections 1 + 5), and architects / library owners (section 4).
>
> **Companion to**: [`data-gathering-checklist.md`](data-gathering-checklist.md) — that file covers queries you run against Azure DevOps, Jira, HR, Finance. **This file covers what you can only get by asking people.**
>
> **Time investment**: ~60–90 minutes of total team time when run as an async survey. ~2 hours when run as a workshop. The first 8 questions alone calibrate ~70% of the model.

---

## How to use this

Pick one of three workflows depending on how much rigor you want:

| Workflow | Time | Who | When to use |
|---|---|---|---|
| **Quick path** — answer the 8 starred questions below as the tech lead | 30 min, solo | Tech lead / EM | Need a calibrated draft model for tomorrow's meeting |
| **Async survey** — copy questions into a Google Form / OneNote; team fills it in over a week | 90 min team-aggregate | Whole team | Standard rollout; produces defensible numbers |
| **Team workshop** — 90-min meeting, screen-share the spreadsheet, walk through each section live | 1.5 hrs × team_size | Whole team | When you also need team buy-in and shared mental model |

For each answer:

1. Read the **Maps to** field — that's the named range / Inputs row in `cost-model.xlsx`.
2. Drop the answer into that orange cell. The whole model recalculates immediately.
3. Note any "I don't know" answers — they become items for [`data-gathering-checklist.md`](data-gathering-checklist.md).

> **★ Quick-path questions are marked with a star.** Answer just these 8 and you have a credible model.

---

## Section 1 — Team & finance basics (5 min, 1 person)

> **Who**: engineering manager or tech lead.

### 1.1 ★ How many developers are actively on this team?

- *Format*: integer
- *Maps to*: `Inputs!team_size`
- *Typical range*: 4–15
- *Notes*: count only people actively writing code on this product. Exclude shared / fractional contributors unless they spend ≥50% of their time here.

### 1.2 ★ What is the fully-loaded hourly rate for an average developer?

- *Format*: $/hour
- *Maps to*: `Inputs!hourly_rate`
- *Typical range*: $60–$150 depending on geography and seniority mix
- *How to compute*: `(avg salary + benefits + overhead) / (working hours per year)`. Finance usually has a "burdened rate" they use for capex/opex. If unavailable, use `(salary × 1.4) / 2000`.
- *Why it matters*: every dollar number scales with this. If you're unsure, run sensitivity at ±20% on the `Sensitivity` tab.

### 1.3 How many days does a developer typically deliver work per sprint (excluding ceremonies, sick, PTO)?

- *Format*: days per 2-week sprint
- *Maps to*: `Inputs!working_days_per_sprint`
- *Typical range*: 7–9 (out of 10 calendar days)

### 1.4 How many 2-week sprints per year (after holidays/PTO)?

- *Format*: integer
- *Maps to*: `Inputs!sprints_per_year`
- *Typical range*: 22–26
- *Notes*: 26 nominal − ~3 for major holidays − ~1 for company offsites = ~22

---

## Section 2 — What runs locally today (15 min, 1–2 senior devs)

> **Who**: 1–2 devs who've onboarded recently OR are the team's "tooling owners". The goal: enumerate what's currently runnable on `localhost` versus what requires hitting a shared/remote environment.

### 2.1 For each of these services / dependencies, can a developer run it locally on their machine today?

Fill in for each row:

| Component | Locally runnable? (Y/P/N) | If P/N, what's the blocker? |
|---|---|---|
| The main application(s) the team owns | | |
| The team's internal NuGet libraries | | |
| SQL Server / database | | |
| Azure SQL / Cosmos DB | | |
| Azure Service Bus / queues / topics | | |
| Azure Storage (blob, queue, table) | | |
| Azure Functions | | |
| Application Insights / telemetry | | |
| Identity provider (Azure AD / B2C / custom) | | |
| Internal HTTP services owned by *other* teams | | |
| Third-party SaaS dependencies (Twilio, SendGrid, payment, etc.) | | |
| Test data (representative records / seed data) | | |

> **Legend**: **Y** = fully works locally; **P** = partial (mocked / stubbed / different version); **N** = not possible, must use shared env.

- *Maps to*: doc only — this is the **input for scoping the docker-compose / emulator set-up**. Specifically informs `setup_engineer_hours` (`Inputs!B74`) and `apply_docker_license_cost` (`Inputs!B79`).
- *Why it matters*: the more "N" rows, the higher the Docker/emulator setup cost AND the larger the realised savings.

### 2.2 ★ What % of the team's typical feature work *cannot* be verified locally end-to-end today?

- *Format*: percentage (0–100)
- *Maps to*: `Inputs!pct_env_or_integration_related` (frame as "% of bugs that are environment/integration-related")
- *Typical range*: 30–60% — most teams without emulators land here
- *Notes*: a feature counts as "cannot verify locally" if a real bug in it would only manifest when hitting real Azure services / other teams' APIs / real data.

### 2.3 For features that can't be verified locally, what's the developer's current Plan B?

Check all that apply (this list informs the `Workarounds` tab):

- [ ] Push to a shared dev environment and check there
- [ ] Use a personal/shadow Azure subscription
- [ ] Skip integration tests; ship and check in dev environment
- [ ] Hardcode connection strings to shared dev resources
- [ ] Mock the dependency in code (loses real-API coverage)
- [ ] Disable auth/security locally to bypass it
- [ ] Paired debugging with another dev who has a working environment
- [ ] Comment-out-and-redeploy to narrow down the bug
- [ ] Other: ___________

- *Maps to*: `Workarounds` tab — flag Y/P/N for each one observed in this team.

### 2.4 How much developer machine spec do we have today?

- *Format*: CPU cores, RAM, SSD
- *Typical baseline*: 8-core i7/Ryzen, 16GB RAM, 500GB SSD
- *Maps to*: `Inputs!machine_upgrade_cost` (`Inputs!B77`) — if devs need ≥32GB to run Docker + emulators comfortably, budget upgrade cost
- *Notes*: Azure emulators + Cosmos + Service Bus + Storage emulator + IDE + browser typically wants **32GB RAM minimum**, ideally 64GB for happy multitasking.

### 2.5 Do we have an existing organisation-level Docker policy?

- [ ] Yes — Docker Desktop is approved and licensed
- [ ] Yes — but only Docker Engine / Rancher / Podman is approved (no Docker Desktop)
- [ ] No — would need approval
- [ ] Unknown

- *Maps to*: `Inputs!apply_docker_license_cost` (1 if Docker Desktop license needed, 0 if free alternative or <250-employee org exempt)

---

## Section 3 — The build / PR loop (15 min, 1 recent shipper)

> **Who**: a developer who's merged ≥5 PRs in the last sprint. Walk them through their last PR.

### 3.1 ★ Across the entire team, what's the median number of PRs per work item?

- *Format*: number (can be fractional, e.g., 1.8)
- *Maps to*: already auto-computed in `Real-Bug-Data!real_avg_prs_per_rework` from your real data (3.04 for rework items). For the **full** team, including no-rework items, you may want to compute this directly.
- *Helpful sub-questions*:
  - In the last 30 work items, how many shipped in exactly 1 PR? In 2? In 3+?
  - For the multi-PR items, was it driven mostly by reviewer feedback or by bugs?
- *Notes*: anything ≥1.5 indicates significant rework that the model captures.

### 3.2 ★ How many distinct build pipelines fire per PR on average?

- *Format*: integer
- *Maps to*: `Inputs!builds_per_dev_per_day` indirectly — if 6 pipelines fire per PR and devs open ~1 PR/day, builds_per_dev_per_day ≈ 6.
- *Helpful sub-questions*:
  - For a PR touching just one project, how many pipelines fire? (typically 1–3)
  - For a PR touching shared libraries / `Common.Services` etc., how many fire? (often 8–20)
  - Do all pipelines run on every PR, or only ones affected by changed files?
- *Notes*: from your `Real-Build-Times` tab there are 24 build entries; learning *which subset* fires per typical PR is the missing piece.

### 3.3 What's the median wall-clock time from PR-push to "all pipelines green"?

- *Format*: minutes
- *Maps to*: `Inputs!pipeline_duration_min` (currently overridden by `real_critical_path_min` when `use_real_data=1`).
- *Notes*: confirm whether the "37 min critical path" from `Real-Build-Times` matches what developers actually wait — they may report longer due to queue time, retries, or sequential pipeline dependencies.

### 3.4 What's the typical queue / wait time before a pipeline *starts*?

- *Format*: minutes
- *Maps to*: `Inputs!queue_wait_min`
- *Typical range*: 0–10 min (more for self-hosted agents at peak hours)
- *Notes*: with 5 MS-hosted + 50 self-hosted agents, queue should be near-zero except at end-of-sprint crunch.

### 3.5 What fraction of pipeline runs FAIL on first attempt?

- *Format*: percentage (0–100)
- *Maps to*: `Inputs!pipeline_failure_rate`
- *Typical range*: 20–40% (DORA "change-failure rate" benchmark)
- *Helpful breakdown*:
  - Pure flaky-test failures (not real bugs): ___%
  - Build/dependency failures (e.g., NuGet restore broken): ___%
  - Actual code bugs caught by CI: ___%
  - QA-test failures requiring a real fix: ___%

### 3.6 Of the pipeline failures above, what % could realistically have been caught if the developer had a working local environment?

- *Format*: percentage (0–100)
- *Maps to*: `Inputs!pct_retries_caused_by_local`
- *Typical range*: 30–60%
- *Notes*: this is a judgement call. Lower if most failures are flaky tests; higher if they're "the code didn't compile against the real X dependency" types of failures.

### 3.7 How long does it take to review a peer's PR (median, from "I see it" to "I leave a comment / approve")?

- *Format*: minutes
- *Maps to*: `Inputs!reviewer_review_min`
- *Typical range*: 15–45 min

### 3.8 How long is the wall-clock wait from PR-open to first reviewer activity?

- *Format*: hours
- *Maps to*: `Inputs!review_wait_hours`
- *Typical range*: 1–8 hours

### 3.9 Average review rounds per bug-fix PR (how many "push more changes after feedback" cycles)?

- *Format*: number (e.g., 1.5)
- *Maps to*: `Inputs!review_rounds`
- *Typical range*: 1.0–2.5

### 3.10 How many code-fix iterations does a developer typically do per hour when actively coding?

- *Format*: count (edit → run → check)
- *Maps to*: `Inputs!iterations_per_dev_per_day` (× 6–7 productive hours/day)
- *Typical range*: 8–15 iterations per hour for tight inner loops
- *Notes*: this is hard to estimate. Try observing one dev for a single coding session.

### 3.11 ★ Today, how long does one "edit → run → see result" iteration take?

- *Format*: minutes (median)
- *Maps to*: `Inputs!cycle_time_today_min`
- *Typical range*: 5–60 min when most verification requires a shared environment

### 3.12 With a working local environment, how long *would* the same iteration take?

- *Format*: minutes (median)
- *Maps to*: `Inputs!cycle_time_target_min`
- *Typical target*: 0.5–3 min

---

## Section 4 — NuGet inner loop (10 min, 1 dev who has worked on shared libs recently)

> **Who**: a developer who's edited a shared internal library (e.g., `Common.Services`, `DAL.Services`) in the last sprint.

### 4.1 ★ How often does a developer trigger a full local NuGet rebuild on a typical workday?

- *Format*: number (rebuilds per dev per day)
- *Maps to*: `Inputs!nuget_rebuilds_per_dev_per_day`
- *Typical range*: 1–5
- *Triggers to count*: `git pull` from development, branch switch, NuGet feed refresh, "my consumer code suddenly can't see the new method"
- *Default in model*: 3

### 4.2 How long does the local NuGet rebuild PowerShell script take in your environment?

- *Format*: minutes (wall-clock)
- *Maps to*: `Inputs!nuget_rebuild_min`
- *Source*: real team data = **6 min** (measured). Confirm this still holds or update.

### 4.3 Of the team's actively-developed internal libraries, how many would benefit from ProjectReference (in-solution, no external consumers)?

- *Format*: integer
- *Notes*: doc only — feeds the scoping conversation. List them by name in the answer.

### 4.4 How many of those libraries DO have external consumers (other teams, customers, SDK) that depend on the published NuGet?

- *Format*: integer
- *Maps to*: doc only — these are the libraries that stay PackageReference (or go hybrid).
- *Notes*: list by name.

### 4.5 During an active "feature touching a shared library" task, how many times does a developer iterate (edit-pack-restore-rebuild) before the change is correct?

- *Format*: integer
- *Maps to*: `Inputs!nuget_pkgs_per_dev_per_cross_pkg_feature`
- *Typical range*: 5–20
- *Default in model*: 12

### 4.6 How many features per year does each developer typically work on that touch a shared library?

- *Format*: integer (per dev per year)
- *Maps to*: `Inputs!cross_pkg_features_per_dev_per_year`
- *Typical range*: 10–30
- *Notes*: think "how often does my work require editing `Common.Services` or `DAL.Services`?"

### 4.7 If we converted in-solution consumers to ProjectReference, what % of the NuGet-rebuild friction would *go away*?

- *Format*: percentage (0–100)
- *Maps to*: `Inputs!pct_nuget_eliminable_with_project_refs`
- *Default in model*: 70%
- *Notes*: not 100% because release pipelines still build packages, and external-consumer libs stay PackageReference.

### 4.8 Rough estimate of refactor effort to migrate in-solution consumers to ProjectReference?

- *Format*: developer-hours, one-time
- *Maps to*: `Inputs!project_ref_migration_one_time_hours` × `project_ref_migration_team_size`
- *Default in model*: 80 hours × 2 devs = 160 hours total
- *Notes*: this is highly solution-dependent. Run a 1-day spike on a single library to validate.

### 4.9 What's the current full-solution build time (Visual Studio cold open → green)?

- *Format*: minutes
- *Notes*: doc only — used to check that converting to ProjectReference won't push solution-build-time past ~5 min. If it does, the mitigation is solution filters (`*.slnf`), not avoiding ProjectReference.

### 4.10 Does the team currently use SLN filters (`*.slnf`), or load the entire solution every time?

- [ ] Use SLN filters
- [ ] Load whole solution every time
- [ ] Don't know what SLN filters are
- *Notes*: doc only — answers what's needed if solution build time grows.

---

## Section 5 — Bug volume and workflow reality (15 min, manager + ticket survey)

> **Who**: engineering manager. Pull the last 6 months of bug work items in Azure Boards as you answer.

### 5.1 ★ How many bugs does the team file per year?

- *Format*: integer (annualised)
- *Maps to*: `Inputs!bugs_per_year`
- *How to estimate*: `(bugs filed in last 90 days) × 4`. If your `Found In` field isn't populated, this still works.
- *Default in model*: 400

### 5.2 ★ Of the bugs filed in the last 6 months, where were they *first detected*? (Use the `Found In` field if populated; otherwise estimate.)

| Detection stage | Count / % | Maps to |
|---|---|---|
| Caught locally by the dev before pushing | __ / __% | `pct_today_local` |
| Caught in the Development environment (CI, QA testing, etc.) | __ / __% | `pct_today_dev` |
| Caught in Staging / UAT | __ / __% | `pct_today_stg` |
| Caught in Production (customer reports, monitoring) | __ / __% | `pct_today_prod` |

- *Default in model*: 35% / 45% / 15% / 5%
- *Notes*: the `data-gathering-checklist.md` has the exact WIQL query to get this from ADO.

### 5.3 For a typical Development-environment bug, what fraction get caught **before QA handoff** vs **after QA handoff**?

- *Format*: percentage split
- *Maps to*: `Inputs!pct_rework_caught_pre_qa`
- *Default in model*: 60% caught pre-QA (the developer's own re-runs catch them), 40% caught by QA
- *Why it matters*: pre-QA rework redoes ~35 of the 46 process steps; post-QA rework redoes ~40. This input governs the blended redo factor.

### 5.4 How many work items in the last 4–6 months required more than one PR to land?

- *Format*: integer
- *Maps to*: confirms the `Real-Bug-Data` tab is current. From your file: 26 rework items in 4.25 months.
- *Notes*: if the number is changing fast, refresh the data quarterly.

### 5.5 What's the team's perception of "average PRs per work item that needed rework"?

- *Format*: number
- *Maps to*: real_avg_prs_per_rework (auto-computed = 3.04 from your data)
- *Notes*: comparing perception vs measured is informative — if perception is much higher than measured, devs are *remembering* the worst cases.

### 5.6 In the last 6 months, how many work items came back from production (escaped defects)?

- *Format*: integer
- *Maps to*: helps validate `pct_today_prod`
- *Notes*: a production escape is the most expensive bug; even one informs the 100× multiplier debate.

### 5.7 How often does a bug found in the Development environment block other developers' work?

- [ ] Rarely (separate stacks)
- [ ] Sometimes (1–2 devs per bug)
- [ ] Often (most of the team affected)

- *Maps to*: the "team contention surcharge" line in `Development-Fix-Workflow` tab (currently assumes 0.25 hr × (team − 1)).

---

## Section 6 — Workarounds self-assessment (10 min, full team async)

> **Who**: every developer on the team. Send as a Google Form or fill in as a workshop.

For each workaround, mark **Y** (yes, I do this regularly), **P** (sometimes / partial), or **N** (no, I don't / don't need to).

| # | Workaround | Y / P / N | Approx hours/week if Y |
|---|---|---|---|
| 1 | Sharing one remote dev/test environment with the whole team | | |
| 2 | Hardcoded connection strings to Azure dev resources in my local config | | |
| 3 | Disabling auth / security checks to make local code "kind-of" work | | |
| 4 | "Push-to-test" debugging (printf statements via the deployed env) | | |
| 5 | Mocking Azure services in code instead of using emulators | | |
| 6 | Maintaining a personal/shadow Azure subscription out of pocket | | |
| 7 | Long-lived feature branches because the integration cycle is slow | | |
| 8 | Comment-out-and-redeploy to narrow down a bug | | |
| 9 | Pair-debugging on someone else's shared environment | | |
| 10 | Skipping integration tests locally — only running unit tests | | |
| 11 | Manual data-prep scripts to seed shared databases | | |
| 12 | Long onboarding (1–2 weeks of "wait for access") for new hires | | |
| 13 | "Your turn / QA" handoffs because verification takes too long | | |
| 14 | Regression suite disabled locally (only runs in CI) | | |

- *Maps to*: `Workarounds` tab — total hours/week sums into `workaround_hours_per_week_total` (`Inputs!B68`)
- *Default in model*: 12 hrs/wk total team time across all 14 workarounds (i.e., 1.5 hrs/dev/wk for 8 devs)
- *Notes*: aggregate by summing each "hours/week" column across all respondents.

---

## Section 7 — Open-ended (10 min, anyone)

> **Who**: anyone with an opinion. Useful for the deck's "voice of the team" quotes.

### 7.1 Describe the most recent feature where the lack of a local environment slowed you down the most. What specifically happened, and how long did it cost you?

*Free text*. This becomes a powerful slide.

### 7.2 If a magic wand gave you a fully working local environment tomorrow, what's the first thing you'd do differently?

*Free text*.

### 7.3 What's your single biggest objection / risk to adopting Docker + emulators?

*Free text*. The deck should address these head-on.

### 7.4 Is there a single library you're aware of that would benefit most from ProjectReference? Why?

*Free text*. Becomes the pilot candidate.

---

## Section 8 — Calibration sanity checks

After plugging in answers, sanity-check the model:

| Check | What "ok" looks like |
|---|---|
| `total_annual_savings` < team's annual fully-loaded payroll | Should always be < 100% of payroll. Sweet spot 10–30%. If higher, something's double-counting. |
| Payback period | Healthy: 3–9 months. <2 months suggests inflated savings; >12 months suggests low confidence in inputs. |
| Sprint capacity erosion % (on `Cycle-Time-Sprint` tab) | 15–40% is plausible. >50% means a numerical input is wrong (probably `iterations_per_dev_per_day` × `cycle_time_today_min`). |
| `Defect-Distribution-PCE!E<weighted-today>` weighted defect cost | Should be in the 5–30 unit range. Higher means the production defect % is unrealistically high. |
| `Nuget-vs-ProjectRef!B<total>` annual NuGet tax | Should be in the $10K–$100K range for an 8-dev team. Outside this, re-check `nuget_rebuilds_per_dev_per_day`. |

If any of these fail, that's a flag to re-examine the inputs that drive them, not to publish a different number.

---

## Quick-path summary (the 8 starred questions)

If you're really tight on time, fill in just these:

| # | Question | Input | Default |
|---|---|---|---|
| 1.1 | Team size | `team_size` | 8 |
| 1.2 | Loaded hourly rate | `hourly_rate` | $85 |
| 2.2 | % features unverifiable locally | `pct_env_or_integration_related` | 35% |
| 3.1 | Median PRs per work item | `real_avg_prs_per_rework` | 3.04 (real data) |
| 3.2 | Build pipelines per PR | informs `builds_per_dev_per_day` | 6 |
| 3.11 | Today's cycle time | `cycle_time_today_min` | 25 |
| 4.1 | NuGet rebuilds per dev per day | `nuget_rebuilds_per_dev_per_day` | 3 |
| 5.1 | Bugs per year | `bugs_per_year` | 400 |

These 8 inputs drive >70% of the headline ROI. Everything else moves the number by <5%.

---

## Where to put the answers

1. **For the spreadsheet inputs**: open `cost-model.xlsx`, navigate to the `Inputs` tab, find the orange cell with the matching named range, type the value in. The whole workbook recalculates on every change.
2. **For free-text / open-ended answers**: paste into a `team-survey-results.md` (gitignored if you want to keep them internal) so they're available for the deck's "voice of the team" slides.
3. **For tally-style answers (workarounds)**: aggregate by summing each respondent's `hours/week` column, then drop the sum into `Inputs!workaround_hours_per_week_total`.
4. **Re-run** `python cost-model.py` if you want a refreshed CSV / Python summary. The Excel file picks up Input changes live without re-running the script.

---

## How this connects to the other docs

- **[`data-gathering-checklist.md`](data-gathering-checklist.md)** covers what you can pull from Azure DevOps / Jira / Finance via queries. *Use that AND this in tandem* — they complement each other (queries for what tools know; questionnaire for what people know).
- **[`developer-workarounds.md`](developer-workarounds.md)** describes each of the 14 workarounds in detail. Use it to expand context on Section 6 if the team asks "what counts as workaround #3?"
- **[`nuget-vs-project-references.md`](nuget-vs-project-references.md)** is the deeper write-up of Section 4's findings. Share that with library owners after gathering Section 4 answers.

> Once Sections 1–5 are filled in, you have everything you need to replace placeholder numbers with team-grounded numbers and present the model with confidence.
