---
name: implement
description: "When the user wants to execute an epic by working through its stories one by one with tests and commits between each. Also use when the user mentions 'implement this epic', 'work through these stories', 'build this', 'start implementing', or 'execute the plan'. For creating the epic first, see epic-breakdown. For creating a technical plan first, see implementation-plan."
metadata:
  version: 1.0.0
---

# Implement

Execute an epic story by story: plan each story, implement it, write tests, verify tests pass, commit, and update the tracker. This skill enforces the discipline of incremental, tested, committed progress.

## Before Starting

1. **Check golden specs access.** Try to list files in `~/.womm-skills/golden-specs/`. If not accessible, tell the user:
   - Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - Add a reference in `~/.claude/CLAUDE.md` pointing to the golden specs directory

2. **Read the epic.** Understand all stories, their dependencies, and their order. Reference golden specs at `~/.womm-skills/golden-specs/` for relevant patterns.

3. **Check for an implementation plan.** Look for a plan file alongside the epic (e.g., `01-user-auth-plan.md` next to `01-user-auth.md`). If one exists, use it as the technical guide. If not, plan each story as you go.

4. **Find the first unfinished story.** Check status markers (checkboxes, status fields) to find where to start.

## For Each Story

### 1. Read the Story

Understand the inputs, outputs, and acceptance criteria.

### 2. Verify the Plan

If an implementation plan exists, review the plan for this specific story. Check that it still makes sense given what's been built so far. If anything has changed (earlier stories revealed new information, patterns shifted), update the approach and tell the user what changed.

If no plan exists, create a brief plan for this story.

### 3. Present the Plan

Brief summary to the user: "For story 2.3, I'm going to create X, modify Y, and test with Z. Sound good?" Wait for approval before proceeding.

### 4. Implement

Write the code.

### 5. Write Tests

Write tests that verify the story's acceptance criteria. Match the project's existing test style and framework.

### 6. Run Tests

Run ALL tests, not just the new ones. If tests fail:
- Fix the failure
- Re-run until all tests pass
- Do NOT move on with failing tests

### 7. Commit

One commit per story. Message references the story:
```
Implement story 2.3: Add webhook signature verification
```

### 8. Update the Tracker

Mark the story as complete in the epic file (check the checkbox, update status field).

### 9. Next Story

Go back to step 1 for the next unfinished story.

## After All Stories

1. **Final check** — Run the full test suite one more time
2. **Update epic status** — Mark the epic as done in `epics.md`
3. **Suggest next steps** — Point to the next epic, or suggest `/golden-spec-create` to capture learnings

## Edge Cases

- **Story depends on something not yet built:** Flag it and ask the user whether to skip ahead or implement the dependency first.
- **Tests don't exist for the project yet:** Set up the test framework as a prerequisite step before starting the first story.
- **Story is too big:** If a story takes more than ~30 minutes, suggest splitting it and update the epic.
- **Implementation plan is wrong:** If the plan for a story doesn't match reality, update the plan and tell the user what changed before proceeding.
