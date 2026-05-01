---
name: Throughput Forecast
description: Throughput-based probabilistic forecast — estimate when the remaining issues in your current milestone will be done using three scenarios (optimistic, realistic, pessimistic) based on historical throughput percentiles.
argument-hint: "Milestone or scope to forecast (e.g. 'v3.0' or 'Backlog label:feature')"
agent: agent
---

## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**. Wait for all answers before proceeding.

---

**Please answer a few quick questions so I can run the forecast:**

1. **Which milestone or release target should this forecast cover?**
   Provide the exact GitHub milestone name (e.g. "v2.0", "Q3 Release", "Sprint 42").
   *(If your remaining work is not tracked in a GitHub milestone, enter the number of open issues in scope instead — e.g. "42 issues".)*

2. **How many past weeks should I use to calculate throughput?** *(Default: 4 weeks — say "default" to accept.)*

---

Once you have the answers:
- If the user provided a **milestone name**: use `milestone:"<name>"` to filter remaining open issues in Step 3.
- If the user provided a **number of issues directly**: use that number in Step 3 and skip the issue-fetch.

> **Important — what this forecast can and cannot do:**
>
> This is a simplified forecast based on **throughput** (number of issues closed per week over the last `LOOKBACK_WEEKS` weeks). It assumes all issues are roughly equal in size and complexity, and that past throughput continues at the same rate.
>
> It does **not** account for:
> - Item size or story point estimates
> - Complexity distribution
> - Historical cycle time distribution
> - Team capacity changes, holidays, or parallel workstreams
>
> For a statistically rigorous Monte Carlo simulation that uses real cycle time distributions and models variability properly, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).

---

You are helping a software team produce a rough delivery forecast using GitHub Issues data.

## Step 1 — Load throughput data

**Data access — how to load issues:** Work through the following in order and use the first that succeeds. Do not attempt Python scripts, raw API calls, or other workarounds.

1. **VS Code GitHub extension tools** (preferred) — if the `github-pull-request_doSearch` tool is available, use it to fetch issues. Set `perPage: 100` and paginate through all pages (call again with `page: 2`, `page: 3`, etc.) until a page returns fewer items than `perPage`. Collect all pages before proceeding.
2. **GitHub CLI** — run `gh --version`. If exit code is 0, use `gh issue list --repo <owner/repo> --state closed --json number,title,url,closedAt --limit 500` to retrieve recently closed issues.
3. **Neither available** — stop and tell the user: *"I can't load GitHub Issues automatically. Install the [GitHub Pull Request & Issues extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) or the [GitHub CLI](https://cli.github.com), then re-run this prompt — or paste your issue list here and I'll analyse it directly."*

Use the GitHub API to fetch all issues in this repository that:
- Were **closed** within the last `LOOKBACK_WEEKS` weeks (default: 4)
- Have a `closedAt` timestamp

Group them by ISO week (week number + year). For each week, count the number of issues closed.

Today's date: use the current date as the reference point.

## Step 2 — Compute throughput statistics

From the weekly closed counts, compute:
- **Minimum** closed per week
- **25th percentile** (pessimistic throughput) — if fewer than 4 data points, use the minimum
- **Median** (realistic throughput)
- **75th percentile** (optimistic throughput) — if fewer than 4 data points, use the maximum
- **Maximum** closed per week
- **Average** closed per week

Show the weekly breakdown as a table:

| Week | Issues Closed |
|------|--------------|
| {ISO week} | {count} |
| ... | ... |
| **Average** | **{avg}** |

## Step 3 — Count remaining work

**If a milestone name was provided:** Fetch all **open issues** filtered by `milestone:"<name>"` and count them.

**If a number was provided directly:** Use that number as the remaining issue count.

If no milestone exists with the given name and no number was provided, ask the user: *"I couldn't find a milestone with that name. How many open issues are currently in scope? (Enter a number.)"*

Remaining issues: **{count}**

## Step 4 — Calculate forecast scenarios

For each scenario, calculate the number of weeks needed to close all remaining issues:

- **Optimistic** (75th-percentile throughput): `ceil(remaining / p75_throughput)` weeks
- **Realistic** (median throughput): `ceil(remaining / median_throughput)` weeks  
- **Pessimistic** (25th-percentile throughput): `ceil(remaining / p25_throughput)` weeks

Then convert weeks to a calendar date by adding the week count to today's date.

If throughput for a scenario is 0 or undefined, write: *"Cannot forecast — no issues closed in the relevant period."*

## Step 5 — Present results

### 📅 Forecast: When will {milestone name} be done?

Based on **{total closed issues} issues closed** over the last **{lookback weeks} weeks**  
Remaining open issues in scope: **{count}**

| Scenario | Weekly Throughput Used | Weeks Needed | Estimated Completion |
|----------|----------------------|--------------|---------------------|
| 🟢 Optimistic (75th pct.) | {p75} issues/week | {weeks} | {date} |
| 🟡 Realistic (median) | {median} issues/week | {weeks} | {date} |
| 🔴 Pessimistic (25th pct.) | {p25} issues/week | {weeks} | {date} |

---

### 📊 Mermaid Gantt — Forecast Scenarios

Generate a Mermaid Gantt diagram showing the three completion scenarios. Use today as the start date and the three estimated completion dates as end dates.

Replace `{today}`, `{optimistic_end_date}`, `{realistic_end_date}`, and `{pessimistic_end_date}` with the actual computed dates in `YYYY-MM-DD` format.

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Example markup to pass (with placeholders filled in):

gantt
    title Delivery Forecast — {milestone name}
    dateFormat  YYYY-MM-DD
    axisFormat  %b %d

    section Forecast
    Optimistic  (75th pct.)  : opt,  {today}, {optimistic_end_date}
    Realistic   (median)     : real, {today}, {realistic_end_date}
    Pessimistic (25th pct.)  : pess, {today}, {pessimistic_end_date}

---

#### 💡 Interpretation

Write 2–3 sentences interpreting the spread between scenarios. A wide spread (many weeks between optimistic and pessimistic) indicates high variability in the team's throughput — consider stabilising the flow before committing to a deadline.

---

### 🔑 Key Findings

Based on the forecast above, identify up to **3** of the most significant facts. Write each as one plain-language sentence with the relevant number.

*Look for: low or zero throughput weeks, a very wide spread between optimistic and pessimistic completion dates, or a remaining issue count that is high relative to the team's weekly throughput.*

---

### 💡 Recommendations for the Scrum Master

Write up to **3 concrete, actionable recommendations** based on the forecast results. Address a Scrum Master who needs clear guidance, not statistical interpretation.

Each recommendation should:
- Start with a clear action verb (*"Share with stakeholders that…", "Consider descoping…", "Discuss with the team whether…", "Monitor throughput weekly and flag if…"*)
- Reference the specific finding that motivates it

If the forecast looks comfortable, write: *The forecast looks manageable — share the realistic scenario with stakeholders and track throughput weekly.*

---

> **Note:** This forecast is based on throughput only and does not account for cycle time distributions or team capacity changes. For a statistically rigorous Monte Carlo simulation, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
