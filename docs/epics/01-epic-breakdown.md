# Epic: `epic-breakdown` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Take a design doc and produce an `epics.md` tracking hub plus individual epic files with dependencies, risk ordering, and status tracking. This is the bridge between "what we're building" and "what we're doing next."

## How It Works

1. User invokes `/epic-breakdown` and points to a design doc (or the skill finds it)
2. Skill reads the project (CLAUDE.md, existing docs structure) for conventions
3. Skill reads the design doc
4. Produces: tracking hub + individual epic files

## SKILL.md Structure

```
skills/epic-breakdown/
├── SKILL.md
└── references/
    ├── epic-template.md
    ├── epics-hub-template.md
    └── story-sizing.md
```

### Frontmatter
```yaml
---
name: epic-breakdown
description: "When the user wants to break a design document into implementable epics with dependencies and status tracking. Also use when the user mentions 'break this down', 'create epics', 'plan the work', 'what do we build first', 'slice this', 'make stories', or 'context-window sized'. For creating the design doc first, see design-doc."
metadata:
  version: 1.0.0
---
```

### Output Structure

**`epics.md` (tracking hub)**
- Status key (Done, In Progress, Ready, Backlog, Idea)
- Table of all epics with: name, status, dependencies, link to epic file
- Organized by phase/priority

**Individual epic files (in `docs/epics/` or matching project convention)**
Each file contains:
- Status, depends on, blocks
- Goal (1-2 sentences)
- Scope (what's in, what's out)
- Stories — numbered (1.1, 1.2, etc.), each with:
  - Description of what to do and why
  - What files/context are needed (inputs)
  - What changes when done (outputs)
  - Acceptance criteria (how to verify)
  - Dependencies on other stories
- Stories should be sized for a single AI coding session (15-30 min, fits in one context window)
- Each story should be self-contained enough that someone with no prior context could pick it up and execute it
- Implementation ordering (by risk: safe changes first, risky changes last)

### Workflow in SKILL.md

```markdown
## Before Starting
1. Check that `~/.womm-skills/golden-specs/` is accessible. If not, provide setup instructions (add path to `~/.claude/settings.json`, add reference in `~/.claude/CLAUDE.md`).
2. Read the project for structure conventions (CLAUDE.md, existing docs/, golden specs at `~/.womm-skills/golden-specs/`)
2. Find the design doc (ask the user or look in docs/)
3. Check for existing epics.md — if present, we're adding to it, not replacing

## Process
1. Read the design doc fully
2. Identify natural work boundaries (by component, by feature, by risk level)
3. Determine dependencies between pieces
4. Order by risk: bug fixes and safe changes first, complex/risky work last
5. Draft the epics hub and individual files
6. Present for review — the user may want to reorder, merge, or split epics
7. Write files

## Ordering Principles
- Things with no dependencies go first
- Within a dependency level, order by risk (lowest risk first)
- Group related changes even if they could technically be separate
- Each epic should be completable in a reasonable session (not multi-week)

## Story Sizing
- Each story should be 15-30 minutes of AI-assisted work (fits in one context window)
- Too big: story touches multiple concerns or requires holding too much context
- Too small: story doesn't produce meaningful progress
- Include enough context in each story that an AI agent can pick it up cold — what files to read, what the goal is, how to verify it worked

## Output
- `epics.md` in the project's docs root
- Individual epic files in `docs/epics/` (or matching convention)
- Stories embedded within each epic (not separate files)

## Cross-References
- Before this: suggest `/design-doc` if no design doc exists
- After this: suggest `/implementation-plan` to plan the technical approach, then `/implement` to execute
```

### `references/epic-template.md`

Template for individual epic files with all sections and guidance on each.

### `references/epics-hub-template.md`

Template for the tracking hub with the status table format.

### `references/story-sizing.md`

Guidelines for sizing stories within epics:
- How to judge if a story is the right size (15-30 min, one context window)
- Common mistakes: too big (multiple concerns), too small (no meaningful progress)
- How to write story context so an AI agent can pick it up cold
- What inputs/outputs/acceptance criteria look like in practice

## Implementation Steps

1. Create `skills/epic-breakdown/SKILL.md`
2. Create `skills/epic-breakdown/references/epic-template.md`
3. Create `skills/epic-breakdown/references/epics-hub-template.md`
4. Update `marketplace.json`
5. Test: point at a design doc, verify it produces hub + individual files
6. Test: run against a project that already has epics, verify it adds rather than replaces
