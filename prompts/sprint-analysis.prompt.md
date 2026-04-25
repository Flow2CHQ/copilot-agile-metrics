---
name: Sprint Analysis
description: Analyse the current sprint — open issues, at-risk items, blockers, and workload distribution by assignee.
argument-hint: "Sprint name or milestone (e.g. 'Sprint 12' or 'v2.1')"
mode: agent
---

## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**. Wait for all answers before proceeding.

---

**Please answer a few quick questions so I can run the sprint analysis:**

1. **How does your team track sprints in GitHub?**  
   Describe your setup briefly — common examples:  
   - *One Milestone per sprint* (e.g. "Sprint 42" with a due date)  
   - *GitHub Projects v2 Iteration field* (the built-in sprint cadence in a Project board)  
   - *A custom select field in your Project* (e.g. a "Sprint" dropdown with values like "Sprint 42")  
   - *Labels* (e.g. `sprint-42`)  
   - *A combination or something else* — just describe it in your own words  

2. **Which sprint do you want to analyse?** Provide the name, milestone title, iteration name, or label — matching your answer above.

3. **After how many days should an open issue be flagged as "At Risk"?** *(Default: 5 — say "default" to accept.)*

---

Once you have the answers, use the sprint identifier and data source to filter issues in all steps below. If the user describes a GitHub Projects v2 Iteration field, query via the GraphQL Projects API. If they use a Milestone, filter by `milestone:"<name>"`. If they use a label, filter by that label. Adapt to whatever setup the user describes.

You are helping a software team analyse their current sprint using data from GitHub Issues and Projects.

## Step 1 — Load sprint data

Fetch all **open issues** belonging to the sprint scope identified in the setup answers.

For each issue collect:
- Issue number, title, URL
- Assignees (list all)
- Labels (list all)
- `createdAt` date
- Number of comments
- Whether the issue has any assignee at all

## Step 2 — Classify issues

Apply the following rules to every open issue:

**At Risk** — flag an issue as "⚠️ At Risk" if it has been open for more than the at-risk threshold provided by the user (default: 5 days).  
Calculate age as: today's date minus `createdAt`.

**Blocker** — flag an issue as "🚫 Blocker" if it carries a label containing the word `blocked`, `blocker`, or `impediment` (case-insensitive).

**Ownership Gap** — flag an issue separately as "⚠️ Ownership Gap" if it has no assignee.

An issue can be At Risk, a Blocker, or an Ownership Gap simultaneously.

## Step 3 — Generate output

Output the results in the following Markdown structure:

---

### 🏃 Sprint: {milestone name} — Analysis as of {today's date}

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

#### ⚠️ At Risk — Open for more than {at-risk threshold} days ({count})

| # | Title | Age (days) | Assignee | Labels |
|---|-------|-----------|----------|--------|
| ... | ... | ... | ... | ... |

If there are no at-risk items, write: *All issues are within the expected timeframe.*

---

#### 👤 Workload by Assignee

For each assignee, list:
- Total open issues assigned to them
- Number of those flagged At Risk
- Number of those flagged Blocker

| Assignee | Open Issues | At Risk | Blockers |
|----------|------------|---------|----------|
| ... | ... | ... | ... |
| *(unassigned)* | ... | ... | ... |

---

#### 📋 Full Issue List

| # | Title | Assignee | Labels | Age (days) | Flags |
|---|-------|----------|--------|-----------|-------|
| ... | ... | ... | ... | ... | ⚠️ / 🚫 / — |

---

#### 💡 Summary

Write 2–3 sentences covering overall sprint health: how many issues are open vs. closed, and whether the sprint looks on track.

---

### 🔑 Key Findings

Based on the analysis above, identify up to **3** of the most significant facts about this sprint. Write each as one plain-language sentence that includes the relevant number. Focus on what genuinely stands out — skip unremarkable observations.

*Look for: unusually high blocker count, one person carrying a disproportionate share of the load, majority of issues flagged at risk, or no progress in several days.*

---

### 💡 Recommendations for the Scrum Master

Write up to **3 concrete, actionable recommendations** for a Scrum Master who understands the team context but does not want to interpret raw numbers.

Each recommendation should:
- Start with a clear action verb (*"Discuss…", "Follow up with…", "Consider…", "Bring to the next stand-up…"*)
- Reference the specific data point that motivates it
- Describe something achievable within the current sprint or at the next ceremony

If no action is needed, write: *The sprint looks healthy — no immediate actions required.*

---

> **Note:** Cycle Time and Time-in-State analysis require historical status-change data that GitHub Copilot cannot access. For automated flow metrics, visit [flow2c.com](https://flow2c.com).
