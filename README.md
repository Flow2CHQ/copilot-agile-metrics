# copilot-agile-metrics

> Reusable GitHub Copilot prompt files for analysing GitHub Projects — works out-of-the-box in **VS Code Copilot Chat** and the **GitHub Web UI**.

[![Awesome Copilot](https://img.shields.io/badge/awesome--copilot-prompts-blue?style=for-the-badge)](https://github.com/github/awesome-copilot)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-green?style=for-the-badge)](LICENSE)

---

## Available Prompts

### 🚀 Start here

| Prompt | Install | Description |
|--------|---------|-------------|
| [setup.agent.md](prompts/setup.agent.md) | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=for-the-badge&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect?url=vscode%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2FFlow2CHQ%2Fcopilot-agile-metrics%2Fmain%2Fprompts%2Fsetup.agent.md) [![Download](https://img.shields.io/badge/Raw-download-grey?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/setup.agent.md) | **One-time setup wizard.** Asks about your project structure once, then generates personalised, pre-configured versions of all five analysis prompts in `prompts/configured/`. **VS Code only** — requires Agent mode to write files. |

### Analysis prompts (generic — ask setup questions each time)

| Prompt | Install | Description |
|--------|---------|-------------|
| [sprint-analysis.prompt.md](prompts/sprint-analysis.prompt.md) | [![Download](https://img.shields.io/badge/Raw-download-0098FF?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/sprint-analysis.prompt.md) | Analyse the current sprint — open issues, at-risk items, blockers, workload by assignee |
| [retro-input.prompt.md](prompts/retro-input.prompt.md) | [![Download](https://img.shields.io/badge/Raw-download-0098FF?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/retro-input.prompt.md) | Generate retrospective input from the last sprint — label patterns, discussion load, communication gaps |
| [monte-carlo-forecast.prompt.md](prompts/monte-carlo-forecast.prompt.md) | [![Download](https://img.shields.io/badge/Raw-download-0098FF?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/monte-carlo-forecast.prompt.md) | Throughput-based probabilistic forecast — when will the remaining issues be done? |
| [burndown-chart.prompt.md](prompts/burndown-chart.prompt.md) | [![Download](https://img.shields.io/badge/Raw-download-0098FF?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/burndown-chart.prompt.md) | Mermaid burndown chart for the current sprint based on issue close dates |
| [stakeholder-update.prompt.md](prompts/stakeholder-update.prompt.md) | [![Download](https://img.shields.io/badge/Raw-download-0098FF?style=for-the-badge&logo=github&logoColor=white)](https://raw.githubusercontent.com/Flow2CHQ/copilot-agile-metrics/main/prompts/stakeholder-update.prompt.md) | Management-ready sprint update: executive summary + risk table, generated in seconds |

> **Tip:** Run `setup.agent.md` once to generate pre-configured versions of all five prompts in `prompts/configured/`. Those versions skip all setup questions and go straight to the analysis.

---

## How to Use

### VS Code Copilot Chat

1. Install the [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) and [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) extensions.
2. Download individual `.prompt.md` / `.agent.md` files (use the **Raw download** or **Install** badges above) and place them in your project's `.github/prompts/` folder — VS Code picks them up automatically in the prompt picker.
   > **Note:** The prompts in *this* repository live in `prompts/` for browsing convenience. Copy them to `.github/prompts/` in *your own* project.
3. Open Copilot Chat (`⌃⌘I` / `Ctrl+Alt+I`) and switch to **Agent mode** (dropdown next to the send button). Agent mode gives Copilot automatic access to your GitHub repository data — no extra steps needed.
4. Select a prompt from the picker (or attach the file via the paperclip icon) and send it.
5. Copilot will ask you a few short questions first (e.g. how sprints are tracked, which sprint to analyse). Answer them in plain text.

> **Ask mode fallback:** If you prefer Ask mode, type `@github` at the start of the chat before selecting a prompt — this gives Copilot access to your repository data.
>
> Copy-paste template: `@github` *(then attach or invoke the prompt from the picker)*

### GitHub Web UI

1. Navigate to your repository on github.com.
2. Open any of the five analysis `.prompt.md` files.
3. Click **"Open in Copilot"** (the Copilot icon in the file toolbar) — available in repositories where GitHub Copilot is enabled.
4. Copilot will ask you a few short questions in the chat before running the analysis. Answer them in plain text.

> **Limitation:** The **setup wizard** (`setup.agent.md`) generates files and therefore only works in VS Code Agent mode — not in the GitHub Web UI. The five analysis prompts work in both environments.
>
> **Tip:** No file editing needed. All analysis prompts ask for parameters interactively.

---

## Limitations

GitHub Copilot reads the **current state** of issues and projects via the GitHub API. It does **not** have access to historical status-change events — meaning it cannot determine when an issue moved from *"In Progress"* to *"Done"* or how long it spent in a particular workflow column.

As a result, the following metrics **cannot be computed** by these prompts:

| Metric | Why it's unavailable |
|--------|----------------------|
| **Cycle Time** | Requires knowing when work *started* on an issue — the first transition into an active state (e.g. "In Progress"), which is not exposed via the API Copilot uses |
| **Time-in-State** | Requires a full history of every status transition with timestamps |
| **Multi-sprint trends** | Require aggregated historical data across many sprints |

What these prompts **can** do:

- Read all open/closed issues and their metadata (labels, assignees, milestones, comments, `closedAt` date)
- Calculate **Lead Time per issue** as `closedAt − createdAt` (creation to close — a useful proxy, though it includes time before work started)
- Estimate **throughput** (issues closed per week) from `closedAt` dates
- Generate simple forecasts, burndowns, and summaries based on throughput
- Identify structural risk signals (no assignee, blocker labels, proximity to deadline)

For automatically tracked Cycle Time, Time-in-State distributions, and real Monte Carlo simulations visit **[flow2c.com](https://flow2c.com)**.

---

## Contributing

Contributions are welcome! Please:

1. Fork the repository and create a feature branch.
2. Follow the existing prompt structure: YAML frontmatter (`name` + `description`) and a `## Before You Begin` section that asks for required parameters interactively.
3. Add your prompt to the table in this README.
4. Open a pull request with a short description of what the prompt does and what GitHub data it requires.

Please keep prompts language-agnostic (English), self-contained, and honest about what Copilot can and cannot compute.

---

*Built and maintained by the team behind [flow2c](https://flow2c.com) – automated flow metrics for GitHub Projects.*
