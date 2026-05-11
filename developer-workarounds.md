# Developer Workarounds — Catalogue and Self-Assessment

## TL;DR

Without a working local environment, developers invent workarounds to keep shipping. These workarounds are individually rational ("I'll just hardcode this for now") but collectively expensive and risky — they create security debt, drift between dev and prod, brittle tribal knowledge, and tax every developer-hour they touch.

> **Self-assessment headline format**: *"We currently rely on **N of 14** workarounds documented below. Eliminating them recovers ~$X/year in direct dev time and removes Y categories of production-incident risk."*

---

## How to use this document

1. As a team, work through the 14 workarounds below.
2. For each, mark **Yes / Partial / No** based on whether your team currently relies on it.
3. Tally the count and the estimated hours/week.
4. Use the headline ("we currently rely on 11 of 14 workarounds") on the deck.

The self-assessment grid at the bottom of this document is the artefact you bring to the management meeting.

---

## The catalogue

### 1. Sharing one remote dev/test environment across the team

| | |
|---|---|
| **Description** | Multiple devs deploy to and use a single shared Development environment to run their in-progress work. |
| **Why it exists** | They can't run their service against real Azure dependencies locally. |
| **Hidden cost** | Queue contention (see queueing math in `cycle-time-and-sprint-impact.md`); destructive-test collisions; "who's deployed right now?" Slack messages; debugging interference from someone else's data. |
| **Risk** | Two devs' broken code masking each other; data corruption; no isolation. |
| **Removed by Docker + emulators?** | **Yes — fully.** |

### 2. Hardcoded connection strings to a shared Azure dev resource

| | |
|---|---|
| **Description** | Code or local config files contain connection strings pointing at a real Azure dev/test storage account, Cosmos DB, Service Bus, etc. |
| **Why it exists** | No local equivalent exists. |
| **Hidden cost** | Azure consumption bills (often surprisingly large); credential rotation breaks every dev's setup; data hygiene problems. |
| **Risk** | **Significant.** Credential leakage; shared mutable state; one dev's tests corrupting another dev's data; auditor findings. |
| **Removed by Docker + emulators?** | **Yes** — Azurite (storage), Cosmos DB Emulator, Service Bus Emulator handle the common cases. |

### 3. Disabling auth/security locally to make things work

| | |
|---|---|
| **Description** | Code paths like `if (env == "local") skipAuth();` or hardcoded test tokens that aren't supposed to ever ship. |
| **Why it exists** | Real auth needs real Azure AD; emulators that work would let it run. |
| **Hidden cost** | Pollutes production code with environment branching; "we don't test the real auth path locally" means auth bugs only surface in the Development environment. |
| **Risk** | **Critical.** Audit findings, accidental ship of dev code, security debt. *Has caused real-world incidents at multiple Fortune 500 companies.* |
| **Removed by Docker + emulators?** | **Yes** — Azurite + Azure AD emulation patterns + dev secret stores remove the need. |

### 4. Push-to-test debugging ("printf via CI")

| | |
|---|---|
| **Description** | Developer adds `Console.WriteLine` or extra logging, pushes to dev/CI, waits 15-30 min, looks at output, repeats 5-15 times to track down a bug. |
| **Why it exists** | Can't attach a debugger or add a breakpoint locally. |
| **Hidden cost** | Per non-trivial bug: 5-15 round-trips × 20 min = 1.5-5 hours. Across the team this is *huge*. |
| **Risk** | Bugs that should take 30 min to fix take half a day. Cycle time slide directly. |
| **Removed by Docker + emulators?** | **Yes — fully.** Local debugging restored. |

### 5. Mocking Azure services in code instead of running real-equivalent

| | |
|---|---|
| **Description** | `IBlobStorageService` mocked in unit tests with a stub that returns canned responses, instead of running against a real Azurite instance. |
| **Why it exists** | Mocks are easy; running real services locally requires the infrastructure we don't have. |
| **Hidden cost** | Tests pass that don't actually test the integration. Bugs that *only* appear when wired to real Azure surface in the Development env, not locally. |
| **Risk** | False sense of safety. Drift between mocked behaviour and real behaviour. |
| **Removed by Docker + emulators?** | **Yes** — emulators give us "real but local" testing for blob, queue, table, Cosmos, SQL. |

### 6. Maintaining a personal/shadow Azure subscription for dev work

| | |
|---|---|
| **Description** | Devs spinning up their own Azure resources on a personal sub or a side project to test things they can't run locally. |
| **Why it exists** | The shared dev sub is always full or corrupted; personal sub is faster. |
| **Hidden cost** | Out-of-pocket developer expense; shadow IT from a security perspective; data going to subscriptions IT can't see. |
| **Risk** | Compliance violations; data leakage; shadow infrastructure. |
| **Removed by Docker + emulators?** | **Yes** — local emulators eliminate the need. |

### 7. Long-lived feature branches to avoid integration friction

| | |
|---|---|
| **Description** | Devs working on multi-day or multi-week branches to avoid the painful integration cycle. |
| **Why it exists** | Integration cycle hurts; minimise its frequency. |
| **Hidden cost** | Merge-conflict spike at end of work; loss of trunk-based-development benefits; harder code review. |
| **Risk** | Big-bang merges that break things; reduced collaboration; Conway's law penalties. |
| **Removed by Docker + emulators?** | **Partial** — local env makes integrating cheaply more attractive, but team must also adopt trunk-based practices. |

### 8. "Comment out and redeploy" debugging

| | |
|---|---|
| **Description** | To narrow down a bug, dev comments out chunks of code, pushes, redeploys, observes; iterates. |
| **Why it exists** | Same as #4 — no local debugging. |
| **Hidden cost** | Same as #4 — 5-15 round-trips per bug. |
| **Risk** | Forgetting to uncomment; merging "debug" code; pollutes git history. |
| **Removed by Docker + emulators?** | **Yes — fully.** |

### 9. Pair-debugging on a shared environment

| | |
|---|---|
| **Description** | Two developers blocked together on one shared Development env trying to reproduce a bug; one drives, one watches. |
| **Why it exists** | The bug only reproduces in the shared env. |
| **Hidden cost** | 2x developer time per bug. |
| **Risk** | Single point of access to env (others queued); collaboration friction. |
| **Removed by Docker + emulators?** | **Yes** — devs can each reproduce on their own machine. |

### 10. Skipping integration tests locally because they "only run in CI"

| | |
|---|---|
| **Description** | The integration test suite requires real Azure resources, so it's gated to CI. Devs ship without running it locally. |
| **Why it exists** | No way to spin up the resources locally. |
| **Hidden cost** | Devs find out about integration-test failures after pushing — adds a CI round-trip to their feedback loop. |
| **Risk** | Half-done features land in CI broken; PR feedback loop slowed. |
| **Removed by Docker + emulators?** | **Yes** — integration tests can run against emulators in seconds locally. |

### 11. Manual data-prep scripts to seed shared databases

| | |
|---|---|
| **Description** | Tribal-knowledge scripts in someone's `tools/` folder that re-seed the shared dev DB after another dev's tests corrupted it. |
| **Why it exists** | Shared mutable state. |
| **Hidden cost** | Onboarding tax for new devs; scripts rot; one-person bus factor; data inconsistency. |
| **Risk** | Tests pass on a particular seeded state; never reproducible. |
| **Removed by Docker + emulators?** | **Yes** — each dev has their own DB to seed however they like. |

### 12. Long onboarding ("you'll need to ask Frank for credentials...")

| | |
|---|---|
| **Description** | New developer spends 1-2 weeks getting access to dev environments, secrets, shared databases; can't write code productively until then. |
| **Why it exists** | Productive work requires shared resources, which require manual provisioning. |
| **Hidden cost** | Onboarding time × hourly rate × hire frequency. |
| **Risk** | New-hire frustration; perceived dysfunction during the most-impressionable period. |
| **Removed by Docker + emulators?** | **Yes** — `docker compose up` and they're productive in an hour. |

### 13. "It works in staging, your turn QA" handoffs

| | |
|---|---|
| **Description** | Developer can't verify their change end-to-end locally, so they push to a deployed environment and rely on QA to confirm it works. |
| **Why it exists** | No local environment to verify against. |
| **Hidden cost** | QA-shaped feedback loop instead of developer-shaped one; bugs found by QA are 10-15x more expensive than bugs found by the developer themselves. |
| **Risk** | Quality bottleneck on QA team; cycle time of QA pickup adds days. |
| **Removed by Docker + emulators?** | **Yes** — devs verify locally before code review. |

### 14. Disabling regression suite locally because it can't run

| | |
|---|---|
| **Description** | Regression suite is gated to CI only; devs can't verify they didn't break anything before they push. |
| **Why it exists** | Test suite needs Azure dependencies; we don't have local equivalents. |
| **Hidden cost** | Failed CI builds caused by trivially-detectable regressions; the entire failed-build-retry overhead in `azure-devops-pipeline-cost.md`. |
| **Risk** | False confidence in pre-push code; broken builds become normal. |
| **Removed by Docker + emulators?** | **Yes** — suite runs in seconds locally against emulators. |

---

## Self-assessment grid

Print this and fill it out as a team. The aggregate produces the slide-7 headline.

| # | Workaround | We do this? | Hours/week impact | Cost/year (hrs × 50 × $85) | Removed by investment? |
|---|---|---|---|---|---|
| 1 | Shared remote dev environment | Y / P / N | _____ | _____ | Yes |
| 2 | Hardcoded conn strings to Azure dev | Y / P / N | _____ | _____ | Yes |
| 3 | Disabled auth locally | Y / P / N | _____ | _____ | Yes |
| 4 | Push-to-test debugging | Y / P / N | _____ | _____ | Yes |
| 5 | Mocking Azure services | Y / P / N | _____ | _____ | Yes |
| 6 | Personal Azure subscriptions | Y / P / N | _____ | _____ | Yes |
| 7 | Long-lived feature branches | Y / P / N | _____ | _____ | Partial |
| 8 | Comment-out-and-redeploy debugging | Y / P / N | _____ | _____ | Yes |
| 9 | Pair-debugging on shared env | Y / P / N | _____ | _____ | Yes |
| 10 | Skipping local integration tests | Y / P / N | _____ | _____ | Yes |
| 11 | Manual data-prep scripts | Y / P / N | _____ | _____ | Yes |
| 12 | Long onboarding | Y / P / N | _____ | _____ | Yes |
| 13 | "Your turn QA" handoffs | Y / P / N | _____ | _____ | Yes |
| 14 | Disabled local regression suite | Y / P / N | _____ | _____ | Yes |
| **Total** | | **___ of 14** | **___ hrs/wk** | **$_____ /yr** | |

The total $/year on this row plugs directly into the **Workarounds** tab in `cost-model.xlsx`.

---

## A note on emulator parity

A common counter-argument to local emulators: *"They don't have full feature parity with real Azure."*

This is true and the business case acknowledges it. The position is:

- The 80% of Azure features that emulators *do* support cover ~95% of day-to-day developer needs.
- The remaining edge cases are explicitly tested in CI against real Azure (which still happens; we are not removing CI integration tests, just moving the *bulk* of dev work earlier).
- Net: most defects shift left, the residual class still has a safety net.

Specific parity status to verify before recommending each emulator:

| Service | Emulator | Parity quality |
|---|---|---|
| Blob / Queue / Table Storage | **Azurite** | Excellent |
| Cosmos DB | **Cosmos DB Linux Emulator** | Good (most APIs, occasional gaps) |
| SQL Server / Azure SQL | **SQL Server in Docker** or **Azure SQL Edge** | Excellent (some HA features absent) |
| Service Bus | **Service Bus Emulator (preview)** | Improving — verify currency at presentation time |
| Event Hubs | Limited; consider **Kafka in Docker** as a stand-in | Partial |
| Functions | **Azure Functions Core Tools** | Excellent |
| Identity (AAD) | **Microsoft.Identity.Web with dev secrets** or **Azurite + custom JWT** | Workable, requires care |
| Key Vault | **Local secrets file in dev** (with a safe-by-default pattern) | Conceptual rather than literal — fine in practice |

Emulator setup is a one-time engineering cost (estimated 40-80 hours total) that is included in the investment side of the ROI calculation.
