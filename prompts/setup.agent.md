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

## Step 0 — Detect data access method

**Do this silently before asking the user anything.** Determine which method is available to load GitHub Issues in this environment. Try each option in order and stop at the first that succeeds:

1. **VS Code GitHub extension** — call the `github-pull-request_doSearch` tool with the query `is:issue is:open`. If it responds (even with an empty result), set `access_method = copilot-extension` (display label: "VS Code GitHub extension").
2. **GitHub CLI** — run `gh --version` in the terminal. If exit code is 0, set `access_method = gh-cli` (display label: "GitHub CLI").
3. If both fail, set `access_method = none`.

**If `access_method = none`:** Output the message below and wait for the user's reply before continuing to Step 1.

---

> ⚠️ **No automatic GitHub data access method detected.**
>
> The prompts this setup generates need a way to read issues from your GitHub repository. Currently, neither of the two supported methods responded:
>
> - **VS Code GitHub Pull Request & Issues extension** — [install from the Marketplace](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) and sign in with GitHub.
> - **GitHub CLI** — [install from cli.github.com](https://cli.github.com), then run `gh auth login`.
>
> Re-run this setup after installing one of the above.
> Or type **continue** to proceed anyway — the generated prompts will include manual data-entry instructions.

---

If the user types **continue**, set `access_method = manual` (display label: "Manual — user provides data"). Proceed to Step 1.

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

**3. Sprint name (for label-based tracking only)**
*(If your team uses labels like `sprint-42`, enter the current sprint label value here — e.g. `sprint-42`. If you use Milestones or GitHub Projects v2, leave blank — sprint names are resolved at runtime from the API.)*

**4. Previous sprint label (for label-based tracking only)**
*(Only needed for the retro prompt. If you use labels, enter the label value for the sprint just before the current one — e.g. `sprint-41`. Leave blank otherwise.)*

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
| Issue access method | {access_method_label} |
| Sprint tracking method | {description} |
| Current sprint label | {label value or — (only needed for label tracking)} |
| Previous sprint label | {label value or — (only needed for label tracking)} |
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

Substitute the sprint-tracking method description from the user's answer into the relevant filter instructions:
- **Milestone:** use `milestone:"{sprint_name}"` as the filter.
- **Label:** use `label:{label_value}` as the filter.
- **GitHub Projects v2 (any field type):** No filter can be pre-configured. For Sprint Analysis, embed: *"fetch all open repository issues — ask for sprint start date at runtime"*. For Retro Input, embed: *"fetch all issues closed in the sprint date window — ask for start and end date at runtime"*. For Burndown, Case (c) applies automatically.
- **Throughput Forecast and Stakeholder Update** always use `milestone:"{sprint_name}"` regardless of sprint tracking method — these two prompts are milestone-scoped, not sprint-scoped.

For the **burndown prompt** specifically: the generated prompt already contains all three cases (a–c) with the correct "Before You Begin" questions. Do not add a separate date-ask section — the prompt handles all tracking methods including the fallback automatically.

**Sprint scope selection substitution:** Replace the three sprint-selection placeholders based on the sprint tracking method:

Replace `{sprint_scope_selection_instructions}` (Sprint Analysis — current sprint) with:
- **Milestone:** `Fetch all open milestones in \`{owner/repo}\` and list them numbered by due date (earliest first). Ask: "Which sprint milestone should I analyse? (Enter number, default: 1)". Store title as ACTIVE_SPRINT_NAME. ACTIVE_SPRINT_FILTER = milestone:"ACTIVE_SPRINT_NAME".`
- **Label:** `Ask: "Sprint label? (default: {label_value})". Use the entered value (or default) as ACTIVE_SPRINT_NAME for headings. ACTIVE_SPRINT_FILTER = label:ACTIVE_SPRINT_NAME.`
- **Projects v2:** `Ask: "Sprint start date? (YYYY-MM-DD)". ACTIVE_SPRINT_FILTER = all open issues. ACTIVE_SPRINT_NAME is not required (use the start date in headings).`

Replace `{retro_sprint_scope_selection_instructions}` (Retro Input — last completed sprint) with:
- **Milestone:** `Fetch all closed milestones in \`{owner/repo}\`, sorted by closed date descending. Use the most recently closed one automatically — no user input needed. Store title as RETRO_SPRINT_NAME. RETRO_SPRINT_FILTER = milestone:"RETRO_SPRINT_NAME".`
- **Label:** `Ask: "Sprint label for the previous sprint? (default: {previous_label_value})". Use the entered value (or default) as RETRO_SPRINT_NAME. RETRO_SPRINT_FILTER = label:RETRO_SPRINT_NAME.`
- **Projects v2:** `Ask: "Sprint start date? (YYYY-MM-DD)" and "Sprint end date? (YYYY-MM-DD)". RETRO_SPRINT_FILTER = issues closed between those dates (closedAt >= start AND closedAt <= end). RETRO_SPRINT_NAME = not required (use date range in headings).`

Replace `{burndown_sprint_scope_selection_instructions}` (Burndown Chart — sprint selection) with:
- **Milestone:** `Ask: "Which option matches your tracking? (a) GitHub Milestone, (b) Label, (c) GitHub Projects v2 or other". For (a): list open milestones numbered, user picks (default: 1). Store title as ACTIVE_SPRINT_NAME. Case (a) flow applies.`
- **Label:** Same question. For (b): ask "Sprint label? (default: {label_value})". Store as ACTIVE_SPRINT_NAME. Case (b) flow applies.
- **Projects v2:** Same question. For (c): no name needed. Case (c) flow applies.
- *(Note: embed the full three-option question regardless of tracking method — the prompt handles all cases)*

**Access method substitution:** In each prompt's Step 1, replace `{access_method_instructions}` with the appropriate instruction based on `access_method` from Step 0:

- **`copilot-extension`:** Instruct the prompt to use the built-in VS Code GitHub extension tools (e.g. `github-pull-request_doSearch`) to fetch issues. Set `perPage: 100` and paginate through all pages (call again with `page: 2`, `page: 3`, etc.) until a page returns fewer items than `perPage`. Collect all pages before proceeding. Explicitly state: do not use scripts or CLI commands.
- **`gh-cli`:** Instruct the prompt to use GitHub CLI: `gh issue list --repo {owner/repo} --state open {filter_flags} --json number,title,url,assignees,labels,createdAt,closedAt,comments --limit 200`. Map the sprint filter to the appropriate flag (`--milestone "{sprint_name}"` for milestones, `--label {label_value}` for labels). For Projects v2, no filter flag is available — fetch all open issues.
- **`manual`:** Instruct the prompt to ask the user to paste their issue list (from the GitHub web interface, a CSV export, or `gh issue list` output) and analyse the provided data. Include this message in the prompt: *"Automatic data loading is not available. Paste your issue list below."*

---

### File 1 — `.github/prompts/sprint-analysis.prompt.md`

````markdown
---
name: Sprint Analysis — {team_name_or_repo}
description: Analyse the current sprint for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}` · At-risk after: **{at_risk_days} days**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team analyse their current sprint using data from GitHub Issues and Projects.

## Before You Begin

Before fetching any data, identify the sprint scope:

{sprint_scope_selection_instructions}

Store the result as `ACTIVE_SPRINT_NAME` (for headings) and `ACTIVE_SPRINT_FILTER` (for API calls).

## Step 1 — Load sprint data

**Data access:** {access_method_instructions}

Fetch all **open issues** in repository `{owner/repo}`.
Filter: `ACTIVE_SPRINT_FILTER`

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

### 🏃 Sprint: `ACTIVE_SPRINT_NAME` — Analysis as of {today}

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

> **Note:** Cycle Time and Time-in-State analysis require historical status-change data that GitHub Copilot cannot access. For automated flow metrics, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
````

---

### File 2 — `.github/prompts/retro-input.prompt.md`

````markdown
---
name: Retro Input — {team_name_or_repo}
description: Generate retrospective input for the last completed sprint for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}`
> High-discussion threshold: **{high_discussion_threshold} comments**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team prepare input for a sprint retrospective using data from GitHub Issues.

## Before You Begin

Before fetching any data, identify the sprint to retrospect:

{retro_sprint_scope_selection_instructions}

Store the result as `RETRO_SPRINT_NAME` (for headings) and `RETRO_SPRINT_FILTER` (for API calls).

## Step 1 — Load last sprint data

**Data access:** {access_method_instructions}

Fetch all issues in repository `{owner/repo}`.
Filter: `RETRO_SPRINT_FILTER`

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

### 🔄 Retrospective Input — `RETRO_SPRINT_NAME`

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

> **Note:** Trend analysis across multiple sprints and Time-in-State breakdowns require historical status-change data that GitHub Copilot cannot access. For automated trend analysis across sprints, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
````

---

### File 3 — `.github/prompts/throughput-forecast.prompt.md`

````markdown
---
name: Throughput Forecast — {team_name_or_repo}
description: Throughput-based delivery forecast for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}`
> Throughput lookback: **{lookback_weeks} weeks**

> **Important — what this forecast can and cannot do:**
>
> This is a simplified forecast based on **throughput** (issues closed per week over the last {lookback_weeks} weeks). It assumes roughly equal issue size and constant team capacity. It does **not** account for item size, complexity distribution, cycle time variability, or team capacity changes.
>
> For a statistically rigorous Monte Carlo simulation that uses real cycle time distributions and models variability properly, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).

---

You are helping a software team produce a rough delivery forecast using GitHub Issues data.

## Step 1 — Load throughput data

**Data access:** {access_method_instructions}

Fetch all issues in repository `{owner/repo}` that were **closed** in the last {lookback_weeks} weeks. Group by ISO week and count closed issues per week.

| Week | Issues Closed |
|------|--------------|
| ... | ... |
| **Average** | **{avg}** |

## Step 2 — Compute throughput statistics

Compute: minimum, 25th percentile (pessimistic), median (realistic), 75th percentile (optimistic), maximum.

## Step 3 — Select milestone and count remaining work

Fetch all open milestones in `{owner/repo}` and present them as a numbered list:

```
Open milestones:
  1. <title> (due <due_on date or "no due date">)
  2. ...
Which milestone should I forecast? (Enter number, default: 1 — the one with the earliest due date)
```

Wait for the user's answer. Store the selected milestone title as `FORECAST_MILESTONE`.

If no milestones exist, ask: *"No open milestones found. How many open issues are currently in scope? (Enter a number.)"* and use that count directly.

Fetch all **open issues** filtered by `milestone:"FORECAST_MILESTONE"` and count them.

Remaining issues: **{count}**

## Step 4 — Calculate scenarios

- **Optimistic** (75th pct.): `ceil(remaining / p75)` weeks from today
- **Realistic** (median): `ceil(remaining / median)` weeks from today
- **Pessimistic** (25th pct.): `ceil(remaining / p25)` weeks from today

If throughput is 0: *Cannot forecast — no issues closed in the lookback period.*

## Step 5 — Present results

### 📅 Forecast: `FORECAST_MILESTONE`

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
    title Delivery Forecast — `FORECAST_MILESTONE`
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

---

> **Note:** This forecast is based on throughput only and does not account for cycle time distributions or team capacity changes. For a statistically rigorous Monte Carlo simulation, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
````

---

### File 4 — `.github/prompts/burndown-chart.prompt.md`

````markdown
---
name: Burndown Chart — {team_name_or_repo}
description: Mermaid burndown chart for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}`
> Typical sprint duration: **{sprint_duration_days} days**
> Sprint tracking: {sprint_tracking_method_summary}

> **How this burndown works:** Uses `closedAt` dates to reconstruct daily remaining issues. Shows an ideal burndown line and an actual burndown line. If the sprint is still running, the projected finish is shown in the written summary — not as a third chart line.
>
> **Limitation:** Based on close dates only — not on within-sprint status transitions.
>
> **Fallback mode (option c):** When sprint tracking uses GitHub Projects v2 (any field type), the burndown is approximated using all issues closed in the repository since sprint start. This may include issues closed outside the sprint scope. A clean team process minimises this error.

## Before You Begin

Before fetching any data, identify the sprint:

{burndown_sprint_scope_selection_instructions}

Store the result as `ACTIVE_SPRINT_NAME` (for headings) and `ACTIVE_SPRINT_FILTER` or date range as appropriate for the case below.

Then ask any remaining questions for the selected case:

**If Case (a) — Milestone:** No further questions needed — dates are derived from the milestone.

**If Case (b) — Label:** Ask: *"Sprint start date? (YYYY-MM-DD) — end date will be calculated as start + {sprint_duration_days} days."*

**If Case (c) — Projects v2:** Ask: *"Sprint start date? (YYYY-MM-DD)"* and *"Total number of issues in this sprint?"* — end date will be calculated as start + {sprint_duration_days} days. No sprint name is needed.

Wait for answers before proceeding.

---

You are helping a software team visualise sprint progress as a burndown chart.

## Step 1 — Load sprint data and derive date range

**Data access:** {access_method_instructions}

Follow the case that matches the user's tracking method:

**Case (a) — GitHub Milestone:**
1. `ACTIVE_SPRINT_NAME` and filter are already set from the scope selection step.
2. Fetch the milestone matching `ACTIVE_SPRINT_NAME`. Use its `due_on` date as `SPRINT_END_DATE`.
2. Fetch all closed milestones sorted by `due_on` descending. Find the most recently closed milestone whose `due_on` is before the current milestone's `due_on` — use that as `SPRINT_START_DATE`.
3. If no prior milestone exists or `due_on` is null, ask the user for the missing date(s) before continuing.
4. Fetch all issues (open and closed) filtered by `milestone:"ACTIVE_SPRINT_NAME"` using the REST API. Apply the filter server-side — do not load all repository issues and filter afterwards.

**Case (b) — Label:**
1. Use the start date provided by the user as `SPRINT_START_DATE`. Compute `SPRINT_END_DATE` = `SPRINT_START_DATE` + {sprint_duration_days} − 1 days.
2. Fetch all issues (open and closed) filtered by `label:{label_value}` using the REST API. Apply the filter server-side.

**Case (c) — GitHub Projects v2 or other:**
1. Use the start date provided by the user as `SPRINT_START_DATE`. Compute `SPRINT_END_DATE` = `SPRINT_START_DATE` + {sprint_duration_days} − 1 days.
2. Use the total issue count provided by the user as `TOTAL_ISSUES_AT_SPRINT_START`.
3. Fetch all issues in the repository that were closed on or after `SPRINT_START_DATE` using `GET /repos/{owner/repo}/issues?state=closed&since=SPRINT_START_DATE`. This is the best available approximation — it may include issues closed outside the sprint scope.
4. Derive open count as: `TOTAL_ISSUES_AT_SPRINT_START` minus number of fetched closed issues. Clamp to a minimum of 0.

**For Cases (a) and (b):**
The total count of fetched issues is `TOTAL_ISSUES_AT_SPRINT_START`. For each **closed** issue, record the `closedAt` date (date portion only, strip the time component). Note how many issues are still **open** as of today.

## Step 2 — Build daily series

For each calendar day from sprint start to sprint end:
- Cumulative closed on or before that day
- Remaining = total_at_start − cumulative_closed (**round to whole integer**)
- Ideal remaining = total_at_start × (1 − (day − start) / (end − start)) (**round to whole integer**)
- For days in the future (after today): record ideal value only, **no actual value**

| Date | Ideal Remaining | Actual Remaining | Status |
|------|----------------|-----------------|--------|

## Step 3 — Projection (if sprint is still running)

Daily close rate = closed so far ÷ elapsed days.
Projected remaining at sprint end = current remaining − (rate × days left). Clamp to 0.
**Do not render projection as a Mermaid line.** Include projected finish date in the Sprint Health Snapshot text.

## Step 4 — Mermaid chart

Render the diagram by calling the renderMermaidDiagram tool with the Mermaid markup directly — do NOT output a fenced code block. Pass only the raw diagram markup (without ```mermaid wrapper) as the markup parameter, and a short descriptive title as the title parameter.

Rules for renderer stability:
- **x-axis labels**: short `MM-DD` format only (e.g. `04-13, 04-14`). No full ISO dates.
- **Only days up to today** in both line arrays — no future dates, no `null` values.
- **All values: whole integers only.**
- **Two lines only**: `Ideal` and `Actual`.
- **Title**: keep short.

Example markup to pass (with placeholders filled in):

xychart-beta
    title "Burndown `ACTIVE_SPRINT_NAME`"
    x-axis [04-13, 04-14, 04-15, 04-16]
    y-axis "Remaining Issues" 0 --> {total_at_start}
    line "Ideal"  [19, 17, 15, 13]
    line "Actual" [19, 16, 16, 15]

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

> **Note:** The burndown is based on issue close dates only, not on status transitions within the sprint. For a full flow chart with in-progress state tracking, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
````

---

### File 5 — `.github/prompts/stakeholder-update.prompt.md`

````markdown
---
name: Stakeholder Update — {team_name_or_repo}
description: Management-ready sprint update for {team_name_or_repo}. Pre-configured — no setup needed.
agent: agent
---

> **Configuration** — generated by `setup.agent.md`
> Repository: `{owner/repo}`
> Deadline risk window: **{deadline_risk_days} days** · Highlight threshold: **{high_discussion_threshold} comments**
> Sprint tracking: {sprint_tracking_method_summary}

You are helping a software team produce a concise, management-ready sprint status update.

## Before You Begin

Fetch all open milestones in `{owner/repo}` and present them as a numbered list:

```
Open milestones:
  1. <title> (due <due_on date or "no due date">)
  2. ...
Which milestone should this update cover? (Enter number, default: 1 — the one with the earliest due date)
```

Wait for the user's answer. Store the selected milestone title as `ACTIVE_MILESTONE`.

If no milestones exist, ask: *"No open milestones found. Please provide a milestone name or describe the scope."*

## Step 1 — Load data

**Data access:** {access_method_instructions}

Fetch the milestone `ACTIVE_MILESTONE` in `{owner/repo}` using `milestone:"ACTIVE_MILESTONE"`.

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

## 📊 Sprint Update: `ACTIVE_MILESTONE`{team_name_suffix}

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

> **Note:** Probability-based completion forecasts and risk scores derived from historical flow patterns require historical flow data that GitHub Copilot cannot access. For automated risk scoring and forecasting, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
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
| Throughput Forecast | `.github/prompts/throughput-forecast.prompt.md` |
| Burndown Chart | `.github/prompts/burndown-chart.prompt.md` |
| Stakeholder Update | `.github/prompts/stakeholder-update.prompt.md` |

These prompts know your project setup. Use them directly — no re-explaining needed.

> **Re-run this setup** whenever your sprint tracking method changes or a new team member joins (e.g. to generate prompts for a different repository).

---

> **Note:** These prompts use GitHub Copilot's ability to read live issue data. They do not access historical status-change events. For automated, historically accurate flow metrics, visit [flow2c.com](https://flow2c.com?utm_source=copilot-agile-metrics).
