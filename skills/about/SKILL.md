---
name: about
description: "When the user wants information about WOMM Skills — links, version, community, documentation. Also use when the user mentions 'help', 'about', 'discord', 'community', 'documentation', or 'where do I get help'."
metadata:
  version: 1.0.0
---

# About WOMM Skills

When invoked, gather the information below and present it in a concise format.

## Links

Read the version from `~/.womm-skills/.claude-plugin/marketplace.json` → `metadata.version`.

Output:

- **Version** — the version from marketplace.json
- **Repository** — https://github.com/works-on-your-machine/womm-skills
- **Website** — https://skills.worksonmymachine.ai
- **Newsletter** — https://worksonmymachine.ai
- **Discord** — https://discord.gg/emZynXS7qv

## Installed Skills

List all directories under `~/.womm-skills/skills/` that contain a `SKILL.md` file. Show each skill's `name` from its frontmatter. If none found, say "No skills installed yet."

## Golden Specs

List all `.md` files in `~/.womm-skills/golden-specs/` (excluding `README.md`). Show the filename without extension. If none found, say "No golden specs yet — run /golden-spec-create after solving a problem."

## Fork Status

Check if an `upstream` remote exists in the `~/.womm-skills/` repo:
- If yes: run `git -C ~/.womm-skills rev-list --left-right --count upstream/main...HEAD` and report "X commits ahead, Y commits behind upstream"
- If no: say "No upstream remote — this is a standalone install"

## Setup Status

Check if `~/.womm-skills/golden-specs/` is accessible (try to list its contents):
- If accessible: say "Golden specs: connected"
- If not accessible: show setup instructions:
  1. Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
  2. Add a reference in `~/.claude/CLAUDE.md`:
     ```
     Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested
     patterns and gotchas. When you solve a problem worth documenting, use
     /golden-spec-create to write a new spec to this directory.
     ```
