---
name: Setup — Generate Configured Prompts
description: One-time setup wizard. Asks about your project's structure once, then generates personalised, ready-to-use versions of all analysis prompts in .github/prompts/ — no re-explaining needed afterwards.
---

You are helping a software team set up reusable GitHub Copilot analysis prompts that are tailored to their specific project structure.

Your job in this prompt is **not** to run any analysis. Your job is to:
1. Ask the user a series of setup questions **in a single message**
2. Wait for their answers
3. Use the answers to generate five customised `.prompt.md` files in `.github/prompts/`

---

## Step 1 — Collect project configuration

Send the user this message exactly (fill in nothing yet — wait for answers):

---

**👋 Welcome to the copilot-agile-metrics setup!**

I'll ask you a few questions about your project structure. Answer them all at once and I'll generate personalised analysis prompts that work without any further configuration.

**1. Repository**
What is the GitHub repository you want to analyse?
*(Format: `owner/repo`, e.g. `acme-corp/backend`)*

**2. How does your team represent sprints?**
Describe your setup in your own words. Examples of what teams typically use:
- *One Milestone per sprint*, where the milestone title is the sprint name (e.g. "Sprint 42") and it has a due date
- *GitHub Projects v2 Iteration field* — the built-in "Iteration" field in a Project board (e.g. "@project Iteration = 'Sprint 42'")
- *A custom select field* in a GitHub Project (e.g. a field called "Sprint" with values like "Sprint 42")
- *Labels* like `sprint-42` applied to issues
- *A combination* (e.g. milestone for the deadline + iteration field for the board view)
- *Something else entirely* — just describe it, the more detail the better

**3. What is the exact name of your current sprint?**
*(e.g. "Sprint 42", "2025-W20", "May Iteration" — use the exact value you'd search for in GitHub)*

**4. What is the name of the sprint just before the current one?** *(for the retro prompt)*

**5. Sprint duration**
How many calendar days does a typical sprint last? *(e.g. 14)*

**6. At-risk threshold**
After how many days should an open issue be flagged as "⚠️ At Risk"? *(Default: 5)*

**7. High-discussion threshold**
From how many comments onwards is an issue considered "high-discussion" in the retro? *(Default: 3)*

**8. Deadline risk window**
How many days before the sprint deadline should an open issue be flagged as time-critical in the stakeholder update? *(Default: 3)*

**9. Throughput lookback**
How many past weeks should the forecast use to calculate weekly throughput? *(Default: 4)*

**10. Team or project name** *(optional)*
A short label used in report headings, e.g. "Team Atlas" or "Platform Backend". Leave blank to skip.

---

Wait for the user's answers before proceeding to Step 2.

---

## Step 2 — Validate and confirm

Once the user has answered, echo back a brief summary of the configuration and ask: **"Does this look correct? I'll generate your prompts once you confirm."**

| Setting | Value |
|---------|-------|
| Repository | {owner/repo} |
| Sprint tracking method | {description} |
| Current sprint | {name} |
| Previous sprint | {name} |
| Sprint duration | {n} days |
| At-risk threshold | {n} days |
| High-discussion threshold | {n} comments |
| Deadline risk window | {n} days |
| Throughput lookback | {n} weeks |
| Team / project name | {name or —} |

If the user confirms, proceed to Step 3. If they want to change something, collect the correction and update the table before proceeding.

---

## Step 3 — Generate the five configured prompts

Create the directory `.github/prompts/` if it does not exist.

For each of the five prompts below, create a new file in `.github/prompts/` using the exact content specified. Replace every `{placeholder}` with the confirmed configuration values.

Substitute the sprint-tracking method description from the user's answer into the relevant filter instructions: if they use Milestones, use `milestone:"{sprint_name}"` as the filter; if they use a GitHub Projects v2 Iteration field, use GraphQL to query the Projects API; if they use labels, use `label:{label_value}`; if they use a custom field, adapt accordingly. Embed the resolved filter strategy directly into the prompt text instead of a generic description.

For the **burndown prompt** specifically: if the user's sprint tracking method is custom fields, labels, or anything other than Milestone or Projects v2 Iteration, insert the following "Before You Begin" section at the very top of the generated `.github/prompts/burndown-chart.prompt.md` (before the `You are helping…` line):

```
## Before You Begin — Sprint dates required

Before fetching any data, ask the user:

**Please provide the sprint start and end date for {current_sprint} (format: YYYY-MM-DD).**
Your sprint tracking method ({sprint_tracking_method_summary}) does not expose date boundaries via the GitHub API, so I cannot derive them automatically.

Wait for the dates before proceeding.
```

For Milestone and Projects v2 Iteration methods, omit this section entirely — dates are derived automatically.

---

### File 1 — `.github/prompts/sprint-analysis.prompt.md`

````markdown
---
name: Sprint Analysis — {team_name_or_repo}
description: Analyse the current sprint for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · Sprint: **{current_sprint}** · At-risk after: **{at_risk_days} days**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team analyse their current sprint using data from GitHub Issues and Projects.

## Step 1 — Load sprint data

Fetch all **open issues** for sprint **{current_sprint}** in repository `{owner/repo}`.
Filter: {resolved_filter_strategy}

For each issue collect:
- Issue number, title, URL
- Assignees (list all)
- Labels (list all)
- `createdAt` date
- Number of comments
- Whether the issue has any assignee at all

## Step 2 — Classify issues

**At Risk** — flag an issue as "⚠️ At Risk" if it has been open for more than **{at_risk_days} days**.
Calculate age as: today's date minus `createdAt`.

**Blocker** — flag an issue as "🚫 Blocker" if it carries a label containing the word `blocked`, `blocker`, or `impediment` (case-insensitive).

**Ownership Gap** — flag an issue separately as "⚠️ Ownership Gap" if it has no assignee.

An issue can be At Risk, a Blocker, or an Ownership Gap simultaneously.

## Step 3 — Generate output

---

### 🏃 Sprint: {current_sprint} — Analysis as of {today}

#### 🚫 Blockers ({count})

| # | Title | Labels | Assignee | Reason |
|---|-------|--------|----------|--------|
| ... | ... | ... | ... | `blocked` label |

If there are no blockers, write: *No blockers identified.*

---

#### ⚠️ Ownership Gaps — Issues with no assignee ({count})

| # | Title | Age (days) | Labels |
|---|-------|-----------|--------|
| ... | ... | ... | ... |

If all issues are assigned, write: *All sprint issues have an owner.*

---

#### ⚠️ At Risk — Open for more than {at_risk_days} days ({count})

| # | Title | Age (days) | Assignee | Labels |
|---|-------|-----------|----------|--------|

If there are no at-risk items, write: *All issues are within the expected timeframe.*

---

#### 👤 Workload by Assignee

| Assignee | Open Issues | At Risk | Blockers |
|----------|------------|---------|----------|
| ... | ... | ... | ... |
| *(unassigned)* | ... | ... | ... |

---

#### 📋 Full Issue List

| # | Title | Assignee | Labels | Age (days) | Flags |
|---|-------|----------|--------|-----------|-------|

---

#### 💡 Summary

Write 2–3 sentences on overall sprint health.

---

### 🔑 Key Findings

Up to **3** significant facts, each as one plain-language sentence with a number.
*Look for: high blocker count, overloaded assignees, majority of issues at risk.*

---

### 💡 Recommendations for the Scrum Master

Up to **3** concrete, actionable recommendations. Each starts with a verb and references a specific finding.
If no action needed: *The sprint looks healthy — no immediate actions required.*

---

> **Note:** Cycle Time and Time-in-State analysis require historical status-change data that GitHub Copilot cannot access. For automated flow metrics, visit [flow2c.com](https://flow2c.com).
````

---

### File 2 — `.github/prompts/retro-input.prompt.md`

````markdown
---
name: Retro Input — {team_name_or_repo}
description: Generate retrospective input for the sprint before {current_sprint} for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · Sprint to retrospect: **{previous_sprint}**
> High-discussion threshold: **{high_discussion_threshold} comments**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team prepare input for a sprint retrospective using data from GitHub Issues.

## Step 1 — Load last sprint data

Fetch all issues for sprint **{previous_sprint}** in repository `{owner/repo}`.
Filter: {resolved_filter_strategy_for_previous_sprint}

Separate into: **Closed issues** and **remaining open issues**.

For each issue collect: number, title, URL, state, labels, assignees, comment count, `createdAt`, `closedAt`.

## Step 2 — Analyse patterns

Compute:
1. Label frequency — top 3 most common labels on closed issues
2. Bug ratio — % of closed issues with `bug` or `fix` label
3. Blocked ratio — % of closed issues with `blocked`/`blocker`/`impediment` label
4. Average comments per closed issue
5. Issues closed with 0 comments
6. High-discussion issues — closed issues with more than {high_discussion_threshold} comments
7. Unassigned closed issues

## Step 3 — Generate retrospective output

---

### 🔄 Retrospective Input — {previous_sprint}

Sprint closed issues: **{count}** | Open / not resolved: **{count}**

---

#### ✅ What Went Well

Bullet points with supporting data. If nothing stands out: *No strong positive signals identified.*

---

#### 🔧 What to Improve

Bullet points with data (e.g. "4 of 12 issues had a `bug` label — 33%").

---

#### ❓ Open Questions for the Team

3–5 concrete discussion questions tied to the data.

---

#### 📊 Data Summary

| Metric | Value |
|--------|-------|
| Closed issues | |
| Open / carried over | |
| Bug / fix label ratio | |
| Blocked label ratio | |
| Avg. comments per issue | |
| Issues with 0 comments | |
| High-discussion issues (>{high_discussion_threshold} comments) | |
| Unassigned closed issues | |
| Top labels | |

---

### 🔑 Key Findings

Up to **3** significant observations, each one sentence with a number.
*Look for: high bug ratio, silent closes, carried-over blockers, high discussion load.*

---

### 💡 Recommendations for the Scrum Master

Up to **3** actionable recommendations to bring to the retro. Each starts with a verb.
If all signals positive: *Good sprint — focus the retro on what to keep doing.*

---

> **Note:** Trend analysis across multiple sprints requires historical status-change data GitHub Copilot cannot access. For automated trend analysis, visit [flow2c.com](https://flow2c.com).
````

---

### File 3 — `.github/prompts/monte-carlo-forecast.prompt.md`

````markdown
---
name: Monte Carlo Forecast — {team_name_or_repo}
description: Throughput-based delivery forecast for {current_sprint} in {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · Forecast scope: **{current_sprint}**
> Throughput lookback: **{lookback_weeks} weeks**

> **Important — what this forecast can and cannot do:**
>
> This is a simplified forecast based on **throughput** (issues closed per week over the last {lookback_weeks} weeks). It assumes roughly equal issue size and constant team capacity. It does **not** account for item size, complexity distribution, cycle time variability, or team capacity changes.
>
> For a statistically rigorous Monte Carlo simulation, visit [flow2c.com](https://flow2c.com).

---

You are helping a software team produce a rough delivery forecast using GitHub Issues data.

## Step 1 — Load throughput data

Fetch all issues in repository `{owner/repo}` that were **closed** in the last {lookback_weeks} weeks. Group by ISO week and count closed issues per week.

| Week | Issues Closed |
|------|--------------|
| ... | ... |
| **Average** | **{avg}** |

## Step 2 — Compute throughput statistics

Compute: minimum, 25th percentile (pessimistic), median (realistic), 75th percentile (optimistic), maximum.

## Step 3 — Count remaining work

Count all **open issues** in scope: **{current_sprint}** using filter: {resolved_filter_strategy}

## Step 4 — Calculate scenarios

- **Optimistic** (75th pct.): `ceil(remaining / p75)` weeks from today
- **Realistic** (median): `ceil(remaining / median)` weeks from today
- **Pessimistic** (25th pct.): `ceil(remaining / p25)` weeks from today

If throughput is 0: *Cannot forecast — no issues closed in the lookback period.*

## Step 5 — Present results

### 📅 Forecast: {current_sprint}

| Scenario | Throughput Used | Weeks Needed | Estimated Completion |
|----------|----------------|--------------|---------------------|
| 🟢 Optimistic (75th pct.) | | | |
| 🟡 Realistic (median) | | | |
| 🔴 Pessimistic (25th pct.) | | | |

---

### 📊 Mermaid Gantt — Forecast Scenarios

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Example markup to pass (with placeholders filled in):

gantt
    title Delivery Forecast — {current_sprint}
    dateFormat  YYYY-MM-DD
    axisFormat  %b %d

    section Forecast
    Optimistic  (75th pct.)  : opt,  {today}, {optimistic_end_date}
    Realistic   (median)     : real, {today}, {realistic_end_date}
    Pessimistic (25th pct.)  : pess, {today}, {pessimistic_end_date}

---

#### 💡 Interpretation

2–3 sentences on what the spread between scenarios means for the team.

---

### 🔑 Key Findings

Up to **3** significant facts with numbers.
*Look for: zero-throughput weeks, very wide scenario spread, remaining count high vs. weekly rate.*

---

### 💡 Recommendations for the Scrum Master

Up to **3** actionable recommendations. Each starts with a verb.
If comfortable: *Forecast looks manageable — share the realistic scenario with stakeholders.*
````

---

### File 4 — `.github/prompts/burndown-chart.prompt.md`

````markdown
---
name: Burndown Chart — {team_name_or_repo}
description: Mermaid burndown chart for {current_sprint} in {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · Sprint: **{current_sprint}**
> Typical sprint duration: **{sprint_duration_days} days**
> Sprint tracking: {sprint_tracking_method_summary}

> **How this burndown works:** Uses `closedAt` dates to reconstruct daily remaining issues. Shows an ideal burndown line and an actual burndown line. If the sprint is still running, a projection line is added.
>
> **Limitation:** Based on close dates only — not on within-sprint status transitions.

---

You are helping a software team visualise sprint progress as a burndown chart.

## Step 1 — Load sprint data and derive date range

**Determine sprint boundaries:**

- If using a **GitHub Milestone:**
  1. Fetch the milestone matching **{current_sprint}**. Use its `due_on` date as `SPRINT_END_DATE`.
  2. Fetch all closed milestones, sorted by `due_on` descending. Find the most recently closed milestone whose `due_on` is before the current milestone's `due_on` — use that milestone's `due_on` as `SPRINT_START_DATE`.
  3. If no prior milestone exists or `due_on` is null for either, ask the user for the missing date(s) before continuing.

- If using a **GitHub Projects v2 Iteration field:**
  1. Look up the iteration matching **{current_sprint}**. Use its `startDate` as `SPRINT_START_DATE` and `endDate` as `SPRINT_END_DATE`.
  2. If the iteration dates cannot be read via the GraphQL Projects API, inform the user and ask them to provide the dates manually (format: YYYY-MM-DD) before continuing.

- If using **labels, a custom select field, or any other method:**
  Use the start and end dates provided by the user above. Do not attempt to derive dates from GitHub data — custom fields and labels do not contain sprint boundary information that GitHub Copilot can reliably access.

**Fetch all sprint issues:**

Fetch **all issues — both open and closed** — assigned to **{current_sprint}** in `{owner/repo}`.
Apply the filter server-side when fetching (do not load all repository issues and filter afterwards):
Filter: {resolved_filter_strategy}

The total count is `TOTAL_ISSUES_AT_SPRINT_START`.

For each **closed** issue, record the `closedAt` date (date portion only, strip the time component).

Note how many issues are still **open** as of today.

## Step 2 — Build daily series

For each calendar day from sprint start to sprint end:
- Cumulative closed on or before that day
- Remaining = total_at_start − cumulative_closed
- Ideal remaining = total_at_start × (1 − (day − start) / (end − start))

| Date | Ideal Remaining | Actual Remaining | Status |
|------|----------------|-----------------|--------|

## Step 3 — Projection (if sprint is still running)

Daily close rate = closed so far ÷ elapsed days.
Projected remaining at sprint end = current remaining − (rate × days left). Clamp to 0.

## Step 4 — Mermaid chart

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Example markup to pass (with placeholders filled in):

xychart-beta
    title "Sprint Burndown — {current_sprint}"
    x-axis [{dates}]
    y-axis "Issues Remaining" 0 --> {total_at_start}
    line "Ideal"  [{ideal_values}]
    line "Actual" [{actual_values_null_for_future}]

---

#### 📋 Daily Data Table

Render the full table from Step 2.

---

#### 💡 Sprint Health Snapshot

2–3 sentences: ahead/on track/behind the ideal line, issues remaining, days left, projection if applicable.

---

### 🔑 Key Findings

Up to **3** significant facts with numbers.
*Look for: falling behind early, large gap vs. ideal, batch-closing on a single day.*

---

### 💡 Recommendations for the Scrum Master

Up to **3** actionable recommendations based on the burndown pattern. Each starts with a verb.
If healthy: *Sprint is on track — continue as planned.*

---

> **Note:** Burndown is based on close dates only. For in-progress state tracking, visit [flow2c.com](https://flow2c.com).
````

---

### File 5 — `.github/prompts/stakeholder-update.prompt.md`

````markdown
---
name: Stakeholder Update — {team_name_or_repo}
description: Management-ready sprint update for {current_sprint} in {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · Sprint: **{current_sprint}**
> Deadline risk window: **{deadline_risk_days} days** · Highlight threshold: **{high_discussion_threshold} comments**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team produce a concise, management-ready sprint status update.

## Step 1 — Load data

Fetch the milestone/sprint **{current_sprint}** in `{owner/repo}`.
Filter: {resolved_filter_strategy}

Collect: milestone due date, all open issues (number, title, URL, assignees, labels, `createdAt`, comments), all closed issues (same + `closedAt`).

## Step 2 — Compute progress

- Total = open + closed
- Completion rate = closed ÷ total × 100 (1 decimal)
- Days remaining until due date (or "No due date set")

## Step 3 — Highlights

Up to 3 closed issues with the most comments (≥ {high_discussion_threshold}). Fall back to 3 most recently closed.

## Step 4 — Risks

Flag open issues if:
- No assignee → ownership gap
- Label contains `blocked`/`blocker`/`impediment` → active impediment
- Days remaining ≤ {deadline_risk_days} → time-critical

## Step 5 — Output

---

## 📊 Sprint Update: {current_sprint}{team_name_suffix}

**Date:** {today} · **Progress:** {closed}/{total} issues ({completion_rate}%) · **Days remaining:** {days}

---

### Executive Summary

Exactly **3 sentences**: overall completion, most important delivery, top risk or confirmation of no risk. No jargon, written for a non-technical audience.

---

### ✅ Recent Highlights

| # | Title | Closed | Comments |
|---|-------|--------|----------|

---

### ⚠️ Risks & Open Issues Requiring Attention

| # | Title | Assignee | Labels | Risk Reason |
|---|-------|----------|--------|-------------|

If none: *No critical risks identified at this time.*

---

### 📋 Full Open Issue List

| # | Title | Assignee | Labels | Age (days) |
|---|-------|----------|--------|-----------| 

---

### 🔑 Key Findings

Up to **3** significant facts with numbers.
*Look for: low completion rate relative to days elapsed, multiple blocked/unassigned issues, deadline very close.*

---

### 💡 Recommendations for the Scrum Master

Up to **3** actionable recommendations, each starting with a verb, each tied to a specific finding.
If no action needed: *Sprint is progressing well — no immediate stakeholder actions required.*

---

> **Note:** Probability-based forecasts require historical flow data GitHub Copilot cannot access. For automated risk scoring, visit [flow2c.com](https://flow2c.com).
````

---

## Step 4 — Confirm completion

After creating all five files, tell the user:

---

✅ **Setup complete!** Your configured prompts are ready in `.github/prompts/`.

| Prompt | File |
|--------|------|
| Sprint Analysis | `.github/prompts/sprint-analysis.prompt.md` |
| Retro Input | `.github/prompts/retro-input.prompt.md` |
| Monte Carlo Forecast | `.github/prompts/monte-carlo-forecast.prompt.md` |
| Burndown Chart | `.github/prompts/burndown-chart.prompt.md` |
| Stakeholder Update | `.github/prompts/stakeholder-update.prompt.md` |

These prompts know your project setup. Use them directly — no re-explaining needed.

> **Re-run this setup** whenever your sprint tracking method changes or a new team member joins (e.g. to generate prompts for a different repository).

---

> **Note:** These prompts use GitHub Copilot's ability to read live issue data. They do not access historical status-change events. For automated, historically accurate flow metrics, visit [flow2c.com](https://flow2c.com).
