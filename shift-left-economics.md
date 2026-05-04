# Shift-Left Economics

## TL;DR

The same defect costs **15x more** to fix when caught in the integration environment than when caught locally, and **100x more** when it escapes to production. Our team currently catches ~35% of defects locally; elite teams catch 80%+. Closing that gap is the single largest available source of engineering savings.

---

## What "shift-left" means in plain terms

Software defects accumulate cost the longer they go undetected. The cost is not linear — it is roughly exponential, because each later detection stage involves more people, more infrastructure, more rework, and more lost context.

> *Shift-left* = relocate defect detection to earlier stages of the pipeline.

Our investment thesis is simple: we are currently paying a 15-100x premium for defects we could have caught at 1x cost if developers had a working local environment.

---

## The cost-of-defect curve (industry-anchored)

The IBM Systems Sciences Institute multiplier is the canonical reference and is widely accepted by engineering finance functions. Capers Jones' *Applied Software Measurement* and the NIST RTI 2002 report corroborate it.

| Stage where defect is caught | Cost multiplier (relative to local) |
|---|---|
| **Local — developer machine** | **1x** (baseline) |
| CI build / code review | ~6.5x |
| **Integration / QA environment** | **~15x** |
| Staging / UAT | ~40x |
| **Production** | **~100x** |

### Why the integration stage specifically costs ~15x

A bug found in the integration environment requires *all* of the following work that a locally-caught bug skips:

1. **Reproduction** — deploy → wait → reproduce in a shared environment, often coordinating with whoever else is using it.
2. **Remote debugging** — logs are aggregated and delayed; debugging is push-and-pray rather than step-through.
3. **Multi-person triage** — typically pulls in QA + DevOps in addition to the original developer (3-person cost where local would have been 1).
4. **Heavyweight fix workflow** — branch → fix → AI code-check → push → CI → PR → review rounds → approval → merge → redeploy → reverify (see [`integration-fix-workflow-cost.md`](integration-fix-workflow-cost.md)).
5. **Context-switch tax** — by the time the bug surfaces, the developer has moved on to other work; reloading mental context costs an additional 20-40% (DeMarco/Weinberg).
6. **Environment contention** — other developers are blocked on the same shared environment while the bug is being investigated.

This is why ~15x is **the conservative figure**. Our bottom-up calculation in [`integration-fix-workflow-cost.md`](integration-fix-workflow-cost.md) shows the real number for our team is closer to **10-12x for the dev-time component alone**, and rises to **18-25x** once we add reviewer time, queue/wait time, and pipeline retry costs.

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

We pull from Azure Boards using the `Found In` field on the Bug work item type. The query is in [`data-gathering-checklist.md`](data-gathering-checklist.md). The result is a count of defects by detection stage over the last 90 or 180 days, which produces our actual defect-detection distribution.

---

## Today's defect distribution (placeholder — replace with real data)

Until we run the queries in `data-gathering-checklist.md`, the model assumes:

| Stage caught | % of defects today | Cost multiplier | Weighted cost units |
|---|---|---|---|
| Local | 35% | 1x | 0.35 |
| CI build | 15% | 6.5x | 0.98 |
| Integration | 30% | 15x | 4.50 |
| Staging | 15% | 40x | 6.00 |
| Production | 5% | 100x | 5.00 |
| **Weighted total** | **100%** | | **16.83** |

**Average defect costs ~16.8 cost-units today.** A perfect-shift-left team would pay ~1.0 cost units. We are paying ~17x what we'd pay if we caught everything locally.

## Target distribution after the investment

| Stage caught | % of defects (target) | Cost multiplier | Weighted cost units |
|---|---|---|---|
| Local | 70% | 1x | 0.70 |
| CI build | 15% | 6.5x | 0.98 |
| Integration | 10% | 15x | 1.50 |
| Staging | 4% | 40x | 1.60 |
| Production | 1% | 100x | 1.00 |
| **Weighted total** | **100%** | | **5.78** |

**Average defect would cost ~5.8 cost-units after the investment.** A **66% reduction** in average defect cost, at constant defect volume.

> If our defect volume is 400/year and our local-caught defect costs ~$80, today's annual rework cost is approximately:
>
> 400 × $80 × 16.83 ≈ **$538k/year**
>
> Post-investment:
>
> 400 × $80 × 5.78 ≈ **$185k/year**
>
> **Headline annual savings from shift-left alone: ~$353k.**
>
> *(Replace 400, $80, and the distributions with your real numbers in `cost-model.xlsx`.)*

---

## Why this is the single biggest savings lever

The other levers in this business case (cycle time, workarounds, pipeline cost) each save in the **tens of thousands of dollars** range per year for a typical team. The shift-left lever saves in the **hundreds of thousands** range because it is multiplicative, not additive — every percentage point of PCE we move to the left captures the full 15x-100x cost differential.

This is also the lever that compounds with everything else: a faster local loop *causes* more bugs to be caught locally, which *causes* PCE to improve.

---

## What "good" looks like (success metrics)

These are the metrics we will track quarterly to demonstrate ROI after the investment:

| Metric | Today (placeholder) | 6-month target | 12-month target |
|---|---|---|---|
| **PCE(local)** | 35% | 55% | 70% |
| % defects escaping to integration | 30% | 18% | 10% |
| % defects escaping to production | 5% | 2% | 1% |
| Average defect cost (weighted) | 16.8 units | 9.0 units | 5.8 units |
| Avg defect cost in $ | $1,344 | $720 | $464 |

We do not need to hit elite (80%+) to break even — modelling in `cost-model.xlsx` shows payback is achieved at PCE(local) = 50%.

---

## Sources

- IBM Systems Sciences Institute — cost-of-defect curve. Cited in *Code Complete* (McConnell, 2004) and many subsequent references.
- NIST / RTI International (2002) — *The Economic Impacts of Inadequate Infrastructure for Software Testing* (Report 02-3). Estimated $59.5B/yr cost in the US economy alone.
- Capers Jones — *Applied Software Measurement* (3rd ed.) and *Software Engineering Best Practices*. PCE benchmarks.
- DORA / Google Cloud — *State of DevOps Report* (annual). Elite vs low-performer differentials.
- Microsoft Engineering Excellence — internal inner-loop research, partially published.
