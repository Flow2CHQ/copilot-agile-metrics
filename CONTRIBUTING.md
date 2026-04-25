# Contributing to copilot-agile-metrics

Thank you for your interest in contributing! Contributions of all kinds are welcome — new analysis prompts, improvements to existing ones, documentation fixes, and bug reports.

## Before you start

By participating in this project you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).
By submitting a contribution you agree that it will be published under the [CC BY 4.0 licence](LICENSE) that covers this project.

---

## How to report a bug or request a feature

Use the issue templates:

- **[Bug report](https://github.com/Flow2CHQ/copilot-agile-metrics/issues/new?template=bug_report.md)** — a prompt produces unexpected output or does not work as described
- **[Feature request](https://github.com/Flow2CHQ/copilot-agile-metrics/issues/new?template=feature_request.md)** — a new prompt idea, a new analysis angle, or an improvement to an existing prompt

---

## How to submit a prompt or improvement

1. **Fork** this repository and create a feature branch from `main`.
2. Add your new prompt file to the `prompts/` directory.
3. Follow the **prompt structure** below.
4. Add a row for your prompt to the table in `README.md`.
5. Open a **pull request** with a short description of:
   - What the prompt does and what output it produces
   - Which GitHub data it uses (Issues REST API, Projects GraphQL API, etc.)
   - Any known limitations or caveats

---

## Prompt structure

Every prompt file must follow these conventions:

### 1. YAML frontmatter

```yaml
---
name: Short Descriptive Name
description: One sentence — what this prompt does and what it produces.
argument-hint: "Hint shown in the VS Code prompt picker (e.g. 'Sprint name or milestone')"
agent: agent
---
```

### 2. Before You Begin section

Every prompt must ask for all required parameters interactively **in a single message** before fetching any data. This makes prompts work out-of-the-box without prior configuration.

```markdown
## Before You Begin — Tell Copilot About Your Setup

Before fetching any data, ask the user the following questions **in a single message**.
Wait for all answers before proceeding.
```

### 3. Be honest about limitations

Prompts must be transparent about what GitHub Copilot can and cannot compute.  
See the [Limitations section in the README](README.md#limitations) for examples.  
Add a `> **Note:**` callout at the end if there are relevant data gaps.

### 4. Language and tone

- Write in **English**.
- Address the Scrum Master or team lead directly.
- Keep output structured (tables, headings) so it is scannable at a glance.
- End each prompt with concrete, actionable recommendations — not just raw data.

---

## Questions and ideas

Open a [Discussion](https://github.com/Flow2CHQ/copilot-agile-metrics/discussions) for general questions, ideas, or to share how you are using these prompts with your team.
