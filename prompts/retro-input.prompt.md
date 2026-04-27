---
name: Retro Input
description: Generate retrospective input from the last sprint — identify patterns in labels, comments, and closed issues to fuel a productive team retrospective.
argument-hint: "Completed sprint name or milestone (e.g. 'Sprint 11')"
agent: agent
---

## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**. Wait for all answers before proceeding.

---

**Please answer a few quick questions so I can generate your retrospective input:**

1. **How does your team track sprints in GitHub?**  
   Describe your setup briefly — common examples:  
   - *One Milestone per sprint* (e.g. "Sprint 41" with a due date)  
   - *Labels* (e.g. `sprint-41`)  
   - *GitHub Projects v2* (Iteration field, custom select field, or other board-based tracking)  
   - *A combination or something else* — describe it in your own words  

2. **Which sprint do you want to retrospect?** Provide the milestone name, label, or — if you use GitHub Projects v2 — the sprint **start and end date** (YYYY-MM-DD).

3. **How many comments on an issue count as "high-discussion"?** *(Default: 3 — say "default" to accept.)*

---

Once you have the answers, apply the appropriate filter strategy:
- **Milestone:** filter by `milestone:"<name>"` server-side. Fall back to the most recently closed milestone if the sprint cannot be identified.
- **Label:** filter by `label:<value>` server-side.
- **GitHub Projects v2 (any field type):** No server-side filter is available. Use the start and end dates provided by the user: fetch all issues with `closedAt` between `SPRINT_START_DATE` and `SPRINT_END_DATE` as a proxy for sprint work. Note the scope limitation at the top of the output: *"⚠️ Scope: all issues closed in the date window — no Projects v2 sprint filter available."*

You are helping a software team prepare input for a sprint retrospective using data from GitHub Issues.

## Step 1 — Load last sprint data

**Data access — how to load issues:** Work through the following in order and use the first that succeeds. Do not attempt Python scripts, raw API calls, or other workarounds.

1. **VS Code GitHub extension tools** (preferred) — if the `github-pull-request_doSearch` tool is available, use it to fetch issues.
2. **GitHub CLI** — run `gh --version`. If exit code is 0, use `gh issue list --repo <owner/repo> --state all --json number,title,url,state,assignees,labels,createdAt,closedAt,comments --limit 200` with the appropriate filter flags.
3. **Neither available** — stop and tell the user: *"I can't load GitHub Issues automatically. Install the [GitHub Pull Request & Issues extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) or the [GitHub CLI](https://cli.github.com), then re-run this prompt — or paste your issue list here and I'll analyse it directly."*

Fetch all issues associated with the sprint identified in the setup answers (fall back to the most recently closed milestone if the sprint cannot be clearly identified).

Separate issues into two groups:
- **Closed issues** — resolved during the sprint
- **Remaining open issues** — not resolved by sprint end (if any)

For each issue collect:
- Issue number, title, URL
- Final state (closed / open)
- Labels
- Assignees
- Number of comments
- `createdAt` and `closedAt` (if closed)
- Whether the issue was re-opened at least once (check for `reopened` events if accessible, otherwise note if `createdAt` is significantly older than the sprint start — treat as a proxy)

## Step 2 — Analyse patterns

Compute the following from the closed issues:

1. **Label frequency** — count how many closed issues carry each label. Identify the top 3 most common labels.
2. **Bug ratio** — percentage of closed issues with a label containing `bug` or `fix` (case-insensitive).
3. **Blocked ratio** — percentage of closed issues that carried a `blocked`, `blocker`, or `impediment` label at some point (use current labels as a proxy).
4. **Average comments per issue** — total comments divided by number of closed issues.
5. **Issues closed without any comment** — count of closed issues with 0 comments.
6. **High-discussion issues** — closed issues with more than `HIGH_DISCUSSION_THRESHOLD` comments (default: 3).
7. **Unassigned closed issues** — issues that were closed without any assignee.

## Step 3 — Generate retrospective output

Output the results in the following Markdown structure:

---

### 🔄 Retrospective Input — {milestone name}

Sprint closed issues: **{count}** | Open / not resolved: **{count}**

---

#### ✅ What Went Well

List observations that indicate smooth delivery. Examples to check for and include if they apply:
- Issues closed quickly (below-average age at closing)
- No or few blocker labels
- Low comment count (clear requirements, little back-and-forth)
- High ratio of planned work completed

Present each observation as a bullet point with supporting data. If no clear positives are found, write: *No strong positive signals identified from the available data.*

---

#### 🔧 What to Improve

List observations that indicate friction or risk. Examples to check for and include if they apply:
- High bug / fix label ratio → potential quality issues
- Issues with many comments → unclear requirements or decision overhead
- Issues closed with no comments → potential lack of documentation or communication
- Unassigned issues → ownership gaps
- Issues re-opened → unexpected scope or unclear acceptance criteria
- Remaining open issues carried over → scope or capacity mismatch

Present each observation as a bullet point with the data behind it (e.g., "4 of 12 closed issues had a `bug` label — 33%").

---

#### ❓ Open Questions for the Team

Based on the patterns found above, suggest 3–5 discussion questions for the retrospective meeting. These should be concrete and tied to the data, for example:
- "Three issues were closed without any comments — were the tasks clear enough, or did communication happen outside GitHub?"
- "The `blocked` label appeared on {count} issues — what were the root causes and can we address them upstream?"

---

#### 📊 Data Summary

| Metric | Value |
|--------|-------|
| Closed issues | {count} |
| Open / carried over | {count} |
| Bug / fix label ratio | {x}% |
| Blocked label ratio | {x}% |
| Avg. comments per issue | {x.x} |
| Issues with 0 comments | {count} |
| High-discussion issues (>threshold comments) | {count} |
| Unassigned closed issues | {count} |
| Top labels | {label1}, {label2}, {label3} |

---

### 🔑 Key Findings

Based on the retrospective analysis above, identify up to **3** of the most significant observations from this sprint. Write each as one plain-language sentence with the supporting number from the data summary.

*Look for: a high bug ratio, many issues closed with zero comments (silent closes), carried-over blockers, or an unusually high comment count per issue.*

---

### 💡 Recommendations for the Scrum Master

Write up to **3 concrete, actionable recommendations** to bring to the retrospective meeting.

Each recommendation should:
- Be grounded in a specific data point from the analysis
- Start with a clear action verb (*"Ask the team about…", "Propose…", "Consider adding to the team norms…"*)
- Be achievable in the next sprint or at the next planning session

If all signals look positive, write: *Good sprint overall — focus the retro on sharing what worked and confirming the team wants to keep doing it.*

---

> **Note:** Trend analysis across multiple sprints and Time-in-State breakdowns require historical status-change data that GitHub Copilot cannot access. For automated trend analysis across sprints, visit [flow2c.com](https://flow2c.com).
