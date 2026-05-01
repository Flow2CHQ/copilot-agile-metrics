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

1. **How does your team track sprints in GitHub?** Choose the option that fits best:
   - **(a) Milestone** — one milestone per sprint, with a due date
   - **(b) Label** — issues get a label like `sprint-42`
   - **(c) GitHub Projects v2 or other** — sprint tracked in a Project board (Iteration field, custom select field, or anything else not covered by (a) or (b))

2. **Which sprint do you want to chart?**
   Provide the milestone title, label name, or iteration name — matching your answer above.

3. **Sprint start and end date** *(skip if using option a — I'll derive them from the milestone automatically)*
   Format: YYYY-MM-DD

4. **Total number of issues in this sprint** *(only needed for option c)*
   You can read this directly from the sprint view in GitHub Projects.

---

Once you have the answers, use the sprint identifier and date information throughout all steps below.

> **How this burndown works:**
>
> This chart uses the `closedAt` date of issues to reconstruct how many issues were resolved on each day of the sprint.
> It shows two lines:
> - **Ideal burndown** — a straight line from the total issue count at sprint start down to 0 at sprint end
> - **Actual burndown** — the cumulative remaining issues, decreasing as issues are closed each day
>
> **Limitation:** The burndown is based on issue close dates only, not on within-sprint status transitions. If an issue was reopened and re-closed, or moved between columns without being closed, this chart will not reflect those intermediate states accurately.
>
> **Fallback mode (option c):** When sprint tracking uses GitHub Projects v2 (any field type), the burndown is approximated using all issues closed in the repository since sprint start. This may include issues closed outside the sprint scope. A clean team process — closing issues only within their sprint — minimises this error.

---

You are helping a software team visualise sprint progress as a burndown chart.

## Step 1 — Load sprint data and derive date range

**Data access — how to load issues:** Work through the following in order and use the first that succeeds. Do not attempt Python scripts, raw API calls, or other workarounds.

1. **VS Code GitHub extension tools** (preferred) — if the `github-pull-request_doSearch` tool is available, use it to fetch issues. Set `perPage: 100` and paginate through all pages (call again with `page: 2`, `page: 3`, etc.) until a page returns fewer items than `perPage`. Collect all pages before proceeding.
2. **GitHub CLI** — run `gh --version`. If exit code is 0, use `gh issue list --repo <owner/repo> --state all --json number,title,url,state,createdAt,closedAt --limit 500` with the appropriate filter flags for the tracking method.
3. **Neither available** — stop and tell the user: *"I can't load GitHub Issues automatically. Install the [GitHub Pull Request & Issues extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) or the [GitHub CLI](https://cli.github.com), then re-run this prompt — or paste your issue list here and I'll analyse it directly."*

Apply the access method above to fetch issues in the appropriate case (a)–(c) below.

Follow the case that matches the user's tracking method:

**Case (a) — GitHub Milestone:**
1. Fetch the milestone matching the sprint identifier. Use its `due_on` date as `SPRINT_END_DATE`.
2. Fetch all closed milestones sorted by `due_on` descending. Find the most recently closed milestone whose `due_on` is before the current milestone's `due_on` — use that as `SPRINT_START_DATE`.
3. If no prior milestone exists or `due_on` is null, ask the user for the missing date(s) before continuing.
4. Fetch all issues (open and closed) filtered by `milestone:"sprint_identifier"` using the REST API. Apply the filter server-side — do not load all repository issues and filter afterwards.

**Case (b) — Label:**
1. Use the dates provided by the user as `SPRINT_START_DATE` and `SPRINT_END_DATE`.
2. Fetch all issues (open and closed) filtered by `label:sprint_identifier` using the REST API. Apply the filter server-side.

**Case (c) — GitHub Projects v2 or other:**
1. Use the dates provided by the user as `SPRINT_START_DATE` and `SPRINT_END_DATE`.
2. Use the total issue count provided by the user as `TOTAL_ISSUES_AT_SPRINT_START`.
3. Fetch all issues in the repository that were closed on or after `SPRINT_START_DATE` using `GET /repos/{owner}/{repo}/issues?state=closed&since=SPRINT_START_DATE`. This is the best available approximation — it may include issues closed outside the sprint scope.
4. Derive open count as: `TOTAL_ISSUES_AT_SPRINT_START` minus number of fetched closed issues. Clamp to a minimum of 0.

**For Cases (a) and (b):**
The total count of fetched issues is `TOTAL_ISSUES_AT_SPRINT_START`. For each **closed** issue, record the `closedAt` date (date portion only, strip the time component). Note how many issues are still **open** as of today.

## Step 2 — Build the daily burndown series

For each calendar day from `SPRINT_START_DATE` to `SPRINT_END_DATE` (inclusive):

1. Count how many issues were closed **on or before** that day (cumulative closed count).
2. Remaining issues on that day = `TOTAL_ISSUES_AT_SPRINT_START` − cumulative closed count.
3. **Round all remaining values to whole integers** — no decimals anywhere in this step.
4. If a day is in the future (after today), mark it as a **forecast** point (see Step 3) but **do not compute an actual value for it**.

Produce a table:

| Date | Ideal Remaining | Actual Remaining | Status |
|------|----------------|-----------------|--------|
| SPRINT_START_DATE | TOTAL_ISSUES_AT_SPRINT_START | TOTAL_ISSUES_AT_SPRINT_START | actual |
| ... | ... | ... | actual |
| today | ... | ... | actual |
| ... | ... | n/a | forecast |
| SPRINT_END_DATE | 0 | n/a | forecast |

**Ideal remaining formula:**  
$\text{Ideal}_d = \text{TOTAL} \times \left(1 - \frac{d - \text{start}}{\text{end} - \text{start}}\right)$

where $d$ is the day number within the sprint.

## Step 3 — Forecast (if sprint is still running)

If today's date is before `SPRINT_END_DATE`, compute a simple linear projection:

- Current daily close rate = cumulative closed issues so far ÷ number of sprint days elapsed
- Projected remaining at sprint end = current remaining − (daily rate × days left until sprint end)
- Clamp projected remaining to a minimum of 0
- **Do not render the projection as a third Mermaid line.** Include the projected finish date and remaining count in the written Sprint Health Snapshot instead.

## Step 4 — Generate the Mermaid chart

Using the data from Step 2, produce a Mermaid `xychart-beta` diagram with exactly **two lines**: Ideal and Actual.

Rules for renderer stability:
- **x-axis labels**: use short `MM-DD` format (e.g. `04-13, 04-14, 04-15`). Do **not** use full ISO dates like `2026-04-13`.
- **Only include days up to today** in both line arrays — do not extend arrays into future dates and do not use `null` or placeholder values.
- **All values must be whole integers** — no decimals.
- **Title**: keep it short (e.g. `"Burndown {CURRENT_SPRINT_MILESTONE}"`).
- **Two lines only**: `Ideal` and `Actual`. No projection line.

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Example markup to pass (with placeholders filled in):

xychart-beta
    title "Burndown {CURRENT_SPRINT_MILESTONE}"
    x-axis [04-13, 04-14, 04-15, 04-16]
    y-axis "Remaining Issues" 0 --> {TOTAL_ISSUES_AT_SPRINT_START}
    line "Ideal"  [19, 17, 15, 13]
    line "Actual" [19, 16, 16, 15]

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

> **Note:** The burndown is based on issue close dates only, not on status transitions within the sprint. For a full flow chart with in-progress state tracking, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
