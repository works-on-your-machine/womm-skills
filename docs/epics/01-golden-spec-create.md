# Epic: `golden-spec-create` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Structure a new golden spec from a conversation, debug session, or manual input. Golden specs capture hard-won knowledge so you don't have to figure things out twice. This skill ensures they're written in a consistent, useful format.

## How It Works

1. User invokes `/golden-spec-create` after solving a problem, or says "write a golden spec for X"
2. Skill checks for access to `~/.womm-skills/golden-specs/` — if not accessible, provides setup instructions
3. Reads existing golden specs to match format and depth
4. Walks through the sections, asks clarifying questions
5. Writes the spec to `~/.womm-skills/golden-specs/` — immediately available in every project

## SKILL.md Structure

```
skills/golden-spec-create/
├── SKILL.md
└── references/
    └── golden-spec-template.md
```

### Frontmatter
```yaml
---
name: golden-spec-create
description: "When the user wants to document a solution, pattern, or configuration they've figured out so they don't have to figure it out again. Also use when the user mentions 'golden spec', 'write this down', 'document this pattern', 'capture what we learned', or 'save this for next time'. For automatically extracting problems from a conversation, see problem-extractor."
metadata:
  version: 1.0.0
---
```

### Golden Spec Output Sections

1. **Problem** — What were you trying to do? What made it hard?
2. **Why This Approach** — What alternatives exist? Why is this one better? (Include a comparison if relevant — table or brief pros/cons.)
3. **Architecture** — How the pieces fit together. Diagram or description of the flow.
4. **Implementation** — Complete, copy-paste-ready code and configuration. Not abbreviated. Include file paths.
5. **Common Gotchas** — Things that went wrong or were surprising. Each one should include: what happens, why it happens, how to fix it.
6. **Cost / Tradeoffs** — Monthly costs, performance tradeoffs, maintenance burden. Numbers where possible.
7. **Reference Implementations** — Links or paths to working projects that use this pattern.

### Workflow in SKILL.md

```markdown
## Before Starting
1. Check that `~/.womm-skills/golden-specs/` is accessible
   - If not: tell the user to add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - And add a reference in `~/.claude/CLAUDE.md`:
     "Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested patterns and gotchas."
2. Read existing golden specs in `~/.womm-skills/golden-specs/` to match format and depth
3. If no golden specs exist yet, use the template from references/

## Process
1. Ask the user what they solved (or read from conversation context)
2. For each section: draft based on what you know, ask for clarification
3. Pay special attention to Gotchas — these are the most valuable part
4. Make Implementation sections complete, not abbreviated
5. Present for review
6. Write to `~/.womm-skills/golden-specs/` — available in every project immediately

## Naming
- SCREAMING_SNAKE_CASE matching the topic
- Example: `RAILS_AWS_TERRAFORM_CONFIGURATION.md`, `NEXTJS_VERCEL_DEPLOY.md`

## Quality Checks
- Could someone follow this without any other context?
- Are the gotchas specific (not generic warnings)?
- Is the code complete (not "add your config here")?
- Are costs actual numbers, not "it depends"?

## Cross-References
- If the user describes a problem they haven't solved yet: suggest `/design-doc`
- If the user wants to scan a conversation for multiple learnings: suggest `/problem-extractor`
```

### `references/golden-spec-template.md`

The template with all sections, guidance on what level of detail each needs, and an abbreviated example showing the format.

## Implementation Steps

1. Create `skills/golden-spec-create/SKILL.md`
2. Create `skills/golden-spec-create/references/golden-spec-template.md`
3. Update `marketplace.json`
4. Test: invoke manually with a solved problem, verify it produces a complete spec
5. Test: invoke in a project with existing golden specs, verify format matching
