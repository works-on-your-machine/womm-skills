---
name: epic-breakdown
description: "When the user wants to break a design document into implementable epics with dependencies and status tracking. Also use when the user mentions 'break this down', 'create epics', 'plan the work', 'what do we build first', 'slice this', 'make stories', or 'context-window sized'. For creating the design doc first, see design-doc."
metadata:
  version: 1.0.0
---

# Epic Breakdown

Take a design doc and produce an `epics.md` tracking hub plus individual epic files with stories, dependencies, and risk ordering.

## Before Starting

1. **Check golden specs access.** Try to list files in `~/.womm-skills/golden-specs/`. If not accessible, tell the user:
   - Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - Add a reference in `~/.claude/CLAUDE.md` pointing to the golden specs directory

2. **Read the project** for structure conventions:
   - `CLAUDE.md` for project-specific instructions
   - Existing `docs/` directory and any `epics.md` already present
   - Golden specs at `~/.womm-skills/golden-specs/` for context

3. **Find the design doc.** Ask the user to point to it, or look in `docs/`, `docs/design/`, etc.

4. **Check for existing `epics.md`.** If present, we're adding to it, not replacing it.

## Process

### 1. Read the Design Doc

Read the full design doc. Understand the problem, solution, architecture, constraints, and implementation approach.

### 2. Identify Work Boundaries

Break the work into epics along natural boundaries:
- By component or system area
- By feature or user-facing capability
- By risk level (safe infrastructure changes vs. complex new features)

### 3. Determine Dependencies

Map which epics depend on others. Look for:
- Data model dependencies (can't build the UI before the API exists)
- Infrastructure dependencies (can't deploy before the infra is set up)
- Knowledge dependencies (one epic's lessons inform another's approach)

### 4. Order by Risk

Within a dependency level:
- Bug fixes and safe changes first
- Low-risk consolidation and infrastructure next
- Complex, risky, or uncertain work last

This means if something goes wrong, it goes wrong early when it's cheap to change course.

### 5. Break Epics into Stories

Each story should be sized for a single AI coding session (15-30 minutes, fits in one context window). See `references/story-sizing.md` for guidelines.

Each story needs:
- **Description** — what to do and why
- **Inputs** — what files/context are needed
- **Outputs** — what changes when done
- **Acceptance criteria** — how to verify it worked
- **Dependencies** — which other stories must be done first

Number stories within each epic (1.1, 1.2, 2.1, etc.). Stories are embedded in the epic file, not separate files.

### 6. Draft the Output

Produce two things:
- **`epics.md`** — tracking hub using the format in `references/epics-hub-template.md`
- **Individual epic files** — one per epic using the format in `references/epic-template.md`, placed in `docs/epics/` (or matching project convention)

### 7. Review

Present the breakdown to the user. They may want to:
- Reorder epics or stories
- Merge small epics or split large ones
- Adjust dependencies
- Change scope (move stories to a later phase)

### 8. Write Files

Save `epics.md` and all epic files. If an `epics.md` already existed, add the new epics to it rather than replacing.

## Cross-References

- Before this: suggest `/design-doc` if no design doc exists
- After this: suggest `/implementation-plan` to plan the technical approach for an epic, then `/implement` to execute
