# Epic: `retrospective` Skill

**Status:** Backlog
**Phase:** 4 — Maintenance & Growth
**Depends on:** golden-spec-create, problem-extractor

## Goal

Run a structured retro after completing an epic, project, or significant piece of work. Identifies what should become golden specs, what skills were missing, and what skills need updating. Feeds back into the system.

## SKILL.md Structure

```
skills/retrospective/
├── SKILL.md
└── references/
    └── retro-template.md
```

### Frontmatter
```yaml
---
name: retrospective
description: "When the user has finished a project, epic, or significant piece of work and wants to capture what they learned. Also use when the user mentions 'retro', 'retrospective', 'what did we learn', 'post-mortem', or 'review the project'. For extracting problems from a single session, see problem-extractor."
metadata:
  version: 1.0.0
---
```

### Workflow

0. **Check golden specs access** — Verify `~/.womm-skills/golden-specs/` is accessible. If not, provide setup instructions (add path to `~/.claude/settings.json`, add reference in `~/.claude/CLAUDE.md`).
1. **Scope the retro** — What work are we reviewing? (epic, project, sprint)
2. **Review what happened** — Read epics, design docs, commits, golden specs at `~/.womm-skills/golden-specs/`
3. **Generate findings:**
   - What went well (keep doing)
   - What was harder than expected (and why)
   - What we'd do differently
   - What should become a golden spec (run problem-extractor if needed)
   - What skills were missing
   - What skills need updating (based on what changed)
4. **Write retro doc** to docs/ or docs/archive/
5. **Create action items** — Draft golden specs to `~/.womm-skills/golden-specs/`, file skill improvement notes

### Reference Files

**`references/retro-template.md`**
- Template with all sections
- Guidance on what to look for in each section

## Implementation Steps

1. Create SKILL.md
2. Create reference file
3. Update marketplace.json
4. Test: run retro on a completed piece of work
