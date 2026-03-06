# Epic: `implementation-plan` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Take an epic (or a phase from a design doc) and produce a detailed technical implementation plan. This sits between "what are we building" (epic-breakdown) and "go build it" (implement). The epic tells you the stories; the implementation plan tells you how you're going to approach them technically.

## The Difference from Epic Breakdown

- **Epic breakdown** produces: organizational structure, stories, dependencies, ordering
- **Implementation plan** produces: technical approach, patterns to use, files to create/modify, architecture decisions, risk areas

An epic story might say "Add webhook handling for Stripe events." The implementation plan says "Create `app/controllers/webhooks/stripe_controller.rb`, use `Stripe::Webhook.construct_event` for signature verification, handle 5 event types via a case statement, write to `Subscription` model idempotently using `stripe_subscription_id` as the unique key."

## SKILL.md Structure

```
skills/implementation-plan/
├── SKILL.md
└── references/
    └── plan-template.md
```

### Frontmatter
```yaml
---
name: implementation-plan
description: "When the user wants a detailed technical plan before implementing an epic, phase, or feature. Also use when the user mentions 'implementation plan', 'technical plan', 'how should we build this', 'plan the implementation', or 'what's the approach'. Can also use the interview flow to gather requirements if no epic exists yet. For breaking work into epics first, see epic-breakdown. For executing the plan, see implement."
metadata:
  version: 1.0.0
---
```

### Inputs (any of these)

- An epic file with stories
- A phase from a design doc
- A raw idea (triggers the interview flow, like design-doc)

### Workflow

1. **Check golden specs access** — Verify `~/.womm-skills/golden-specs/` is accessible. If not, provide setup instructions (add path to `~/.claude/settings.json`, add reference in `~/.claude/CLAUDE.md`).
2. **Read the project** — CLAUDE.md, existing code, conventions, test patterns, golden specs at `~/.womm-skills/golden-specs/`
3. **Read the input** — epic file, design doc phase, or user's description
4. **If no structured input exists** — Interview the user using AskUserQuestion (same deep interview as design-doc: technical implementation, concerns, tradeoffs, constraints)
5. **Analyze the codebase** — Look at existing patterns, similar implementations, test structure
6. **Produce the plan** — For each story (or logical chunk):
   - Files to create or modify
   - Patterns and approaches to use (informed by existing codebase)
   - Key technical decisions and why
   - Risk areas and edge cases
   - Test strategy (what to test, at what level)
7. **Present for review** — User approves or adjusts before implementation begins
8. **Write the plan** — Save alongside the epic or in docs/

### Output Sections

1. **Overview** — One paragraph: what we're building, high-level approach
2. **Technical Decisions** — Key choices and rationale (e.g., "use service objects, not concerns, because the existing codebase uses that pattern")
3. **Story-by-Story Plan** — For each story:
   - What to build
   - Files to create/modify (with brief description of changes)
   - Dependencies on other stories
   - Test approach
   - Risk/complexity notes
4. **Shared Patterns** — Common approaches that apply across stories (error handling strategy, naming conventions, etc.)
5. **Open Questions** — Technical unknowns that might change the plan during implementation

### Reference Files

**`references/plan-template.md`**
- Template with all sections
- Examples of good technical plans at the right level of detail
- Guidance on when a plan is too vague ("implement the feature") vs too detailed (pseudocode for every function)
- How to read an existing codebase for patterns to follow

## Implementation Steps

1. Create SKILL.md with the workflow above
2. Create reference file with template and examples
3. Update marketplace.json
4. Test: point at an epic, verify it produces a detailed technical plan
5. Test: invoke without an epic, verify the interview flow works
