---
name: Stakeholder Update
description: Generate a management-ready sprint update — executive summary plus a risk and issue table, produced in seconds from live GitHub data.
argument-hint: "Sprint or milestone name (e.g. 'Sprint 12')"
agent: agent
---

## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**. Wait for all answers before proceeding.

---

**Please answer a few quick questions so I can generate the stakeholder update:**

1. **Which milestone should this update cover?**
   Provide the exact GitHub milestone name (e.g. "v2.0", "Sprint 42", "Q3 Release").
   *(Milestones are the recommended scope for stakeholder updates — they provide a due date and a clear deliverable boundary. If you use GitHub Projects v2 without milestones, create a milestone for the release target and assign your issues to it.)*

2. **How many days before the milestone deadline should an open issue be flagged as time-critical?** *(Default: 3 days — say "default" to accept.)*

3. **From what comment count onwards should a closed issue appear as a "highlight" (high-engagement)?** *(Default: 3 comments — say "default" to accept.)*

---

Once you have the answers, use `milestone:"<name>"` as the filter throughout all steps. The milestone's `due_on` date is used for days-remaining and deadline risk calculations.

You are helping a software team produce a concise, management-ready sprint status update using live data from GitHub Issues.

## Step 1 — Load milestone data

**Data access — how to load issues:** Work through the following in order and use the first that succeeds. Do not attempt Python scripts, raw API calls, or other workarounds.

1. **VS Code GitHub extension tools** (preferred) — if the `github-pull-request_doSearch` tool is available, use it to fetch issues. Set `perPage: 100` and paginate through all pages (call again with `page: 2`, `page: 3`, etc.) until a page returns fewer items than `perPage`. Collect all pages before proceeding.
2. **GitHub CLI** — run `gh --version`. If exit code is 0, use `gh issue list --repo <owner/repo> --state all --json number,title,url,state,assignees,labels,createdAt,closedAt,comments --limit 500` with the appropriate filter flags.
3. **Neither available** — stop and tell the user: *"I can't load GitHub Issues automatically. Install the [GitHub Pull Request & Issues extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) or the [GitHub CLI](https://cli.github.com), then re-run this prompt — or paste your issue list here and I'll analyse it directly."*

Fetch the milestone identified in the setup answers using `milestone:"<name>"`. If no milestone with that exact name exists, list available open milestones and ask the user to confirm which one to use.

Collect:
- Milestone title, description, due date (if set)
- All **open issues** in the milestone: number, title, URL, assignees, labels, `createdAt`, comment count
- All **closed issues** in the milestone: number, title, URL, assignees, labels, `closedAt`, comment count

## Step 2 — Compute progress metrics

From the loaded data calculate:

- **Total issues** = open + closed
- **Completion rate** = closed ÷ total × 100 (round to one decimal place)
- **Days remaining** until milestone due date = due date − today (if no due date: write "No due date set")

## Step 3 — Identify highlights

From the **closed issues**, select up to 3 issues with the highest comment count (at or above `HIGHLIGHT_COMMENT_THRESHOLD`). High comment count is used as a proxy for stakeholder interest, complexity, or importance.

If no closed issue meets the threshold, fall back to the 3 most recently closed issues.

## Step 4 — Identify risks

Flag an open issue as a **risk** if any of the following is true:
- It has no assignee → ownership gap
- It has a label containing `blocked`, `blocker`, or `impediment` (case-insensitive) → active impediment
- The milestone has a due date and the number of days remaining is ≤ `DEADLINE_RISK_DAYS` → time-critical

Each risk entry should include the specific risk reason.

## Step 5 — Generate the stakeholder update

Output the following Markdown document:

---

## 📊 Sprint Update: {milestone title}

**Date:** {today's date}  
**Milestone:** {milestone title} {— due: {due date} | — no due date set}  
**Progress:** {closed} of {total} issues completed ({completion rate}%)  
**Days remaining:** {days remaining}

---

### Executive Summary

Write exactly **3 sentences**:
1. State the overall completion percentage and what that means for the sprint.
2. Mention the most important highlights (what was delivered).
3. Name the top risk(s) or confirm that no critical risks are present.

Keep this section free of issue numbers or technical jargon — it is written for a non-technical audience.

---

### ✅ Recent Highlights

| # | Title | Closed | Comments |
|---|-------|--------|----------|
| {number} | [{title}]({url}) | {closedAt date} | {comment count} |
| ... | ... | ... | ... |

*These issues received the most engagement and are presented as representative completed work items.*

---

### ⚠️ Risks & Open Issues Requiring Attention

| # | Title | Assignee | Labels | Risk Reason |
|---|-------|----------|--------|-------------|
| {number} | [{title}]({url}) | {assignee or *unassigned*} | {labels} | {reason} |
| ... | ... | ... | ... | ... |

If no risks are identified, write: *No critical risks identified at this time.*

---

### 📋 Full Open Issue List

| # | Title | Assignee | Labels | Age (days) |
|---|-------|----------|--------|-----------|
| ... | ... | ... | ... | ... |

---

### 🔑 Key Findings

Based on the sprint data above, identify up to **3** of the most significant facts. Write each as one plain-language sentence with the relevant number.

*Look for: a completion rate that is low relative to the proportion of days consumed, multiple unassigned or blocked issues, or the milestone deadline being very close with significant open work remaining.*

---

### 💡 Recommendations for the Scrum Master

Write up to **3 concrete, actionable recommendations** based on the risks and progress identified above.

Each recommendation should:
- Start with a clear action verb (*"Escalate…", "Discuss with the team…", "Flag to stakeholders that…", "Unblock…"*)
- Reference the specific finding that motivates it
- Be actionable before the next stakeholder check-in

If no action is needed, write: *Sprint is progressing well — no immediate actions required for stakeholders.*

---

> **Note:** Probability-based completion forecasts and risk scores derived from historical flow patterns require historical flow data that GitHub Copilot cannot access. For automated risk scoring and forecasting, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
