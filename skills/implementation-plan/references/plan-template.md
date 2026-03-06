# Implementation Plan Template

Use this format for technical implementation plans.

---

```markdown
# Implementation Plan: [Epic or Feature Name]

## Overview

One paragraph: what we're building and the high-level technical approach.

## Technical Decisions

Key choices and rationale. Each decision should explain what we're doing and why — especially when there are reasonable alternatives.

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [Area] | [What we chose] | [Why, including what we considered and rejected] |

## Story-by-Story Plan

### Story X.1: [Story Name]

**What to build:**
Brief description of the technical work.

**Files to create/modify:**
- `path/to/new_file.rb` — [What this file does]
- `path/to/existing_file.rb` — [What changes and why]

**Approach:**
How to implement this. Reference existing patterns in the codebase.

**Test approach:**
- [What to test and at what level]
- [Specific test cases if non-obvious]

**Risk/complexity:** [Low | Medium | High] — [Brief explanation if medium or high]

**Dependencies:** [Other stories that must be done first, or "None"]

---

### Story X.2: [Story Name]

[Same format as above]

---

## Shared Patterns

Common approaches that apply across multiple stories:

- **Error handling:** [How errors are handled in this feature]
- **Naming conventions:** [Any specific naming relevant to this work]
- **[Other patterns]:** [Description]

## Open Questions

Technical unknowns that might change the plan during implementation:

- **[Question]** — [Why it matters, what we'd do if we had to decide now]
```

---

## Guidelines

**Right level of detail:**
- **Too vague:** "Implement the webhook handler" — This is the story description, not a plan.
- **Too detailed:** Full pseudocode for every method — This belongs in the code itself.
- **Right:** Specific files, specific patterns, specific decisions — enough that the `/implement` skill knows exactly what to build without guessing.

**Reading the codebase for patterns:**
- Look at 2-3 similar features to understand conventions
- Note the test style: do they use factories? Fixtures? Mocks?
- Check for shared base classes, concerns, or modules to extend
- Look at how configuration and secrets are handled

**When the plan is ready:**
- Every story has a clear technical approach
- Key decisions are documented with rationale
- Someone (or an AI agent) could start implementing without asking "but how?"
- Open questions don't block the first few stories
