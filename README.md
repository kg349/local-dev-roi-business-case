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

### Prerequisites

| To do this | You need |
|------------|----------|
| Read the docs | Any markdown viewer (GitHub renders them automatically) |
| Open and edit the cost model | Microsoft Excel **or** LibreOffice Calc / Google Sheets (see *Without Excel* below) |
| Regenerate the spreadsheet from Python | Python 3.9+ and `pip install openpyxl` |
| Pull real data from your tools | Read access to Azure DevOps Analytics (or Jira), and a willing Finance/HR contact |

### Get the files

```bash
git clone https://github.com/kg349/local-dev-roi-business-case.git
cd local-dev-roi-business-case
```

### Pick your workflow based on how much time you have

#### If you have 30 minutes — *just understand the argument*

1. Read [`business-case.md`](business-case.md) (5-7 pages, the executive summary alone is one screen).
2. Open [`cost-model.xlsx`](cost-model.xlsx) and look at the **ROI-Summary** tab. The placeholder defaults will show you what the headline numbers look like for an 8-developer team at $85/hr.
3. You're now equipped to discuss the proposal. The numbers aren't *yours* yet, but the framing is sound.

#### If you have 1 day — *make it credible for your team*

1. Read [`business-case.md`](business-case.md) and [`shift-left-economics.md`](shift-left-economics.md).
2. Run the **top 3 priority queries** from [`data-gathering-checklist.md`](data-gathering-checklist.md) (defect distribution, hourly rate, headcount). About 2 hours of work; this is the highest-leverage investment.
3. Open [`cost-model.xlsx`](cost-model.xlsx), go to the **Inputs** tab, and replace the orange-shaded cells with your team's actual numbers. The ROI-Summary tab updates automatically.
4. Run the team self-assessment from [`developer-workarounds.md`](developer-workarounds.md) as a 30-minute meeting (~12 of your 14 workarounds will be a Yes, I'd bet).
5. You now have a defensible, your-team-specific case ready for management.

#### If you have 1 week — *the full bottom-up case*

In addition to the 1-day path:

1. Run **all** the queries in [`data-gathering-checklist.md`](data-gathering-checklist.md), including PR cycle time and pipeline failure rates.
2. Run the **1-week developer time-tracking template** (in `data-gathering-checklist.md` §7) — this measures inner-loop cycle time empirically rather than estimating it. Strongly worth one developer-week of effort.
3. Customise [`pitch-deck-outline.md`](pitch-deck-outline.md) with 1-2 specific recent integration-bug examples your team will recognise.
4. Build the actual slides in PowerPoint/Google Slides using the outline + speaker notes.
5. Present. Use the *Sensitivity* tab to defend the numbers under scrutiny ("here's what happens if every input is 25% worse than I claim — the case still holds").

### Regenerate the spreadsheet (optional, for power users)

The Python script lets you change the *placeholder defaults* and regenerate the xlsx, which is faster than editing the spreadsheet by hand if you're trying many scenarios.

```bash
pip install openpyxl
# Edit the defaults at the top of cost-model.py (the `Inputs` dataclass)
python cost-model.py
# Outputs: cost-model.xlsx + cost-model.csv
```

For one-off changes to your team's numbers, **just edit the Inputs tab directly in Excel** — much easier than touching Python.

### Without Excel

If you only have a CSV/spreadsheet viewer:

- [`cost-model.csv`](cost-model.csv) contains the headline outputs as a flat table.
- The xlsx will open fine in **LibreOffice Calc** or **Google Sheets** (upload to Drive → "Open with Google Sheets"). All formulas, named ranges, and formatting transfer correctly.
- The model uses only standard formulas (`SUM`, `IF`, etc.) — no Excel-specific features.

### Where to look in the workbook for what

| If you want to … | Look at this tab |
|---|---|
| See the bottom-line ROI and payback | **ROI-Summary** |
| Understand where bugs are caught today vs target | **Defect-Distribution-PCE** |
| See the per-step cost of the integration-fix workflow | **Integration-Fix-Workflow** |
| See the sprint-capacity erosion math | **Cycle-Time-Sprint** |
| Break down ADO pipeline cost (compute + idle + retries) | **Pipeline-Cost** |
| Tally the team's workaround cost | **Workarounds** |
| Stress-test the case at ±25% on each input | **Sensitivity** |
| Change any input | **Inputs** (orange cells are editable) |

### Two important toggles on the Inputs tab

- **`use_bottom_up_multiplier`** (default 1) — Use your team's actual 13-step PR workflow cost as the integration-stage multiplier instead of IBM's textbook 15x. Recommended ON because it's *your* number, not a citation. Set to 0 to fall back to IBM 15x and add the workflow tax as a separate line (no double-counting either way).
- **`apply_docker_license_cost`** (default 1) — Set to 0 if your org has fewer than 250 employees / under $10M revenue (Docker Desktop is free at that scale), or if you'll use Podman/Rancher instead.

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
