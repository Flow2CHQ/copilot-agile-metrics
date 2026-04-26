---
name: Burndown Chart
description: Generate a Mermaid burndown chart for the current sprint — ideal burndown line vs. actual burndown based on issue close dates.
argument-hint: "Sprint name or milestone (e.g. 'Sprint 12' or 'v2.1')"
agent: agent
---

## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**. Wait for all answers before proceeding.

---

**Please answer a few quick questions so I can generate the burndown chart:**

1. **How does your team track sprints in GitHub?**  
   For example: one Milestone per sprint, a GitHub Projects v2 Iteration field, labels like `sprint-42`, or a combination — describe it briefly.

2. **Which sprint do you want to chart?** Provide the milestone name, iteration name, or label — matching your answer above.

3. **Sprint start and end date** — answer depends on your tracking method:
   - *Milestone:* Leave blank — I'll derive the dates from the milestone's `due_on` and the previous milestone automatically.
   - *GitHub Projects v2 Iteration field:* Leave blank — I'll read `startDate` and `endDate` from the iteration directly.
   - *Labels, custom select field, or anything else:* Please provide both dates explicitly: **sprint start date** and **sprint end date** (format: YYYY-MM-DD). GitHub Copilot cannot read sprint boundaries from custom fields or labels.

---

Once you have the answers, use the sprint identifier and date information throughout all steps below. Derive `SPRINT_START_DATE` and `SPRINT_END_DATE` automatically only for Milestone and Iteration tracking — for all other methods, use the dates provided by the user.

> **How this burndown works:**
>
> This chart uses the `closedAt` date of issues to reconstruct how many issues were resolved on each day of the sprint.
> It shows two lines:
> - **Ideal burndown** — a straight line from the total issue count at sprint start down to 0 at sprint end
> - **Actual burndown** — the cumulative remaining issues, decreasing as issues are closed each day
>
> **Limitation:** The burndown is based on issue close dates only, not on within-sprint status transitions. If an issue was reopened and re-closed or was moved between columns without being closed, this chart will not reflect those intermediate states accurately.

---

You are helping a software team visualise sprint progress as a burndown chart.

## Step 1 — Load sprint data and derive date range

**Determine sprint boundaries:**

- If using a **GitHub Milestone:**
  1. Fetch the milestone matching the sprint identifier. Use its `due_on` date as `SPRINT_END_DATE`.
  2. Fetch all closed milestones in the repository, sorted by `due_on` descending. Find the most recently closed milestone whose `due_on` is before the current milestone's `due_on` — use that milestone's `due_on` as `SPRINT_START_DATE`.
  3. If no prior milestone exists or `due_on` is null for either milestone, ask the user for the missing date(s) before continuing.

- If using a **GitHub Projects v2 Iteration field:**
  1. Look up the iteration matching the sprint identifier. Use its `startDate` as `SPRINT_START_DATE` and `endDate` as `SPRINT_END_DATE`.
  2. If the iteration dates cannot be read via the GraphQL Projects API, inform the user and ask them to provide the dates manually (format: YYYY-MM-DD) before continuing.

- If using **labels, a custom select field, or any other method:**
  Use the start and end dates provided by the user in the setup answers. Do not attempt to derive dates from GitHub data — custom fields and labels do not contain sprint boundary information that GitHub Copilot can reliably access.

**Fetch all sprint issues:**

Fetch **all issues — both open and closed** — assigned to this sprint. The total count is `TOTAL_ISSUES_AT_SPRINT_START`.

For each **closed** issue, record the `closedAt` date (date portion only, strip the time component).

Note how many issues are still **open** as of today.

## Step 2 — Build the daily burndown series

For each calendar day from `SPRINT_START_DATE` to `SPRINT_END_DATE` (inclusive):

1. Count how many issues were closed **on or before** that day (cumulative closed count).
2. Remaining issues on that day = `TOTAL_ISSUES_AT_SPRINT_START` − cumulative closed count.
3. If a day is in the future (after today), mark it as a **forecast** point (see Step 3).

Produce a table:

| Date | Ideal Remaining | Actual Remaining | Status |
|------|----------------|-----------------|--------|
| SPRINT_START_DATE | TOTAL_ISSUES_AT_SPRINT_START | TOTAL_ISSUES_AT_SPRINT_START | actual |
| ... | ... | ... | actual |
| today | ... | ... | actual |
| ... | ... | ... | forecast |
| SPRINT_END_DATE | 0 | ... | forecast |

**Ideal remaining formula:**  
$\text{Ideal}_d = \text{TOTAL} \times \left(1 - \frac{d - \text{start}}{\text{end} - \text{start}}\right)$

where $d$ is the day number within the sprint.

## Step 3 — Forecast (if sprint is still running)

If today's date is before `SPRINT_END_DATE`, compute a simple linear projection:

- Current daily close rate = cumulative closed issues so far ÷ number of sprint days elapsed
- Projected remaining at sprint end = current remaining − (daily rate × days left until sprint end)
- Clamp projected remaining to a minimum of 0

Add the projected value for `SPRINT_END_DATE` to the forecast portion of the table.

## Step 4 — Generate the Mermaid chart

Using the data from Step 2 and Step 3, produce the following Mermaid `xychart-beta` diagram.

Include:
- The **ideal** line (straight, from total to 0)
- The **actual** line (based on real `closedAt` data)
- If the sprint is still running, include a **forecast** annotation in the summary text (add a sentence explaining the projected finish date)

Replace all `{placeholders}` with actual computed values before rendering. Ensure every date in the x-axis corresponds to a value in both line arrays (use `null` for future actual values so the line stops at today).

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Example markup to pass (with placeholders filled in):

xychart-beta
    title "Sprint Burndown — {CURRENT_SPRINT_MILESTONE}"
    x-axis [{comma-separated list of dates from SPRINT_START_DATE to SPRINT_END_DATE}]
    y-axis "Issues Remaining" 0 --> {TOTAL_ISSUES_AT_SPRINT_START}
    line "Ideal"  [{comma-separated ideal remaining values}]
    line "Actual" [{comma-separated actual remaining values, use null for future dates}]

---

#### 📋 Daily Data Table

Render the complete daily table from Step 2 here for reference.

---

#### 💡 Sprint Health Snapshot

Write 2–3 sentences covering: is the team ahead of, on track with, or behind the ideal burndown line? How many issues remain and how many days are left? If a projection was computed, is the team on track to finish by sprint end?

---

### 🔑 Key Findings

Based on the burndown data above, identify up to **3** of the most significant facts. Write each as one plain-language sentence with the relevant number.

*Look for: falling noticeably behind the ideal line early in the sprint, a large cumulative gap between actual and ideal, or an unusually high concentration of closes on a single day (possible sign of batch-closing at sprint end).*

---

### 💡 Recommendations for the Scrum Master

Write up to **3 concrete, actionable recommendations** based on the burndown pattern.

Each recommendation should:
- Start with a clear action verb (*"Discuss in the next stand-up…", "Consider pulling scope out…", "Watch for…", "Raise with the PO that…"*)
- Reference the specific burndown pattern that motivates it

If the burndown looks healthy, write: *The sprint is on track — continue as planned and check again tomorrow.*

---

> **Note:** The burndown is based on issue close dates only, not on status transitions within the sprint. For a full flow chart with in-progress state tracking, visit [flow2c.com](https://flow2c.com).
