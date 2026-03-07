# WOMM Skills

A self-improving skill suite for building software with AI. Ships with a repeatable workflow (design doc, epic breakdown, implementation plan, implement, capture) and a growing knowledge base of golden specs — documents written after solving real problems, covering what worked, what didn't, and the gotchas.

Designed to be forked and customized. Your fork diverges quickly. That's expected.

## Installation

### Option A: Fork & Clone (recommended)

Forking gives you full control — modify skills, add your own, and use `upstream-sync` to pull updates selectively.

```bash
# 1. Fork the repo on GitHub, then:
git clone https://github.com/YOUR-USERNAME/womm-skills ~/.womm-skills
cd ~/.womm-skills

# 2. Set upstream so you can pull future updates
git remote add upstream https://github.com/works-on-your-machine/womm-skills

# 3. Register as a local plugin marketplace
claude plugin marketplace add ~/.womm-skills

# 4. Install the plugin
claude plugin install womm-skills@womm-skills
```

### Option B: Plugin Marketplace

If you install from the marketplace, skills work immediately. But you still need a place for golden specs to accumulate:

```bash
# Create the golden specs directory
mkdir -p ~/.womm-skills/golden-specs
```

### After Either Option: Enable Golden Specs

Golden specs live outside the plugin cache so they're always available and don't require reinstall. Two setup steps:

**1. Allow file access** — Add `~/.womm-skills/` to your allowed paths in `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(~/.womm-skills/**)"
    ]
  }
}
```

**2. Add a reference** — Add this to your `~/.claude/CLAUDE.md` so every project can access your golden specs:

```
Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested patterns
and gotchas. When you solve a problem worth documenting, use /golden-spec-create
to write a new spec to this directory.
```

## How It Works

**Skills** are installed via the plugin system. They're cached and available in every project. Reinstall when upstream adds new skills.

**Golden specs** are flat markdown files at `~/.womm-skills/golden-specs/`. They're read directly from disk via the `~/.claude/CLAUDE.md` reference — no reinstall needed. Write a new one and every project sees it immediately.

Skills are the stable process. Golden specs are the growing knowledge.

## Skills

### Process Skills — The build, capture, learn loop

| Skill | What It Does |
|-------|-------------|
| `/design-doc` | Deep interview then structured design document |
| `/epic-breakdown` | Design doc to epics with stories, dependencies, and risk ordering |
| `/implementation-plan` | Epic to detailed technical plan (patterns, files, decisions) |
| `/implement` | Epic to story-by-story execution with tests and commits |
| `/golden-spec-create` | Solved problem to golden spec document |

### Fork Lifecycle Skills — Stay in sync, share what you build

| Skill | What It Does |
|-------|-------------|
| `/about` | Version, links, community, installed skills, fork status |
| `/upstream-sync` | Selectively pull upstream changes without losing customizations |
| `/fork-share` | Generate structured summary of what your fork changed and why |

## Customization

**Add golden specs:** Solve a problem, run `/golden-spec-create`, and the spec is written to `~/.womm-skills/golden-specs/`. Available in every project immediately.

**Modify skills:** Edit any `SKILL.md` in your fork. After editing, refresh the cache by running `claude plugin install womm-skills@womm-skills`.

**Add skills:** Create a new directory under `skills/` with a `SKILL.md` (YAML frontmatter + markdown instructions), add optional `references/` for templates, and update `.claude-plugin/marketplace.json`.

### Alternative Install

If you don't want the plugin system, copy the `skills/` directory into your project's `.claude/skills/`.

## Staying Current

If you forked (Option A), pull upstream changes selectively:

```bash
cd ~/.womm-skills
git fetch upstream
# Use /upstream-sync for guided selective merging
```

## Links

- **Repository:** [github.com/works-on-your-machine/womm-skills](https://github.com/works-on-your-machine/womm-skills)
- **Website:** [skills.worksonmymachine.ai](https://skills.worksonmymachine.ai)
- **Newsletter:** [worksonmymachine.ai](https://worksonmymachine.ai)
- **Discord:** [discord.gg/emZynXS7qv](https://discord.gg/emZynXS7qv)
- **License:** MIT
