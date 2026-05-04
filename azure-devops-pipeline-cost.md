# Azure DevOps Pipeline Cost — Per-Build Breakdown

## TL;DR

Each Azure DevOps build/regression run has **four cost components**, of which only the first is on Microsoft's invoice. The other three — engineer idle time, retry overhead, and queue contention — are typically 5-10x larger than the compute cost itself but invisible in the budget.

> A 30-minute pipeline that runs 6× per developer per day on an 8-developer team is costing ~$50-150k/year all-in, of which only ~$8-15k shows up on the Azure invoice.

---

## The four components

### 1. Compute cost (the visible one)

#### Microsoft-hosted agents (current Azure DevOps Services pricing)

| Tier | Cost |
|---|---|
| First parallel job (private projects) | **Free**, 1,800 minutes/month included |
| Each additional parallel job (Microsoft-hosted) | **$40 / parallel job / month** |
| Each additional parallel job (self-hosted) | **$15 / parallel job / month** + your VM cost |
| Public projects | 10 free parallel jobs, no per-minute cap |

> Source: [Microsoft Azure DevOps pricing](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/) (verify current pricing — it has been stable for years but should be confirmed at presentation time).

#### Deriving the marginal per-minute cost

Microsoft prices in parallel jobs (concurrency slots), not minutes. To convert to a per-minute figure for the cost model:

```
marginal_cost_per_minute = $40 / minutes_used_that_month_on_that_job

Example: a parallel job running 100 hours/month (6,000 min):
  $40 / 6,000 = $0.0067/min ≈ $0.40/hour

Example: a parallel job running only 30 hours/month (1,800 min, just enough to fully utilise it):
  $40 / 1,800 = $0.022/min ≈ $1.33/hour
```

**Implication**: pipelines that run *more often* are *cheaper per minute* on Microsoft-hosted agents. Underutilised parallel jobs are very expensive per minute. **The cost model uses $0.015/min as a balanced default ($40 ÷ ~2,650 min/mo).**

#### Self-hosted agents

If you run agents on your own VMs:

```
self_hosted_cost_per_minute = ($15 / parallel_job / month + monthly_vm_cost) / minutes_used
```

A typical Azure D4s_v5 VM costs ~$140/month (24/7) or ~$50/month (work hours only with autoscale). For a moderately used job:

```
($15 + $50) / 2,650 min = $0.025/min
```

Self-hosted is cheaper per parallel slot, but the VM has to be paid for whether or not it's used. Microsoft-hosted is better for bursty workloads; self-hosted is better for steady high-utilisation workloads. **The model defaults to Microsoft-hosted.**

---

### 2. Regression test execution cost

For a team running CI on every push:

```
annual_regression_minutes =
    devs × pushes_per_dev_per_day × suite_minutes × working_days_per_year

Example (8 devs × 6 pushes/day × 30 min × 250 days):
    = 360,000 minutes/year

annual_regression_compute_cost =
    360,000 × $0.015/min ≈ $5,400/year
```

This is the small number. The big number is below.

---

### 3. Engineer idle time during build/queue (the largest hidden cost)

When a developer pushes and waits for CI before they can validate their change, the wait time has three components:

| Component | Description |
|---|---|
| **Queue wait** | Time before the build starts, gated by parallel job availability. Grows non-linearly with utilisation (M/M/1 queueing). |
| **Build duration** | Compile + unit tests. |
| **Test/regression duration** | Integration tests, often the largest chunk. |

The total is usually 15-45 minutes for our pipeline. During this time the developer is *partially* productive on other work but pays a context-switch tax.

```
engineer_idle_cost_per_build =
    (queue_wait + build_duration + test_duration) × hourly_rate × idle_recovery_factor

  where idle_recovery_factor = 1 - context_switch_recovery
        (e.g. 50% if half the wait time is genuinely productive)

Example: 25 min wait × $85/hr × 0.5 idle factor = $17.71/build

Annual cost (8 devs × 6 builds/day × 250 days):
  12,000 builds × $17.71 = $212,500/year
```

**This is typically 30-50x the compute cost.** It's invisible to Azure billing but very visible to the team's velocity.

---

### 4. Failed-build retry cost (the shift-left lever)

If our CI failure rate is 30% (industry average is 20-40%), and 50% of those failures are caused by issues that *should have been caught locally*:

```
retries_caused_by_missing_local_env =
    total_builds × failure_rate × pct_caused_by_missing_local_env

Example: 12,000 × 0.30 × 0.50 = 1,800 retry builds/year

cost_per_retry = compute_cost + engineer_idle_cost
              = ($0.015 × 25 min) + ($17.71)
              = $18.09

annual_retry_cost = 1,800 × $18.09 = $32,562/year
```

**This entire line is recoverable savings.** A working local environment lets developers run the same checks before pushing, eliminating the retry round-trip.

---

## Putting it all together — annual ADO pipeline cost

For an 8-developer team with a 30-min pipeline running 6×/dev/day, 250 working days, 30% retry rate, $85/hr loaded rate, $0.015/min compute:

| Component | Annual cost |
|---|---|
| Compute (regression + builds) | $5,400 |
| Parallel-job subscription (e.g. 5 extra jobs) | $2,400 |
| Engineer idle during build/queue | $212,500 |
| Failed-build retry overhead | $32,562 |
| **Total all-in pipeline cost** | **~$252,862/year** |

Of which roughly **$32-50k/year is recoverable** by enabling proper local testing (eliminate retries; reduce push-to-test as primary debugging mechanism).

The compute line is small but the *behavioural* line items (idle, retries) are the targets.

---

## How shared environments amplify the cost (the contention curve)

When N developers share one integration environment, they share its capacity. M/M/1 queueing theory gives wait time as a function of utilisation ρ:

```
expected_wait_time ≈ (ρ / (1 - ρ)) × service_time
```

| Utilisation ρ | Wait multiplier |
|---|---|
| 50% | 1.0× service time |
| 70% | 2.3× |
| 80% | 4.0× |
| 90% | 9.0× |
| 95% | 19.0× |

A 25-minute pipeline at 80% utilisation has an effective wait of **125 minutes** including queueing. This is why "the build queue exploded today" tickets cluster — it's not bad luck, it's the math.

**Local environments break the contention curve entirely** because each developer has their own environment with ρ = (their own utilisation, typically <30%).

---

## What to ask the platform/DevOps team

1. **How many parallel jobs are we paying for?** ($40/each Microsoft-hosted, $15/each self-hosted.)
2. **What's our pipeline utilisation?** Pull from Azure DevOps Analytics.
3. **What's the median and p95 queue wait time?** Plot on the contention curve to identify saturation.
4. **What's the build/test failure rate?** And of failures, how many are environment-related vs code-related?
5. **What's the average pipeline duration end-to-end?** Including any deploy-to-test-env stages.
6. **Are we using Microsoft-hosted, self-hosted, or both?**

Queries are in [`data-gathering-checklist.md`](data-gathering-checklist.md).

---

## Sources

- Microsoft — *Azure DevOps Services pricing*, [https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/) (verify at presentation time)
- M/M/1 queueing theory — standard operations-research result. Cited in *The Phoenix Project* (Kim, Behr, Spafford) for the same purpose.
- DORA *State of DevOps* — change-failure rate benchmarks (elite ≤15%, low 46-60%).
