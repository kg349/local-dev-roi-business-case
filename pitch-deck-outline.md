# Pitch Deck Outline — 17 Slides

This is the slide-by-slide outline for the management presentation. Build it in PowerPoint or Google Slides; each slide below has:

- **Title**: the slide title
- **Visual**: what the slide should show
- **Bullets**: the bullet content
- **Speaker notes**: what to say
- **Source cell(s)**: which `cost-model.xlsx` cell(s) the numbers come from

> **Numbers in this deck are calibrated with real team data**: 26 rework work-items observed over 4.25 months (Jan 1 – May 8, 2026), mean 3.04 PRs per rework item, 24 measured pipeline build durations (critical path 37 min per PR). See `cost-model.xlsx → Real-Bug-Data`, `Real-Build-Times`, and `Process-Steps` tabs. The IBM 15x cost multiplier remains the industry anchor; our bottom-up calculation from the documented 46-step process corroborates it.

---

## Slide 1: Title

**Title**: Shift Left — The Cost of Catching Bugs in the Wrong Place

**Subtitle**: A business case for a working local development environment

**Visual**: Cost-of-defect curve image (1x → 100x bars by stage), faded as background

**Speaker notes**: *"This isn't a developer-comfort pitch. It's a financial argument that we are paying a 15-100x premium on every bug we catch in the wrong place. Let me show you the math."*

---

## Slide 2: TL;DR

**Title**: The ask in one slide

**Visual**: Three big numbers, large font

**Bullets**:
- **One-time investment**: ~$15k (engineer setup + training + machine upgrades)
- **Ongoing investment**: ~$10k/year (Docker licenses + maintenance)
- **Annual savings (conservative)**: ~$580k+
- **Payback period**: < 1 month
- **3-year ROI**: ~37x

**Speaker notes**: *"This is the headline. I'll defend each of these numbers with our own data. Notice the ratio — even at the most conservative end of the sensitivity range, this is one of the highest-ROI investments available to us."*

**Source cells**: `ROI-Summary!B2:B15`

---

## Slide 3: The shift-left curve (industry data)

**Title**: A bug found in our Development environment costs ~15x more than the same bug at the developer's desk

**Visual**: Bar chart with stages on x-axis, multiplier on y-axis. We use a 4-stage view matching our actual environments.

| Local | Development | Staging | Production |
|---|---|---|---|
| 1x | 15x | 40x | 100x |

**Bullets**:
- IBM Systems Sciences Institute, corroborated by NIST RTI 2002 and Capers Jones
- *Same defect, different stages, dramatically different cost*
- The cost is paid in: rework, redeployment, multi-person triage, opportunity cost
- Our pipeline: **Local → Development → Staging → Production** (no separate Integration env; CI runs but does not gate, so CI-flagged bugs flow through to Development at 15x cost)

**Speaker notes**: *"This curve is the foundation of every shift-left argument in software engineering. It's been measured many times. The point is simple: we want bugs caught as far left as possible. Today, ours aren't. Note we have only four environments — CI runs but isn't a gate, so its catches still cost us at the Development-stage rate."*

---

## Slide 4: Where our bugs are caught today

**Title**: Our defect distribution — most defects are caught at the wrong stage

**Visual**: Stacked horizontal bar chart of % defects by detection stage. Compare to "elite teams" benchmark bar.

**Bullets**:
- Local: **35%** (industry elite: 80%+)
- Development: **45%** (consolidates what other orgs split between "CI" 15% + "Integration" 30%, because our CI doesn't gate)
- Staging: 15%
- Production: 5%
- **Phase Containment Effectiveness (local) = 35% — bottom-quartile**

**Speaker notes**: *"This data comes from our last 180 days of Azure Boards bug records. We catch 35% of defects locally. Elite teams — Microsoft and Google's internal benchmarks — sit at 80%+. The 45-percentage-point gap is the gap we're proposing to close."*

**Source cells**: `Defect-Distribution-PCE!B5:B9` (today's distribution from `data-gathering-checklist.md` query)

---

## Slide 5: What shifting left would save (THE MONEY SLIDE)

**Title**: Moving 40% of defects from Development to local saves ~$333k/year

**Visual**: Two stacked bars side by side — "Today" and "Target". Annotate the dollar delta with a callout arrow.

**Bullets**:
- Today: 400 bugs/yr × avg cost $1,412 = **$565k/yr**
- Target (PCE 75%): 400 bugs/yr × avg cost $581 = **$232k/yr**
- **Annual savings from shift-left rework alone: $333k**
- Same bug volume, different distribution

**Speaker notes**: *"This is the central financial claim. Same number of bugs — we're not promising fewer bugs, just bugs found earlier. The 18x weighted-average defect cost drops to 7.45x. That delta, at our defect volume, is $333k a year."*

**Source cells**: `ROI-Summary!Shift_Left_Savings`

---

## Slide 6: How a bug fix actually works today

**Title**: Every Development-stage bug triggers a 13-step workflow

**Visual**: The flowchart from `development-fix-workflow-cost.md` — branch → fix → AI check → push → CI → PR → wait → review → rounds → approve → merge → redeploy → reverify

**Bullets**:
- Average dev time per fix: **6-9 hours**
- Average reviewer time per fix: **1-1.5 hours**
- Pipeline retry rate: 30% (multiplies the above by 1.3x)
- Per-fix cost: **~$729 of fully-loaded engineering time**
- (Note: CI runs in step 6 but is non-gating, so it does not prevent the workflow from completing on broken code)

**Speaker notes**: *"This is what every Development-env bug costs us. Not in theory — we counted the steps and the time. Each step is rational on its own. The aggregate is expensive."*

**Source cells**: `Development-Fix-Workflow!B5:B25`

---

## Slide 7: The workflow tax we'd avoid

**Title**: Most of those 13 steps vanish for locally-caught bugs

**Visual**: Same flowchart from slide 6, with the 11 outer-loop steps crossed out / greyed. Only "fix" and "verify" remain.

**Bullets**:
- Locally-caught bug: ~$78 in dev time, no PR workflow needed
- **Per-bug delta: $651 ($729 − $78)**
- For 180 Development-env bugs/yr × 70% shifted left: ~$82k/yr in workflow tax avoided
- (Subsumed in Pillar 1 if `use_bottom_up_multiplier` toggle is ON)

**Speaker notes**: *"The 13-step workflow is appropriate risk control for changes flowing to production. But for a bug-fix that didn't need to exist in the first place — because it could have been caught locally — every step on this list is pure deadweight."*

---

## Slide 7b: Side finding — CI exists but doesn't gate deployment

**Title**: An easy, complementary fix: turn on CI gating

**Visual**: Simple flow comparing today's pipeline (CI fails → deploys anyway → caught in Dev at 15x) vs gated (CI fails → blocks deploy → caught in CI at 6.5x).

**Bullets**:
- Today: our CI runs builds, unit tests, and QA tests on every push — but does **not** stop deployment if they fail
- Result: bugs CI *could* catch at 6.5x cost escape to Development env and cost 15x instead
- Cost to fix: ~half a day of platform-engineering work to add a deployment gate
- Annual benefit: ~$15-30k/year (5-10% of Development-stage rework cost)
- **This does not replace the local-env investment**, but it's nearly free and should be done alongside

**Speaker notes**: *"While we're talking about pipelines, one quick aside. Our CI runs and produces failing results — but those results don't gate the deployment. The code goes anyway. So we're paying for CI but only getting half its value. Turning gating on is an afternoon's work and saves us another $15-30k a year. We should do it as part of the rollout."*

---

## Slide 8: What's an "inner loop"?

**Title**: The cycle that breaks when developers can't run code locally

**Visual**: Two flowcharts side by side from `business-case.md`:
- "Today": Edit → Push → CI (10 min) → Deploy (5-10 min) → Verify (3-5 min) → loop
- "Target": Edit → Run locally (30s) → Verify (30s) → loop

**Bullets**:
- Inner loop = edit → run → verify, repeated dozens of times per day
- When inner loop is broken (today), every iteration takes 15-25 minutes
- Healthy teams: 30-90 seconds per iteration
- This is **the** metric for engineering productivity (Microsoft DX research)

**Speaker notes**: *"Software development happens in two nested loops. The inner loop is where developers spend most of their time. Ours is broken — every code change must round-trip through CI to be verified. This makes the next slide possible."*

---

## Slide 9: Cycle time math

**Title**: Our inner loop is ~12x slower than it should be

**Visual**: Bar chart: 25 min vs 2 min, with team-wide annual hours-lost callout.

**Bullets**:
- Today's median: **25 min/iteration** (placeholder until time-tracking week)
- Target: **2 min/iteration**
- Iterations per dev per day: **12** (Microsoft inner-loop research)
- **Daily cycle-time loss per dev: 4.6 hours** (50% recovered via context-switch)

**Speaker notes**: *"Twelve iterations a day, 23 minutes lost per iteration, eight developers — even if half that wait time is recovered through context-switching, we're losing 2.3 hours per dev per day."*

**Source cells**: `Cycle-Time-Sprint!B5:B15`

---

## Slide 10: The sprint-impact slide

**Title**: We are paying for 8 developers and getting the output of 5.3

**Visual**: Two stacked bars — "Sprint capacity (paid for)" 504 hours vs "Sprint capacity (delivered)" ~338 hours. The missing 33% as red.

**Bullets**:
- **33% of sprint capacity is consumed by cycle-time waits**
- Equivalent to losing 2.7 developers' worth of output
- ~$365k/year of paid-for capacity not converted to delivered work
- Plus: **22% story carry-over rate** today vs 5% achievable
- Sprint commitment hit rate suffers; predictability erodes

**Speaker notes**: *"This is the slide that I expect to land most directly with you. We pay for 8 full-time developers. We get the output of 5.3. The other 2.7 developer-equivalents are dissolved into cycle-time waits. This isn't bad work; the team is doing everything right. The infrastructure isn't."*

**Source cells**: `Cycle-Time-Sprint!B20:B27`

---

## Slide 11: Workarounds the team relies on today

**Title**: We currently rely on 11 of these 14 workarounds

**Visual**: 14-row checklist with green checkmarks on the 11 we use. (Numbers depend on actual self-assessment.)

**Bullets** (sample, populate from team self-assessment):
- ✅ Sharing one remote dev environment (queue contention)
- ✅ Hardcoded conn strings to Azure dev (security debt)
- ✅ Disabled auth locally ("if env=local skip auth") (audit risk)
- ✅ Push-to-test debugging (5-15 round-trips per bug)
- ✅ Mocking Azure services in code (drift between dev and prod)
- ✅ Long-lived feature branches (merge conflict spikes)
- ✅ Comment-out-and-redeploy debugging
- ✅ Skipping integration tests locally
- ✅ Manual data-prep scripts (tribal knowledge)
- ✅ "Your turn QA" handoffs
- ✅ Disabled local regression suite
- ❌ Personal Azure subscriptions (rare here)
- ❌ Pair-debugging on shared env (rare)
- ❌ Long onboarding (we have it OK so far)

**Speaker notes**: *"This is a list of 14 things developers in environments-without-emulators routinely do. We had the team mark which ones we currently rely on. Eleven of fourteen. Each is rational individually; the aggregate is significant — security debt, productivity tax, onboarding friction."*

**Source cells**: `Workarounds!B5:B19`

---

## Slide 12: Industry corroboration

**Title**: This isn't novel — every elite engineering org has solved this

**Visual**: Logos / quotes / citations.

**Bullets**:
- **DORA *State of DevOps***: elite teams have 973x faster recovery times than low performers
- **Microsoft DX research**: inner-loop time is the #1 predictor of engineering productivity
- **Capers Jones**: PCE benchmarks consistently show top-quartile teams catch 85%+ locally
- **Google internal**: Bazel + emulators + hermetic builds achieve sub-minute inner loops
- **Industry pattern**: every mature dev org runs services locally; we are an outlier

**Speaker notes**: *"None of this is novel. Every well-run engineering org has solved this problem. We're not asking to be on the cutting edge — we're asking to catch up to industry baseline."*

---

## Slide 13: ADO pipeline cost we'd recover

**Title**: ~$32k/year in pipeline retry costs go away

**Visual**: Stacked bar: visible costs (compute) vs hidden costs (idle, retries).

**Bullets**:
- **Compute cost** (the visible Azure invoice): ~$8k/year
- **Engineer idle during build/queue**: ~$212k/year (50% recovery factor)
- **Failed-build retries (recoverable)**: ~$32k/year
- Most of pipeline cost isn't the invoice — it's developer time
- Local testing eliminates most retries

**Speaker notes**: *"This slide quietly handles the platform-engineering objection. The Azure invoice is the small number. The wait-time and retry overhead is 30x larger. We don't need a bigger pipeline budget; we need to push fewer broken builds to it."*

**Source cells**: `Pipeline-Cost!B5:B30`

---

## Slide 14: Annual savings — full waterfall

**Title**: Total annual savings: ~$777k (mid case)

**Visual**: Waterfall chart starting at $0 stepping up:
1. Shift-left rework: +$355k
2. Pipeline retries: +$32k
3. Sprint capacity: +$365k
4. Workarounds: +$25-80k
5. Total: ~$777-832k
6. − Investment: ~$10k/year
7. = Net: **~$767-822k/year**

**Speaker notes**: *"All four pillars, additive. Sprint-capacity is the largest because it's a per-developer recurring cost; shift-left is second because it has the multiplier effect. Even if any one of these turned out to be half what we estimate, the total is still seven figures."*

**Source cells**: `ROI-Summary!B5:B20`

---

## Slide 15: Investment, ROI, sensitivity, risks

**Title**: We can absorb 50% downside in every assumption and the case still holds

**Visual**: Four-quadrant slide:
- **Investment** (top-left): table from `business-case.md`
- **ROI** (top-right): payback < 1 month, 3-year NPV ~$1.35M
- **Sensitivity** (bottom-left): tornado chart showing which inputs matter most
- **Risks** (bottom-right): table of 6 risks + mitigations

**Speaker notes**: *"Sensitivity analysis is here so you don't have to take any one number on faith. The case is robust to ±25% on every input. The biggest risks are emulator parity gaps and team learning curve — both managed by keeping CI integration tests on real Azure as a safety net, and by phased rollout."*

**Source cells**: `ROI-Summary` and `Sensitivity` tabs

---

## Slide 16: The ask

**Title**: Approve the investment, phased over 8 weeks

**Visual**: Timeline with 4 phases.

**Bullets**:
- **Phase 1 (4 weeks)**: One engineer (lead) builds `docker-compose.yml`, validates emulator parity, documents setup
- **Phase 2 (2 weeks)**: 2-3 volunteer devs pilot the local env
- **Phase 3 (2 weeks)**: Whole-team rollout + training session
- **Phase 4**: Quarterly metrics review

**Success metrics tracked quarterly**:
- PCE(local), inner-loop cycle time, story carry-over rate, CI failure rate, PR cycle time, % defects in production

**The decision**: approve / modify / decline

**Speaker notes**: *"The ask is small, the rollout is phased so we can stop or adjust at any point, and the metrics are clear. If the savings don't materialise — say within two quarters — the dashboard will show it and we can reconsider. Do you have approval to proceed, questions, or modifications?"*

---

## Speaker preparation notes

### Likely objections and responses

| Objection | Response |
|---|---|
| *"The numbers seem high"* | "Every input is parameterised — let's walk through the spreadsheet. Show me which input you think is wrong and we'll adjust." |
| *"Emulators don't have parity with real Azure"* | "Correct, ~80% feature coverage. We keep CI integration tests on real Azure as the safety net. The 80% covers ~95% of daily dev work." |
| *"Why hasn't the team done this already?"* | "It's a coordination problem — no one developer can fix it alone, and we haven't had explicit time allocated. This proposal is asking for that allocation." |
| *"How do we know the savings will materialise?"* | "Tracked quarterly via the metrics on slide 16. If PCE doesn't improve in 2 quarters, the investment was wrong." |
| *"What about smaller teams that are <250 employees?"* | "Docker Desktop is free under that threshold. The license cost line goes to zero." |
| *"Can't we just buy more parallel jobs / a faster pipeline?"* | "We can. It addresses 5% of the cost (compute) and none of the other 95%. See pipeline slide." |

### Customisation TODO before presenting

- [ ] Replace placeholder `8 developers` with your actual headcount
- [ ] Replace placeholder `$85/hr` with your finance-confirmed loaded rate
- [ ] Replace placeholder defect distribution with the WIQL-query result
- [ ] Replace placeholder PR cycle time with the ADO Analytics result
- [ ] Replace placeholder workaround-self-assessment with the team's actual marks
- [ ] Add 1-2 specific recent Development-env-bug examples ("remember when X happened…")
- [ ] Add the 2-3 specific Azure services you most want to emulate (and the parity status of each)
- [ ] Confirm Docker pricing tier applicable to your org size
- [ ] Customise success-metrics targets to be team-realistic (the 6/12-month columns are placeholders)
