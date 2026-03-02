# Copilot Markdown Workflow

A methodology for controlling what GitHub Copilot reads — and getting better output as a result.

## Contents

| File | Description |
|------|-------------|
| `how-context-works-copilot-claude-generic.md` | Full guidelines document. Explains the five context layers, how to write effective instructions, and the story-level context document workflow. |
| `markdown-first-workflow-copilot.pptx` | Presentation deck (15 slides). No company-specific content — all customizable placeholders in `[BRACKETS]`. |
| `demo-sequence.md` | Speaker notes and timing for a 30-minute team demo. |

## Quick Summary

Copilot indexes your entire repo and searches it automatically on every prompt ([source](https://code.visualstudio.com/docs/copilot/reference/workspace-context)). You don't control what it reads. But you can control what it's **guaranteed** to read:

| Layer | File | Loading |
|-------|------|---------|
| 1 | `copilot-instructions.md` | Always loaded |
| 2 | `*.instructions.md` | Auto-loads by file type |
| 3 | `*.skill.md` | Loaded when referenced |
| 4 | `*.prompt.md` | Invoked with /command |
| 5 | Story context document | Manually provided |

Layers 1–4 are Copilot platform features. Layer 5 is the methodology contribution: a structured markdown document (problem statement, constraints, design decisions, technical spec, acceptance criteria) that you write before coding and provide to Copilot as context.

## Customizing for Your Team

The `.pptx` is designed to be adapted. Every company-specific element is a `[BRACKET PLACEHOLDER]`. To customize:

1. Replace `[PRESENTER NAME]`, `[DATE]`, `[TEAM]` on title and closing slides
2. Replace code blocks on slides 3–6 with excerpts from your actual repo files
3. Add your organization's AI strategy screenshot to the alignment slide
4. Take before/after Copilot Chat screenshots for the demo slides
5. Replace `[Output]` placeholders with your own shipped artifacts

## License

MIT