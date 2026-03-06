# Epic: `about` Skill

**Status:** Ready
**Phase:** 0 — Infrastructure
**Depends on:** Repo Scaffolding

## Goal

Give users a way to find help, community, and documentation. Every skill suite should have this.

## SKILL.md Structure

```
skills/about/
└── SKILL.md
```

No reference files needed — this is a simple output skill.

### Frontmatter
```yaml
---
name: about
description: "When the user wants information about WOMM Skills — links, version, community, documentation. Also use when the user mentions 'help', 'about', 'discord', 'community', 'documentation', or 'where do I get help'."
metadata:
  version: 1.0.0
---
```

### Behavior

Output the following:

- **Version** — Read from marketplace.json metadata
- **Repository** — GitHub link
- **Website** — skills.worksonmymachine.ai
- **Newsletter** — worksonmymachine.ai
- **Discord** — Invite link
- **Blog post** — Link to the post explaining the project
- **Installed skills** — List skills found in the local skills/ directory
- **Golden specs** — List golden specs found at `~/.womm-skills/golden-specs/`
- **Fork status** — If upstream remote exists, show how many commits ahead/behind
- **Setup status** — Check if `~/.womm-skills/golden-specs/` is accessible. If not, show setup instructions:
  1. Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
  2. Add reference in `~/.claude/CLAUDE.md` pointing to golden specs directory

Keep it short. No walls of text.

## Implementation Steps

1. Create SKILL.md
2. Update marketplace.json
3. Test: invoke `/about` and verify all links are correct
