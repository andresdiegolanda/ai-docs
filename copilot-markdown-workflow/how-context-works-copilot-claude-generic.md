# How Context Works in GitHub Copilot with Claude

> A visual guide to understanding what Claude sees when you use GitHub Copilot

---

## The Big Picture

When you type a prompt in Copilot Chat or accept an inline suggestion, you're not talking directly to Claude. You're talking to the **Copilot extension**, which assembles context from multiple sources, packages it into a single request, and sends it to Claude. Claude never sees your repo directly — it only sees what the extension decides to include.

Think of it like a briefing package. The Copilot extension is the assistant who prepares the briefing. Claude is the executive who reads it and makes decisions. The quality of Claude's output depends entirely on what's in that briefing.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false
skinparam roundcorner 10
skinparam defaultFontSize 13

title How a Copilot Request Reaches Claude

actor Developer as dev
rectangle "VS Code" {
  rectangle "Copilot Extension" as ext #LightBlue
  rectangle "Your Editor\n(open files, cursor position)" as editor
}

cloud "GitHub Copilot Service" as service
cloud "Claude (LLM)" as claude #LightGreen

dev -> editor : types prompt or code
editor -> ext : current file, cursor, selection
ext -> ext : assembles context\nfrom multiple sources
ext -> service : sends assembled context\n+ your prompt
service -> claude : forwards request
claude -> service : returns response
service -> ext : delivers response
ext -> editor : shows suggestion\nor chat response
editor -> dev : sees result

@enduml
```

**Key insight:** Claude is powerful, but it can only work with what it receives. If the Copilot extension doesn't include a piece of context, Claude doesn't know it exists. This is why understanding the context assembly process matters.

---

## The Five Layers of Context

The Copilot extension assembles context from five sources, each with different loading rules. They stack on top of each other — Claude receives all applicable layers merged into one context window.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false
skinparam roundcorner 10
skinparam defaultFontSize 12

title The Five Context Layers (Bottom = Always Loaded, Top = Manual)

rectangle "LAYER 5: Developer-Attached Context\n#file: references, chat history, selected code" as L5 #FFA07A
rectangle "LAYER 4: Prompt Templates\n.github/prompts/*.prompt.md\n(loaded when developer invokes with /command)" as L4 #FFD700
rectangle "LAYER 3: Skills\n.github/skills/*.skill.md\n(loaded when referenced by instructions or prompts)" as L3 #FFDAB9
rectangle "LAYER 2: File-Pattern Instructions\n.github/instructions/*.instructions.md\n(loaded when applyTo pattern matches open file)" as L2 #87CEEB
rectangle "LAYER 1: Base Instructions\n.github/copilot-instructions.md\n(ALWAYS loaded, every session)" as L1 #90EE90

L1 -[hidden]up- L2
L2 -[hidden]up- L3
L3 -[hidden]up- L4
L4 -[hidden]up- L5

note right of L1
  Loaded: **Automatically, always**
  Contains: Repo structure, build workflow,
  testing rules, PR conventions
end note

note right of L2
  Loaded: **Automatically, by file pattern**
  Trigger: You open a file matching a pattern
  Contains: Language/framework-specific standards
end note

note right of L3
  Loaded: **On demand, when referenced**
  Contains: Technical patterns, code snippets,
  best practices for specific technologies
end note

note right of L4
  Loaded: **Manually, by developer command**
  Contains: Multi-step workflow templates
  with parameters and validation
end note

note right of L5
  Loaded: **Manually, by developer action**
  Contains: Whatever you attach — context
  documents, other files, code selections
end note

@enduml
```

---

## Layer by Layer: What Each One Does

### Layer 1: Base Instructions (Always On)

**File:** `.github/copilot-instructions.md`

**When loaded:** Every single Copilot interaction in the repo. No exceptions. No action required from the developer.

**What it tells Claude:**
- How the repository is structured (monorepo layout, project locations, shared libraries)
- How to build and test (which tools, which commands, which configurations)
- What conventions to follow (linting rules, commit hooks, PR standards)
- What not to do (don't break shared configs, don't edit production code without permission)

**Analogy:** This is the employee handbook. Every new hire reads it on day one. It covers the rules that apply to everything, everywhere, always.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title Layer 1: Base Instructions — Always Active

rectangle "Developer opens\nany file in the repo" as trigger
rectangle "Copilot Extension\nloads copilot-instructions.md" as load #90EE90
rectangle "Claude receives:\n• Repo structure\n• Build commands\n• Testing rules\n• PR conventions\n• What NOT to do" as result

trigger -> load : automatic
load -> result : always included

note bottom of result
  This happens whether you're
  editing a .ts file, a .py file,
  a test file, or anything else.
  
  The developer does nothing.
  It just works.
end note

@enduml
```

---

### Layer 2: File-Pattern Instructions (Automatic, Conditional)

**File:** `.github/instructions/<framework>.instructions.md`

**When loaded:** Automatically, when the developer opens or edits a file matching the `applyTo` pattern.

**Example patterns:**
- `**/*.ts, **/*.html, **/*.scss` → loads Angular/TypeScript instructions
- `**/*.py` → loads Python instructions
- `**/*.java` → loads Java/Spring instructions
- `**/*.spec.ts, **/*.test.ts` → loads testing-specific instructions

**What it tells Claude (in addition to Layer 1):**
- Framework-specific coding standards
- Preferred patterns and conventions
- Architecture rules (component design, state management, dependency injection)
- Testing patterns specific to the framework

**Analogy:** This is the department-specific training manual. A frontend developer gets frontend rules. A backend developer gets backend rules. Each set loads automatically based on what file you're working on.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title Layer 2: File-Pattern Instructions — Conditional Loading

rectangle "Developer opens\nuser.component.ts" as trigger1
rectangle "Developer opens\nREADME.md" as trigger2

rectangle "Pattern match:\n**/*.ts ✅" as match1 #87CEEB
rectangle "Pattern match:\n**/*.ts ❌" as match2 #FFB6C1

rectangle "Claude receives Layer 1\n+ framework instructions" as result1 #87CEEB
rectangle "Claude receives Layer 1 only\n(no framework instructions)" as result2

trigger1 -> match1
trigger2 -> match2
match1 -> result1 : framework instructions loaded
match2 -> result2 : no additional instructions

note bottom of result1
  Claude now knows both the repo rules
  AND the framework coding standards.
  
  This stacking is automatic.
end note

@enduml
```

---

### Layer 3: Skills (On Demand, Referenced)

**Files:** `.github/skills/*.skill.md`

**When loaded:** When an instruction file or prompt references them. Not loaded by default.

**What they contain:**
- Technical reference patterns (not tutorials)
- Code snippets showing the correct way to implement specific patterns
- Best practices condensed into actionable rules

**Examples:**
- `state-management.skill.md` — How to build stores
- `testing-patterns.skill.md` — Test setup conventions
- `api-integration.skill.md` — How to handle API calls, caching, error handling

**Analogy:** These are the reference manuals on the shelf. You don't read the entire plumbing manual every day — but when you're working on pipes, you pull it down and check the specs.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title Layer 3: Skills — Loaded When Referenced

rectangle "Developer edits\nuser.store.ts" as trigger

rectangle "Layer 1: copilot-instructions.md\n(always loaded)" as L1 #90EE90
rectangle "Layer 2: framework.instructions.md\n(pattern match: *.ts)" as L2 #87CEEB
rectangle "Layer 2 references:\nstate-management.skill.md" as ref
rectangle "Layer 3: skill loaded\nState management patterns" as L3 #FFDAB9

rectangle "Claude receives:\nRepo rules + Framework standards\n+ State management patterns" as result

trigger -> L1 : automatic
trigger -> L2 : pattern match
L2 -> ref : instruction contains reference
ref -> L3 : skill loaded into context
L1 -> result
L2 -> result
L3 -> result

note bottom of result
  Claude now knows:
  • How the repo works (Layer 1)
  • How to write framework code (Layer 2)
  • How to build stores properly (Layer 3)
  
  All three layers combined.
end note

@enduml
```

---

### Layer 4: Prompt Templates (Manual, Invoked by Command)

**Files:** `.github/prompts/*.prompt.md`

**When loaded:** Only when the developer explicitly invokes them with a slash command.

**Examples:**
- `/migrate-feature featureName="user-dashboard"` → loads the feature migration workflow
- `/review reviewType="component"` → loads the component review checklist
- `/create-prompt prompt="deploy-feature"` → loads the prompt creation workflow

**What they contain:**
- Multi-step workflow definitions
- Configuration variables (parameters the developer provides)
- Imperative language ("YOU MUST", "YOU NEVER")
- Validation checkpoints

**Analogy:** These are the standard operating procedures. You don't read the "how to do a building inspection" checklist every day — you pull it out when you're doing a building inspection.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title Layer 4: Prompt Templates — Explicit Invocation

actor Developer as dev

rectangle "Developer types:\n/review reviewType=\"component\"" as cmd
rectangle "Copilot loads:\nreview.prompt.md" as load #FFD700
rectangle "Prompt references:\nframework.instructions.md\n+ component checklist" as refs

rectangle "Claude receives:\nLayer 1 + Layer 2 + Layer 3\n+ Review workflow with\nrequirements, suggestions,\nand structured output format" as result

dev -> cmd : explicit command
cmd -> load : prompt loaded
load -> refs : dependencies resolved
refs -> result : full context assembled

note bottom of result
  Prompts are the most structured
  layer. They define exact steps,
  expected outputs, and quality gates.
  
  Claude follows them like a recipe.
end note

@enduml
```

---

### Layer 5: Developer-Attached Context (Fully Manual)

**This is where your markdown context documents live.**

**When loaded:** Only when the developer explicitly attaches them using `#file:`, selects code, or references them in chat.

**Examples:**
- `#file:stories/PROJ-1234-user-enrollment.md` → attaches your context document
- Selecting code and pressing `Ctrl+I` → attaches the selected code
- Previous messages in the chat session → maintained as conversation history

**What context documents contain:**
- Problem statement for a specific story
- Assumptions and constraints
- Design decisions with alternatives considered
- Technical specification
- Architecture diagrams
- Acceptance criteria mapped to tests
- Open questions

**Analogy:** Layers 1-4 are the company knowledge. Layer 5 is the specific brief for today's task. A surgeon knows medicine (Layers 1-4), but they still need the patient's chart (Layer 5) before they operate.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title Layer 5: Developer-Attached Context — Your Context Documents

actor Developer as dev

rectangle "Developer types in Copilot Chat:\n#file:stories/PROJ-1234-user-enrollment.md\nGenerate the enrollment service based on\nthe technical specification" as prompt

rectangle "Copilot Extension assembles:" as ext {
  rectangle "Layer 1: Repo rules" as L1 #90EE90
  rectangle "Layer 2: Framework standards" as L2 #87CEEB
  rectangle "Layer 3: Referenced skills" as L3 #FFDAB9
  rectangle "Layer 5: Your context document\n• Problem statement\n• Constraints\n• Design decisions\n• Technical spec\n• Acceptance criteria" as L5 #FFA07A
}

rectangle "Claude generates code that:\n✅ Follows repo conventions (L1)\n✅ Uses framework patterns correctly (L2)\n✅ Implements patterns properly (L3)\n✅ Solves YOUR specific problem (L5)\n✅ Respects YOUR constraints (L5)\n✅ Reflects YOUR design decisions (L5)" as result

dev -> prompt
prompt -> ext : context assembled
ext -> result : all layers sent to Claude

note bottom of result
  Without Layer 5, Claude generates
  generic code that follows the repo
  patterns but doesn't know YOUR
  requirements.
  
  Layer 5 is the difference between
  "correct code" and "the right code."
end note

@enduml
```

---

## The Complete Flow: All Layers Together

Here's what happens from the moment you type a prompt to the moment Claude responds:

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false
skinparam sequenceMessageAlign center

title Complete Context Flow: Developer Prompt → Claude Response

actor Developer as dev
participant "VS Code\nEditor" as vscode
participant "Copilot\nExtension" as ext
database ".github/\nRepository Files" as repo
cloud "Claude\n(LLM)" as claude

== Developer Action ==
dev -> vscode : Opens source file
dev -> vscode : Types in Copilot Chat:\n#file:stories/PROJ-1234-enrollment.md\n"Generate enrollment service"

== Context Assembly (automatic) ==
vscode -> ext : Current file + chat prompt\n+ attached file

ext -> repo : Read copilot-instructions.md
repo --> ext : Layer 1: Repo rules ✅

ext -> repo : File pattern matches?\nRead matching instructions
repo --> ext : Layer 2: Framework standards ✅

ext -> repo : Instructions reference skills?\nRead referenced skill files
repo --> ext : Layer 3: Technical patterns ✅

ext -> ext : Attach developer's\ncontext document
note right : Layer 5: Story context ✅

== Request to Claude ==
ext -> claude : Sends single request containing:\n—————————————\n1. System context (Layers 1+2+3)\n2. User prompt\n3. Attached context doc (Layer 5)\n4. Current file content\n5. Chat history

== Claude Processing ==
claude -> claude : Reads all context\nUnderstands:\n• Repo structure\n• Framework patterns\n• Technical conventions\n• THIS story's requirements\n• THIS story's constraints\n• THIS story's design decisions

== Response ==
claude --> ext : Generated code\nthat respects ALL layers
ext --> vscode : Displays response
vscode --> dev : Sees contextually\naware suggestion

@enduml
```

---

## Why Layer 5 Changes Everything

Without your context document, Claude works with Layers 1-3: it knows how the repo works, how to write code in your framework, and how to use specific libraries. It generates **correct but generic** code.

With your context document, Claude also knows the specific problem, the constraints, the design decisions, and the acceptance criteria. It generates **correct AND specific** code.

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title The Same Prompt, With and Without Layer 5

rectangle "Prompt: Generate user enrollment service" as prompt

rectangle "WITHOUT Context Document\n(Layers 1-3 only)" as without {
  rectangle "Claude generates:\n• Generic enrollment service\n• Standard HTTP calls\n• Default error handling\n• No awareness of:\n  - Analytics tracking requirements\n  - Authentication method decisions\n  - Security token constraints\n  - Specific error scenarios\n  - Acceptance criteria" as bad
}

rectangle "WITH Context Document\n(Layers 1-3 + Layer 5)" as with {
  rectangle "Claude generates:\n• Enrollment service matching spec\n• Analytics tracking integration\n• Auth method per design decision\n• Specific error handling scenarios\n• Timeout and retry logic per constraints\n• Tests mapped to AC1, AC2, AC3" as good #90EE90
}

prompt --> without
prompt --> with

note bottom of bad
  Works. Compiles. Follows patterns.
  But you'll spend an hour
  adapting it to your actual needs.
end note

note bottom of good
  Works. Compiles. Follows patterns.
  AND reflects your specific
  requirements from the start.
end note

@enduml
```

---

## What Claude Does NOT See

Understanding what's excluded is as important as understanding what's included.

| Source | Does Claude See It? | Why / Why Not |
|--------|-------------------|---------------|
| Your entire repo | **No** | Only files explicitly included by the extension or attached by you |
| Files in other branches | **No** | Copilot operates on the current workspace state |
| Issue tracker stories | **No** | Unless you copy the content into your context document |
| Wiki / documentation platform | **No** | Unless you copy the content into your context document |
| Team chat conversations | **No** | Unless you copy relevant decisions into your context document |
| Previous Copilot sessions | **No** | Each session starts fresh — no memory between sessions |
| Your teammate's context documents | **No** | Unless they committed them and you attach them |
| Git history / blame | **No** | Claude doesn't see who wrote what or when |
| Runtime behavior / logs | **No** | Claude works with static code, not execution data |
| Your `.env` files | **No** (and shouldn't) | These are typically gitignored for security |

**This is why context documents matter.** Everything Claude doesn't see automatically, you need to provide explicitly. The context document is your mechanism for bridging that gap — it captures the knowledge that lives in your issue tracker, your wiki, your team chat, and your head, and puts it where Claude can actually use it.

---

## Practical Implications

### 1. Order matters inside documents

Claude processes context sequentially. In your context document, put the most important information first:

```
✅ Good order:
  Problem Statement → Constraints → Technical Spec → Decisions → Open Questions

❌ Bad order:
  Open Questions → Historical Context → Nice-to-Haves → Oh by the way, the actual requirements
```

### 2. Explicit constraints beat implicit assumptions

Claude follows what you write, not what you assume. If something is off the table, say so.

```
✅ "MUST NOT call external APIs during the enrollment flow — all data comes from the SDK"
❌ (assuming Claude will figure out the constraint from the code)
```

### 3. Layers can conflict — higher layers win

If your context document says "use a different pattern than what's in the framework instructions," Claude will follow your context document. Layer 5 (your explicit instructions) overrides Layer 2 (general patterns) when they conflict. Be intentional about this.

### 4. Context window has limits

Claude has a large but finite context window. If you attach a 50-page context document plus 10 source files plus a full chat history, something gets truncated. Keep context documents focused on one story. Reference other documents by name rather than including their full contents.

### 5. Each session starts from zero

Claude has no memory between Copilot sessions. When you start a new chat, all five layers are assembled fresh. Your context document is the only thing that carries your story's knowledge forward. Without it, you re-explain everything from scratch.

---

## Summary

```plantuml
@startuml
skinparam backgroundColor white
skinparam shadowing false

title What Claude Knows = What The Extension Sends

rectangle "Repo-Level Knowledge\n(Automatic)" as auto #90EE90 {
  rectangle "Layer 1: copilot-instructions.md\nHow the repo works" as L1
  rectangle "Layer 2: framework.instructions.md\nHow to write code in your stack" as L2
  rectangle "Layer 3: *.skill.md\nSpecific technical patterns" as L3
}

rectangle "Task-Level Knowledge\n(Manual)" as manual #FFA07A {
  rectangle "Layer 4: *.prompt.md\nReusable workflows" as L4
  rectangle "Layer 5: Your context document\nThis specific story's requirements,\nconstraints, decisions, and criteria" as L5
}

rectangle "Claude's Output Quality" as output #LightYellow

auto --> output : "Correct code that\nfollows patterns"
manual --> output : "The RIGHT code that\nsolves YOUR problem"

note bottom of output
  Layers 1-3 make Claude competent in your repo.
  Layers 4-5 make Claude effective on your task.
  
  You need both.
end note

@enduml
```

**The bottom line:** GitHub Copilot's extension handles the plumbing — loading repo rules, matching file patterns, resolving skill references. Claude handles the reasoning — understanding your context, applying constraints, generating code. Your markdown context document is the bridge between what the extension knows automatically and what Claude needs to know specifically. Without it, Claude is a skilled developer who showed up to work without reading the brief. With it, Claude is a skilled developer who knows exactly what to build and why.
