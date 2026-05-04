# Cycle Time and Sprint Impact

## TL;DR

Without a working local environment, every code change requires a 15-45 minute round-trip to the integration env to verify. Healthy teams verify changes in under 2 minutes locally. Multiplied across iterations, developers, and sprints, this cycle-time gap **directly subtracts from sprint capacity** — typically 20-30% of nominal velocity, equivalent to losing 1.5-2 developers' worth of output on an 8-person team.

> **For an 8-dev team: ~$340-510k/year in sprint capacity lost to cycle-time delays alone.**
> **Plus: ~20% story carry-over rate today vs ~5% achievable, eroding sprint predictability.**

---

## The inner loop is the loop that matters

Software development happens in two nested loops:

- **Inner loop**: edit → run → verify (seconds to minutes). Repeated dozens of times per day.
- **Outer loop**: commit → push → CI → review → deploy → verify (minutes to hours). Repeated a few times per day.

The inner loop is where most engineering work actually happens. When the inner loop is broken (because the developer can't run the code locally and must use the outer loop to verify *every* change), productivity collapses non-linearly.

```
inner_loop_work = N iterations × T_iteration
where N is large (typically 50-100 per dev per day)
```

If T_iteration goes from 1.5 min to 20 min (a 13x degradation), the dev's productive day shrinks from ~2.5 hours of inner-loop work to *the entire day spent waiting*. This isn't theoretical — Microsoft's internal research on inner-loop developer productivity treats inner-loop time as the single most important metric for engineering effectiveness.

---

## Cycle-time math

### Today (broken inner loop)

```
T_today = edit + commit + push + CI_build + deploy_to_shared_dev + repro + verify
        ≈ 2 + 1 + 1 + 8 + 6 + 3 + 4 = 25 min/iteration  (placeholder)
```

Plus queue time when shared environments are contended:

```
T_today_with_contention = T_today + queue_wait
                        ≈ 25 + (variable, often 0-30 min)
```

### Target (working inner loop)

```
T_target = edit + run_locally_against_emulator + verify
         ≈ 1 + 0.5 + 0.5 = 2 min/iteration
```

### The delta

```
cycle_time_delta = 25 - 2 = 23 min/iteration
```

---

## Sprint capacity erosion

### Inner-loop iterations per developer per day

Microsoft's inner-loop research and DX (Developer Experience) surveys converge on **8-15 iterations per dev per day** for non-trivial work. We use **12** as the default.

```
daily_cycle_time_lost_per_dev = 12 iterations × 23 min/iteration delta
                              = 276 min/day
                              = 4.6 hours/day
```

### But devs aren't *just* idle — they context-switch

Half-realistic devs fill cycle-time waits with other work (Slack, email, code review for someone else, the parallel feature). DeMarco/Weinberg's *Peopleware* research shows context-switching costs ~20% of nominal productivity. We model this as:

```
effective_cycle_time_loss = daily_cycle_time_lost × idle_recovery_factor

  where idle_recovery_factor = 0.5
        (half the wait time is lost productivity even with context switching)

effective_loss_per_dev_per_day = 4.6 × 0.5 = 2.3 hours/day
```

### Sprint impact

For 2-week sprints with 9 working days per dev (10 weekdays minus 1 day of ceremonies):

```
sprint_hours_lost_per_dev = 2.3 × 9 = 20.7 hours/sprint

For 8 devs:
  total_sprint_hours_lost = 165.6 hours/sprint
                          = 4.1 dev-weeks/sprint

  Sprint nominal capacity:
    8 devs × 9 days × 7 prod hours = 504 hours/sprint

  Capacity erosion:
    165.6 / 504 = 33%
```

> **One-third of sprint capacity is currently consumed by cycle-time waits.**
>
> Or equivalently: we are paying for 8 developers and getting the output of 5.3.

### Annualised dollarisation

```
annual_sprint_capacity_lost_usd =
    devs × hours_lost_per_dev_per_sprint × sprints_per_year × loaded_hourly_rate

Example (8 devs × 20.7 hrs × 26 sprints × $85):
  = $365,976/year
```

---

## Story carry-over and predictability

The hours-lost number above is the *direct* cost. There's a second-order effect that often hurts more: **story carry-over**.

Stories that don't finish in the sprint either:
1. Get carried to the next sprint (dropping next sprint's capacity for new work), or
2. Get rushed to "done" with quality compromises (creating future bug volume — feeds the shift-left problem).

Industry benchmarks for healthy teams: **5-10% story carry-over rate**.
Teams with broken inner loops typically run at **20-30%**.

Carry-over rate matters to management because it directly drives **sprint predictability** — the ability to credibly commit to "this will ship in this sprint". Predictability is the metric Product Managers, Account Managers, and execs actually feel, even when they don't quote it numerically.

```
predictability_premium =
    revenue_at_risk_per_missed_sprint × probability_of_miss

The "revenue at risk" is qualitative — it includes:
- Customer commitment slips
- Delayed feature launches that would unlock revenue
- Sales conversations that depend on roadmap credibility
- Internal stakeholder confidence in engineering
```

We do **not** put a hard dollar number on predictability premium in the spreadsheet because it varies wildly by business model. But every business has *some* premium for delivering on schedule — and a chronic 20% sprint-miss rate erodes it constantly.

---

## The contention amplifier (shared environments)

When multiple devs share one integration/dev environment, cycle time gets *worse* than the per-dev calculation suggests. M/M/1 queueing theory:

```
expected_wait_time = (ρ / (1 - ρ)) × service_time
where ρ = utilisation (0 to 1)
```

| ρ | Wait multiplier | Effective cycle time (25 min base) |
|---|---|---|
| 50% | 1.0× | 50 min |
| 70% | 2.3× | 82 min |
| 80% | 4.0× | 125 min |
| 90% | 9.0× | 250 min (>4 hours) |

This is why "the dev environment is broken / overloaded" days happen in clusters. Above 70-80% utilisation, the wait time **explodes**. Local environments eliminate this entirely because each dev has their own (ρ ≈ their own, typically <30%).

---

## What "good" looks like

Track these metrics quarterly to demonstrate the recovery:

| Metric | Today (placeholder) | 6-month target | 12-month target |
|---|---|---|---|
| Median inner-loop time per iteration | 25 min | 8 min | 2 min |
| Iterations per dev per day | 12 | 25 | 40 |
| Sprint capacity erosion from cycle time | 33% | 18% | 8% |
| Story carry-over rate | 22% | 12% | 6% |
| Sprint commitment hit rate | 78% | 88% | 94% |
| Builds queued > 5 min | weekly | monthly | rare |

**Note**: iterations/day rises dramatically in the target state. Faster loops don't just save time on existing iterations; they unlock entirely new ways of working (TDD, fearless refactoring, exploratory debugging) that the 25-min-loop world simply doesn't allow.

---

## Why this slide lands with managers

Managers care about:

1. **Sprint commitments** — predictability of delivery.
2. **Headcount efficiency** — output per developer.
3. **Stakeholder relationships** — credibility with PMs and execs.

This slide speaks all three:

- "We are paying for 8 developers and getting 5.3" → headcount efficiency.
- "1 in 5 stories carries over to next sprint, every sprint" → predictability.
- "Each cycle of feature delivery is 33% slower than it should be" → stakeholder relationships.

Pair this with two recent real examples — *"remember when story X was supposed to ship sprint 24 but slipped to 26 because of integration-env issues?"* — and the abstract becomes concrete.

---

## Sources

- Microsoft — Inner-loop developer productivity research (partially published; the *DX Core 4* whitepaper is the most accessible public source).
- Tom DeMarco & Timothy Lister — *Peopleware: Productive Projects and Teams* (3rd ed., 2013). Context-switching tax.
- Gerald Weinberg — *Quality Software Management Vol. 1*. Estimates ~10% productivity loss per concurrent task.
- Gene Kim, Kevin Behr, George Spafford — *The Phoenix Project*. Queueing theory applied to software delivery.
- DORA / Google Cloud — *State of DevOps Report*. Lead time for changes; deployment frequency.
