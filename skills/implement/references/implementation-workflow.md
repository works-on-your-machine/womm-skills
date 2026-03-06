# Implementation Workflow Reference

Detailed reference for the `/implement` skill's story-by-story execution workflow.

## Commit Message Format

Reference the story number and describe what was done:

```
Implement story 1.1: Set up database schema for subscriptions
Implement story 1.2: Add Subscription model with validations
Implement story 2.1: Create webhook endpoint for Stripe events
```

Keep messages short (under 72 characters for the first line). If more detail is needed, add a blank line and a body.

## Updating the Tracker

### Checkbox Format
```markdown
- [x] Story 1.1: Set up database schema
- [ ] Story 1.2: Add Subscription model
```
Change `[ ]` to `[x]` when a story is done.

### Status Field Format
```markdown
### Story 1.1: Set up database schema
**Status:** Done
```
Change `Ready` or `In Progress` to `Done`.

### Epic Status in epics.md
When all stories in an epic are complete, update the epic's row in `epics.md`:
```markdown
| User Auth | Done | Nothing | [docs/epics/01-user-auth.md] |
```

## When Tests Fail

1. **Read the error** — Understand what failed and why
2. **Check if it's your change** — Did you break something, or was it already broken?
3. **Fix in the same story** — Don't create a new story for a bug you just introduced
4. **Re-run the full suite** — Make sure the fix didn't break something else
5. **If stuck after 2-3 attempts** — Ask the user. Don't loop endlessly.

## When to Stop and Ask

Stop and ask the user when:
- The story's acceptance criteria are ambiguous
- You've found a bug in an earlier story that affects this one
- The implementation plan contradicts what you see in the codebase
- A dependency is missing and you're not sure whether to build it or skip ahead
- Tests keep failing and you've tried 2-3 approaches

Don't stop and ask when:
- The path forward is clear from the story description
- You need to make a minor technical choice (variable name, exact file location) that doesn't change the outcome
- The implementation plan covers this story in detail

## Story-to-Story Continuity

Between stories:
- Each commit should leave the codebase in a working state
- The next story should be implementable without knowledge of the previous story's details (the code speaks for itself)
- If a story reveals that the next story's plan needs updating, update the plan before starting
