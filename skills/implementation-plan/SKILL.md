---
name: implementation-plan
description: "When the user wants a detailed technical plan before implementing an epic, phase, or feature. Also use when the user mentions 'implementation plan', 'technical plan', 'how should we build this', 'plan the implementation', or 'what's the approach'. Can also use the interview flow to gather requirements if no epic exists yet. For breaking work into epics first, see epic-breakdown. For executing the plan, see implement."
metadata:
  version: 1.0.0
---

# Implementation Plan

Produce a detailed technical plan for an epic, phase, or feature. This sits between "what are we building" (epic-breakdown) and "go build it" (implement). The epic tells you the stories; the implementation plan tells you how to approach them technically.

**The difference from epic-breakdown:**
- Epic breakdown produces: organizational structure, stories, dependencies, ordering
- Implementation plan produces: technical approach, patterns to use, files to create/modify, architecture decisions, risk areas

An epic story might say "Add webhook handling for Stripe events." The implementation plan says "Create `app/controllers/webhooks/stripe_controller.rb`, use `Stripe::Webhook.construct_event` for signature verification, handle 5 event types via a case statement, write to `Subscription` model idempotently using `stripe_subscription_id` as the unique key."

## Before Starting

1. **Check golden specs access.** Try to list files in `~/.womm-skills/golden-specs/`. If not accessible, tell the user:
   - Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - Add a reference in `~/.claude/CLAUDE.md` pointing to the golden specs directory

2. **Read the project:**
   - `CLAUDE.md` for project-specific instructions
   - Existing code to understand patterns, naming, structure
   - Test files to understand the testing approach
   - Golden specs at `~/.womm-skills/golden-specs/` for relevant prior solutions

3. **Read the input** — any of:
   - An epic file with stories
   - A phase from a design doc
   - A raw idea from the user

## Process

### 1. Understand the Work

If the input is an epic or design doc phase, read it fully.

If the input is a raw idea with no structured docs, interview the user first — same deep interview approach as `/design-doc`: problem space, technical implementation, concerns, tradeoffs, constraints.

### 2. Analyze the Codebase

Look at existing patterns and similar implementations:
- How are similar features structured?
- What conventions does the codebase follow? (naming, file organization, error handling)
- What test patterns are used?
- Are there shared utilities or base classes to build on?

### 3. Produce the Plan

Write the plan using the template in `references/plan-template.md`. For each story (or logical chunk):

- **Files to create or modify** — specific paths with brief description of changes
- **Patterns and approaches** — informed by existing codebase, not abstract best practices
- **Key technical decisions** — choices and rationale
- **Risk areas and edge cases** — what could go wrong
- **Test strategy** — what to test, at what level (unit, integration, system)

### 4. Review

Present the plan to the user. They should approve or adjust before implementation begins. Pay attention to:
- Does the plan match their mental model of how the codebase works?
- Are there patterns they prefer that the plan doesn't follow?
- Are there risks they're aware of that the plan doesn't address?

### 5. Write

Save the plan alongside the epic or in the project's docs directory. Name it to match the epic (e.g., `01-user-auth-plan.md` for epic `01-user-auth.md`).

## Cross-References

- Before this: suggest `/design-doc` if there's no design doc, or `/epic-breakdown` if there are no epics
- After this: suggest `/implement` to execute the plan story by story
