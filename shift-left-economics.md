# Shift-Left Economics

## TL;DR

The same defect costs **15x more** to fix when caught in our Development environment than when caught locally, and **100x more** when it escapes to production. Our team currently catches ~35% of defects locally; elite teams catch 80%+. Closing that gap is the single largest available source of engineering savings.

---

## What "shift-left" means in plain terms

Software defects accumulate cost the longer they go undetected. The cost is not linear — it is roughly exponential, because each later detection stage involves more people, more infrastructure, more rework, and more lost context.

> *Shift-left* = relocate defect detection to earlier stages of the pipeline.

Our investment thesis is simple: we are currently paying a 15-100x premium for defects we could have caught at 1x cost if developers had a working local environment.

---

## Our pipeline topology

We have four detection stages:

```
Local  ->  Development  ->  Staging  ->  Production
```

**Note on CI**: our CI pipeline runs builds, unit tests, and QA tests on every push, but **it does not gate deployment** — failing CI does not stop code from reaching the Development environment. So CI is a *signal*, not a *containment stage*. CI-flagged bugs flow through to Development where they are operationally caught and fixed, at Development-stage cost. (Turning CI into a true gating stage is a separate, complementary improvement discussed at the bottom of this document.)

---

## The cost-of-defect curve (industry-anchored)

The IBM Systems Sciences Institute multiplier is the canonical reference and is widely accepted by engineering finance functions. Capers Jones' *Applied Software Measurement* and the NIST RTI 2002 report corroborate it.

Adapted to our four-stage topology:

| Stage where defect is caught | Cost multiplier (relative to local) |
|---|---|
| **Local — developer machine** | **1x** (baseline) |
| **Development environment** | **~15x** (IBM "Integration" stage — same workflow, just our naming) |
| Staging / UAT | ~40x |
| **Production** | **~100x** |

### Why the Development stage specifically costs ~15x

A bug found in the Development environment requires *all* of the following work that a locally-caught bug skips:

1. **Reproduction** — deploy → wait → reproduce in a shared environment, often coordinating with whoever else is using it.
2. **Remote debugging** — logs are aggregated and delayed; debugging is push-and-pray rather than step-through.
3. **Multi-person triage** — typically pulls in QA + DevOps in addition to the original developer (3-person cost where local would have been 1).
4. **Heavyweight fix workflow** — branch → fix → AI code-check → push → CI → PR → review rounds → approval → merge → redeploy → reverify (see [`development-fix-workflow-cost.md`](development-fix-workflow-cost.md)).
5. **Context-switch tax** — by the time the bug surfaces, the developer has moved on to other work; reloading mental context costs an additional 20-40% (DeMarco/Weinberg).
6. **Environment contention** — other developers are blocked on the same shared environment while the bug is being investigated.

This is why ~15x is **the conservative figure**. Our bottom-up calculation in [`development-fix-workflow-cost.md`](development-fix-workflow-cost.md) shows the real number for our team is closer to **10-12x for the dev-time component alone**, and rises to **18-25x** once we add reviewer time, queue/wait time, and pipeline retry costs.

---

## Phase Containment Effectiveness (PCE) — the metric that operationalises shift-left

PCE is the standard SEI / Capers Jones metric for "how good are we at catching bugs at the stage they're introduced?"

### Formula

```
PCE(phase) = bugs_caught_in_phase / bugs_originating_in_or_before_phase
```

For our purposes the most important PCE is **PCE(local)**:

```
PCE(local) = bugs_caught_locally / total_bugs_found
```

### Industry benchmarks

| Performer tier | PCE(local) range | Source |
|---|---|---|
| Low-performing teams | 25-40% | Capers Jones |
| Median teams | 50-65% | Capers Jones |
| Elite teams (Microsoft, Google internal) | 80-90% | Microsoft engineering effectiveness research |

### How we measure ours

We pull from Azure Boards using the `Found In` field on the Bug work item type, with values **Local / Development / Staging / Production**. The query is in [`data-gathering-checklist.md`](data-gathering-checklist.md). The result is a count of defects by detection stage over the last 90 or 180 days, which produces our actual defect-detection distribution.

---

## Today's defect distribution (placeholder — replace with real data)

Until we run the queries in `data-gathering-checklist.md`, the model assumes:

| Stage caught | % of defects today | Cost multiplier | Weighted cost units |
|---|---|---|---|
| Local | 35% | 1x | 0.35 |
| Development | 45% | 15x | 6.75 |
| Staging | 15% | 40x | 6.00 |
| Production | 5% | 100x | 5.00 |
| **Weighted total** | **100%** | | **18.10** |

**Average defect costs ~18.1 cost-units today.** A perfect-shift-left team would pay ~1.0 cost units. We are paying ~18x what we'd pay if we caught everything locally.

Note: the 45% Development figure consolidates what would otherwise be split between "CI-caught" (~15%) and "Development-env-caught" (~30%) in a topology with CI gating — because in our topology CI does not gate, all of those bugs operationally cost the Development-stage rate.

## Target distribution after the investment

| Stage caught | % of defects (target) | Cost multiplier | Weighted cost units |
|---|---|---|---|
| Local | 75% | 1x | 0.75 |
| Development | 18% | 15x | 2.70 |
| Staging | 5% | 40x | 2.00 |
| Production | 2% | 100x | 2.00 |
| **Weighted total** | **100%** | | **7.45** |

**Average defect would cost ~7.45 cost-units after the investment.** A **59% reduction** in average defect cost, at constant defect volume.

> If our defect volume is 400/year and our local-caught defect costs ~$78, today's annual rework cost is approximately:
>
> 400 × $78 × 18.10 ≈ **$565k/year**
>
> Post-investment:
>
> 400 × $78 × 7.45 ≈ **$232k/year**
>
> **Headline annual savings from shift-left alone: ~$333k.**
>
> *(Replace 400, $78, and the distributions with your real numbers in `cost-model.xlsx`.)*

---

## Why this is the single biggest savings lever

The other levers in this business case (cycle time, workarounds, pipeline cost) each save in the **tens of thousands of dollars** range per year for a typical team. The shift-left lever saves in the **hundreds of thousands** range because it is multiplicative, not additive — every percentage point of PCE we move to the left captures the full 15x-100x cost differential.

This is also the lever that compounds with everything else: a faster local loop *causes* more bugs to be caught locally, which *causes* PCE to improve.

---

## What "good" looks like (success metrics)

These are the metrics we will track quarterly to demonstrate ROI after the investment:

| Metric | Today (placeholder) | 6-month target | 12-month target |
|---|---|---|---|
| **PCE(local)** | 35% | 60% | 75% |
| % defects escaping to Development | 45% | 25% | 18% |
| % defects escaping to Staging | 15% | 8% | 5% |
| % defects escaping to Production | 5% | 3% | 2% |
| Average defect cost (weighted) | 18.1 units | 11.0 units | 7.45 units |
| Avg defect cost in $ | $1,412 | $858 | $581 |

We do not need to hit elite (80%+) to break even — modelling in `cost-model.xlsx` shows payback is achieved at PCE(local) = 50%.

---

## A related, separate improvement: turn on CI gating

Since CI runs but doesn't gate deployment, there's a complementary improvement available *independently* of the local-env investment:

- **What**: configure the CI pipeline to block deployment to Development when CI fails.
- **Cost**: an afternoon of platform-engineering work, essentially zero ongoing.
- **Benefit**: bugs that CI catches would be caught at ~6.5x cost (the IBM CI multiplier) instead of escaping to ~15x cost at the Development stage.
- **Estimated savings**: 5-10% of current Development-stage rework cost, i.e. ~$15-30k/year at our placeholder defect volume.

This is too small to anchor a business case on its own but is worth mentioning on the deck as a "we should do this too, but it doesn't replace the local-env investment."

---

## Sources

- IBM Systems Sciences Institute — cost-of-defect curve. Cited in *Code Complete* (McConnell, 2004) and many subsequent references.
- NIST / RTI International (2002) — *The Economic Impacts of Inadequate Infrastructure for Software Testing* (Report 02-3). Estimated $59.5B/yr cost in the US economy alone.
- Capers Jones — *Applied Software Measurement* (3rd ed.) and *Software Engineering Best Practices*. PCE benchmarks.
- DORA / Google Cloud — *State of DevOps Report* (annual). Elite vs low-performer differentials.
- Microsoft Engineering Excellence — internal inner-loop research, partially published.
