# Copilot Markdown Workflow

A methodology for controlling what GitHub Copilot reads — and getting better output as a result.

## Contents

| File | Description |
|------|-------------|
| `how-context-works-copilot-claude-generic.md` | Full guidelines document. Explains the five context layers, how to write effective instructions, and the story-level context document workflow. |
| `markdown-first-workflow-copilot.pptx` | Presentation deck (15 slides). No company-specific content — all customizable placeholders in `[BRACKETS]`. |
| `demo-sequence.md` | Speaker notes and timing for a 30-minute team demo. |
| `fidelity-specific-updates.md` | Guide for replacing placeholders with team-specific content. |

## Quick Summary

Copilot indexes your entire repo and searches it automatically on every prompt ([source](https://code.visualstudio.com/docs/copilot/reference/workspace-context)). You don't control what it reads. But you can control what it's **guaranteed** to read:

| Layer | File | Loading |
|-------|------|---------|
| 1 | `copilot-instructions.md` | Always loaded |
| 2 | `*.instructions.md` | Auto-loads by file type |
| 3 | `*.skill.md` | Copilot decides when to load |
| 4 | `*.prompt.md` | Invoked with /command |
| 5 | Story context document | Manually referenced by path |

Layers 1–4 are Copilot platform features. Layer 5 is the methodology contribution: a structured markdown document (problem statement, constraints, design decisions, technical spec, acceptance criteria) that you write before coding and provide to Copilot as context.

## Slide Overview

| # | Title | Content |
|---|-------|---------|
| 1 | Title | Presenter, date, team placeholders |
| 2 | Copilot Already Reads Your Code | Pyramid diagram placeholder |
| 3 | Layer 1: copilot-instructions.md | Always loaded, repo-wide rules |
| 4 | Layer 2: File-Pattern Instructions | Auto-loads by file type, scoping principle |
| 5 | Layer 3: Skills | Copilot decides when to load, open standard |
| 6 | Layer 4: Prompt Templates | Slash commands, three examples |
| 7 | Organizational Alignment | Your org's AI strategy screenshot placeholder |
| 8 | Layer 5: Your Story Context | Problem, constraints, design, spec, AC |
| 9 | Writing Good Instructions | Six output standards with rationale |
| 10 | Make Every Call Count | Bad→good prompt pairs, compound prompt example |
| 11 | With Context Document | Screenshot placeholder |
| 12 | Test Generation with AC Traceability | Screenshot placeholder |
| 13 | Real Outputs — Already Shipped | Three shipped artifact placeholders |
| 14 | Three Things to Try | Read, Write, Share |
| 15 | Closing | Feedback invitation |

## Customizing for Your Team

The `.pptx` is designed to be adapted. Every company-specific element is a `[BRACKET PLACEHOLDER]`. To customize:

1. Replace `[PRESENTER NAME]`, `[DATE]`, `[TEAM]` on title and closing slides
2. Replace code blocks on slides 3–6 with excerpts from your actual repo files
3. Add your organization's AI strategy screenshot to the alignment slide (slide 7)
4. Take Copilot Chat screenshots for the demo slides (slides 11–12)
5. Replace `[Output]` placeholders on slide 13 with your own shipped artifacts

See `fidelity-specific-updates.md` for detailed per-slide guidance.

## License

MIT
