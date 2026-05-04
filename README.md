# Local Dev Environment ROI — Business Case Package

This folder contains a complete, management-ready business case for investing in a proper local development environment (Docker + Azure emulators) to fix a broken inner loop.

The argument is built around **shift-left economics**: a defect caught at the developer's desk costs ~1x; the same defect caught in the integration environment costs ~15x; in production, ~100x. We currently catch most defects too far right.

## What's in this folder

### Read these in order (~30 min total)

| # | File | Purpose | Audience |
|---|------|---------|----------|
| 1 | [`business-case.md`](business-case.md) | The main 5-7 page document. Exec summary, problem, model, ROI, recommendation. | Director / VP / CFO |
| 2 | [`shift-left-economics.md`](shift-left-economics.md) | The organising principle. Defines PCE/DDP, plots today's defect distribution vs target, dollarises the shift. | Engineering leadership |
| 3 | [`integration-fix-workflow-cost.md`](integration-fix-workflow-cost.md) | Itemised step-by-step cost of the current PR workflow that every integration-env bug triggers. **This is the slide that proves the multiplier with your own data, not textbook citations.** | Engineering managers |
| 4 | [`cycle-time-and-sprint-impact.md`](cycle-time-and-sprint-impact.md) | How the broken inner loop directly eats sprint capacity, story carry-over, and predictability. | Product + delivery managers |
| 5 | [`azure-devops-pipeline-cost.md`](azure-devops-pipeline-cost.md) | ADO pricing deep-dive: parallel jobs, per-minute cost, regression test minutes, queue wait as engineer idle. | Platform / DevOps |
| 6 | [`developer-workarounds.md`](developer-workarounds.md) | Catalogue of 14 workarounds devs do today + self-assessment checklist. | The whole team (fill this in as a group) |

### The numbers

| File | Purpose |
|------|---------|
| [`cost-model.py`](cost-model.py) | Python script (uses `openpyxl`) that generates the Excel model. Edit the defaults at the top to match your team. |
| [`cost-model.xlsx`](cost-model.xlsx) | Generated Excel model with live formulas. Tabs: Inputs, Defect-Distribution-PCE, Bug-Cost-by-Stage, Integration-Fix-Workflow, Pipeline-Cost, Cycle-Time-Sprint, Workarounds, ROI-Summary, Sensitivity. |
| [`cost-model.csv`](cost-model.csv) | Flat CSV fallback of the inputs and computed outputs. |

### The presentation

| File | Purpose |
|------|---------|
| [`pitch-deck-outline.md`](pitch-deck-outline.md) | 16-slide outline with bullet content and speaker notes. Each numeric claim references the spreadsheet cell it comes from. |
| [`data-gathering-checklist.md`](data-gathering-checklist.md) | Exact Azure DevOps / Jira queries (WIQL + KQL), HR/Finance questions, and a 1-week dev time-tracking template to replace the placeholders with your real numbers. |

## How to use this

1. **Skim** `business-case.md` to understand the overall argument.
2. **Run** `python cost-model.py` to regenerate `cost-model.xlsx`. (Requires `pip install openpyxl`.)
3. **Gather data** using `data-gathering-checklist.md`. The first hour of data collection makes the numbers ~10x more credible.
4. **Plug your numbers** into the Inputs tab of `cost-model.xlsx`. The ROI Summary tab updates automatically.
5. **Customise** `pitch-deck-outline.md` with your team's name and the spreadsheet outputs.
6. **Present.**

## Important framing notes

- The model is **parameterised**, not prescriptive. Every multiplier and rate is a placeholder you can override.
- We **do not** claim a specific dollar figure as your savings — the spreadsheet computes *your* number from *your* inputs.
- Industry benchmarks are cited from published, verifiable sources only: IBM Systems Sciences Institute, NIST RTI 2002, DORA *State of DevOps*, Capers Jones, Microsoft Azure DevOps pricing, Docker pricing.
- The recommended posture in front of management is **conservative**: lead with figures that survive a 25% downside sensitivity test. The Sensitivity tab shows you which.

## Sources cited in this package

- IBM Systems Sciences Institute — *Implementing Software Inspections*. Cost-of-defect cumulative multiplier curve (1x / 6.5x / 15x / 40x / 100x).
- NIST / RTI International (2002) — *The Economic Impacts of Inadequate Infrastructure for Software Testing*. Report 02-3.
- Capers Jones — *Software Engineering Best Practices* (2010) and *Applied Software Measurement*. Phase Containment Effectiveness benchmarks.
- DORA / Google Cloud — *State of DevOps Report* (annual). Lead time for changes, change-failure rate, MTTR by performer tier.
- Tom DeMarco & Timothy Lister — *Peopleware*. Context-switch productivity research.
- Microsoft — *Azure DevOps pricing* and *Inner-loop developer productivity research*.
- Docker — *Docker Desktop subscription pricing*.
