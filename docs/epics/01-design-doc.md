# Epic: `design-doc` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Take a raw idea, conversation, or rough notes and produce a structured design document. This is the first step in the build workflow — turning vague intent into something concrete enough to break into epics.

## How It Works

1. User invokes `/design-doc` with an idea, or says "write a design doc for X"
2. Skill reads the project directly — CLAUDE.md, existing docs, code files — to understand stack and conventions
3. Skill reads existing design docs in the project (if any) to match format
4. Produces a design doc and writes it to the docs directory

## SKILL.md Structure

```
skills/design-doc/
├── SKILL.md
└── references/
    └── design-doc-template.md
```

### Frontmatter
```yaml
---
name: design-doc
description: "When the user wants to create a structured design document from a raw idea, conversation, or rough notes. Also use when the user mentions 'design doc', 'spec out', 'architecture doc', 'write up the design', 'plan this out', 'brainstorm this', or 'help me think through this'. Includes a deep interview phase before writing. For breaking a design doc into epics, see epic-breakdown. For capturing solved problems, see golden-spec-create."
metadata:
  version: 1.0.0
---
```

### Sections in the Design Doc Output

1. **Problem** — What need does this solve? Why now?
2. **Solution** — High-level approach. What are we building?
3. **Architecture** — How it fits into the existing system. Key components, data flow, integrations.
4. **Constraints** — What we can't change. Budget, timeline, existing dependencies, technical limits.
5. **Implementation Approach** — Rough ordering. What gets built first and why.
6. **Open Questions** — Things we don't know yet that could change the approach.
7. **Related Golden Specs** — Link to any golden specs relevant to this work.

### Workflow in SKILL.md

```markdown
## Before Starting
1. Check that `~/.womm-skills/golden-specs/` is accessible. If not, provide setup instructions (add path to `~/.claude/settings.json`, add reference in `~/.claude/CLAUDE.md`).
2. Read the project to understand stack and conventions:
   - CLAUDE.md for project-specific instructions
   - Gemfile / package.json / go.mod for language and framework
   - Existing docs/ directory for format conventions
   - Golden specs at `~/.womm-skills/golden-specs/` for context on past decisions
3. Check for existing design docs in the project (look in docs/, docs/design/, etc.)
4. If existing docs found, match their format and level of detail

## Process
1. Ask the user to describe the idea (or read a conversation/notes they provide)
2. **Deep interview** — Use AskUserQuestion to interview the user in detail:
   - Start with problem space: what problem does this solve, who has it, why now?
   - Move to technical implementation: architecture, data model, integrations
   - Cover UI & UX if applicable: user flows, edge cases, error states
   - Explore concerns and tradeoffs: what could go wrong, what are we giving up?
   - Ask about constraints: budget, timeline, existing systems, team size
   - Questions should be specific and non-obvious — don't ask things you can infer from the project files
   - Keep interviewing until you've covered all major areas — don't stop after 3-4 questions
   - Synthesize answers as you go to ask better follow-up questions
3. Draft the design doc using the template (informed by the interview)
4. Present it for review
5. Iterate based on feedback
6. Write to the appropriate location (respect the project's existing docs structure)

## Output
- A markdown file in the project's docs directory
- Filename: descriptive, lowercase, hyphens (e.g., `user-authentication.md`)

## Cross-References
- After writing: suggest `/epic-breakdown` to break it into implementable work
- If the user describes a solved problem instead of a new idea: suggest `/golden-spec-create`
```

### `references/design-doc-template.md`

A reference template the skill can use. Contains:
- The 7 sections with brief guidance on what goes in each
- Examples of good vs. bad problem statements
- Guidance on appropriate level of detail (enough to build from, not a full implementation spec)
- Notes on when to stop and move to epic-breakdown vs. keep refining

## Implementation Steps

1. Create `skills/design-doc/SKILL.md`
2. Create `skills/design-doc/references/design-doc-template.md`
3. Update `marketplace.json`
4. Test: invoke with a rough idea, verify it reads project files and produces a complete doc
5. Test: invoke in a project that already has design docs, verify it matches existing format
