# Epic: `problem-extractor` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** golden-spec-create

## Goal

Scan a conversation, debug session, or git history for problems the user solved, and draft golden specs from them. This is how the system grows from use — you work normally, then run this skill to capture what you learned.

## How It Works

1. User invokes `/problem-extractor` after a work session, or says "what did we learn?"
2. Skill scans the available context (conversation, recent commits, modified files)
3. Identifies moments where something was hard, unexpected, or required a workaround
4. For each identified problem: drafts a golden spec using the golden-spec-create template
5. Presents drafts for review — user picks which to keep, which to discard

## SKILL.md Structure

```
skills/problem-extractor/
├── SKILL.md
└── references/
    └── extraction-patterns.md
```

### Frontmatter
```yaml
---
name: problem-extractor
description: "When the user wants to review a recent work session and capture what they learned. Also use when the user mentions 'what did we figure out', 'extract learnings', 'capture problems', 'what should be a golden spec', or 'review this session'. For writing a single golden spec manually, see golden-spec-create."
metadata:
  version: 1.0.0
---
```

### What to Look For

The skill should identify:
- **Configuration gotchas** — "we had to set X to Y because otherwise Z happens"
- **Version-specific issues** — "this changed in version X, the docs still show the old way"
- **Unexpected behavior** — "turns out A doesn't do what you'd expect, you need B instead"
- **Sizing / performance discoveries** — "X is too small, you need at least Y"
- **Workarounds** — "the official way doesn't work because of Z, here's what works"
- **Integration issues** — "connecting A to B requires this non-obvious step"
- **Cost discoveries** — "this costs more/less than expected because of X"

### Workflow in SKILL.md

```markdown
## Before Starting
1. Check that `~/.womm-skills/golden-specs/` is accessible
   - If not: tell the user to add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - And add a reference in `~/.claude/CLAUDE.md`:
     "Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested patterns and gotchas."
2. Read existing golden specs in `~/.womm-skills/golden-specs/` to avoid duplicating already-captured knowledge

## Process
1. Scan the conversation (or ask the user what they worked on)
2. List candidate problems found — brief one-liner each
3. Ask the user which ones are worth capturing
4. For each selected problem: draft a golden spec following golden-spec-create format
5. Present drafts for review
6. Write approved specs to `~/.womm-skills/golden-specs/` — available in every project immediately

## Signals That Something Is Worth Capturing
- The user said "oh, that's why" or "I didn't expect that"
- Something took multiple attempts to get right
- A default value or configuration had to be changed
- The solution was different from what documentation suggested
- A library or service behaved differently than expected
- Something that would bite the user again in a future project

## Signals to Skip
- Standard implementation following documentation
- Preference choices (tabs vs spaces, naming conventions)
- Project-specific decisions that won't generalize

## Output
- One or more draft golden specs in `golden-specs/`
- Each named per golden-spec-create conventions

## Cross-References
- Uses golden-spec-create's template and format
- If there's nothing to extract: that's fine, say so
```

### `references/extraction-patterns.md`

Examples of the kinds of problems to look for, organized by category. Each with a "before" (raw conversation) and "after" (extracted golden spec draft) example.

## Implementation Steps

1. Create `skills/problem-extractor/SKILL.md`
2. Create `skills/problem-extractor/references/extraction-patterns.md`
3. Update `marketplace.json`
4. Test: run after a real work session, verify it identifies actual problems
5. Test: run in a clean session with nothing to extract, verify it doesn't force output
