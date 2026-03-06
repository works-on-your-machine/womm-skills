# Epic: `implement` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Take an epic and work through its stories one by one: plan each story, implement it, write tests, verify tests pass, commit, and update the epic tracker. This is the execution skill — it enforces the discipline of incremental, tested, committed progress.

## Why This Is a Skill

Without this, the common failure modes are:
- Implement everything at once, commit at the end (big messy commits, hard to revert)
- Skip tests or defer them ("I'll add tests later")
- Forget to update the epic tracker (status goes stale)
- Start the next story before the current one's tests pass

This skill encodes the discipline: one story at a time, tests pass before moving on, commit after each story, tracker stays current.

## SKILL.md Structure

```
skills/implement/
├── SKILL.md
└── references/
    └── implementation-workflow.md
```

### Frontmatter
```yaml
---
name: implement
description: "When the user wants to execute an epic by working through its stories one by one with tests and commits between each. Also use when the user mentions 'implement this epic', 'work through these stories', 'build this', 'start implementing', or 'execute the plan'. For creating the epic first, see epic-breakdown. For creating a technical plan first, see implementation-plan."
metadata:
  version: 1.0.0
---
```

### Workflow

For the whole epic:

1. **Check golden specs access** — Verify `~/.womm-skills/golden-specs/` is accessible. If not, provide setup instructions.
2. **Read the epic** — Understand all stories, their dependencies, their order. Reference golden specs at `~/.womm-skills/golden-specs/` for relevant patterns.
3. **Check for an implementation plan** — Look for a plan file alongside the epic. If one exists, use it as the technical guide. If not, that's fine — plan each story as you go.
4. **Identify the first unfinished story** — Check status markers (checkboxes, status fields) to find where to start

For each story:

5. **Read the story** — Understand inputs, outputs, acceptance criteria
6. **Verify/enhance the plan** — If an implementation plan exists, review the plan for this specific story. Check that it still makes sense given what's been built so far. If anything has changed (earlier stories revealed new information, patterns shifted), update the approach. If no plan exists, create a brief plan for this story.
7. **Present the plan to the user** — Brief summary: "For story 2.3, I'm going to create X, modify Y, and test with Z. Sound good?" Wait for approval.
8. **Implement** — Write the code
9. **Write tests** — Tests that verify the story's acceptance criteria
10. **Run tests** — All tests, not just the new ones. If tests fail:
    - Fix the failure
    - Re-run until all tests pass
    - Do NOT move on with failing tests
11. **Commit** — One commit per story. Message references the story (e.g., "Implement story 2.3: Add webhook signature verification")
12. **Update the epic tracker** — Mark the story as complete (check the box, update status)
13. **Move to the next story** — Go back to step 5

After all stories:

14. **Final check** — Run the full test suite one more time
15. **Update the epic status** — Mark the epic as done in the tracker
16. **Suggest next steps** — Recommend `/retrospective` to capture learnings, or point to the next epic

### Edge Cases

- **Story depends on something not yet built**: Flag it and ask the user whether to skip ahead or implement the dependency first
- **Tests don't exist for the project yet**: Set up the test framework as part of the first story (or as a prerequisite step)
- **Story is too big**: If a story takes more than ~30 minutes, suggest splitting it and update the epic
- **Implementation plan is wrong**: If the plan for a story doesn't match reality (earlier work changed things), update the plan and tell the user what changed

### Reference Files

**`references/implementation-workflow.md`**
- Detailed workflow with examples
- Commit message format (reference story numbers)
- How to update different tracker formats (checkbox lists, status fields, etc.)
- What to do when tests fail (debugging approach, when to ask for help)
- When to stop and ask the user vs. push through

## Implementation Steps

1. Create SKILL.md with the workflow above
2. Create reference file with detailed workflow and examples
3. Update marketplace.json
4. Test: run against a real epic with stories, verify it works through them correctly
5. Test: verify it handles the case where no implementation plan exists
6. Test: verify it correctly updates epic tracker status
