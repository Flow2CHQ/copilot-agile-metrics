---
description: "Use when editing any file in prompts/ — especially burndown-chart.prompt.md, throughput-forecast.prompt.md, sprint-analysis.prompt.md, retro-input.prompt.md, stakeholder-update.prompt.md, or setup.agent.md. Ensures that changes to individual prompts are always reflected in the setup agent template, and vice versa."
applyTo: "prompts/**"
---

# Prompt ↔ Setup Agent Sync Rule

`prompts/setup.agent.md` enthält in Step 3 eingebettete **Templates** für alle fünf generierten Prompts. Diese Templates müssen immer mit den Originalprompts in `prompts/` übereinstimmen.

## Pflicht bei jeder Änderung

**Wenn `prompts/<name>.prompt.md` geändert wird:**
Prüfe, ob das entsprechende Template in `prompts/setup.agent.md` (unter `### File N — .github/prompts/<name>.prompt.md`) dieselbe Änderung benötigt. Wenn ja, passe das Template an.

**Wenn `prompts/setup.agent.md` geändert wird:**
Prüfe, ob die Änderung eine Logik betrifft, die auch im Original-Prompt `prompts/<name>.prompt.md` vorhanden ist. Wenn ja, passe den Original-Prompt an.

## Mapping: Original → Template in setup.agent.md

| Original-Datei | Template-Abschnitt in setup.agent.md |
|---|---|
| `prompts/sprint-analysis.prompt.md` | `### File 1 — .github/prompts/sprint-analysis.prompt.md` |
| `prompts/retro-input.prompt.md` | `### File 2 — .github/prompts/retro-input.prompt.md` |
| `prompts/throughput-forecast.prompt.md` | `### File 3 — .github/prompts/throughput-forecast.prompt.md` |
| `prompts/burndown-chart.prompt.md` | `### File 4 — .github/prompts/burndown-chart.prompt.md` |
| `prompts/stakeholder-update.prompt.md` | `### File 5 — .github/prompts/stakeholder-update.prompt.md` |

## Was synchronisiert werden muss

- Schritte und Schritt-Logik (z. B. "Fetch server-side", "three sprint boundary cases")
- Fallback-Regeln und Fehlerfälle (z. B. "ask user if dates cannot be read")
- "Before You Begin"-Abschnitte
- Hinweise und Limitations
- Frontmatter `name` und `description`

## Was NICHT synchronisiert werden muss

Templates in `setup.agent.md` verwenden `{placeholders}` wie `{owner/repo}`, `{current_sprint}`, `{resolved_filter_strategy}`. Die Original-Prompts formulieren dieselbe Logik generisch. Die Placeholder-Syntax selbst muss nicht identisch sein.
