---
name: upstream-sync
description: "When the user wants to check for and pull in upstream changes to their forked WOMM Skills. Also use when the user mentions 'sync', 'update skills', 'check upstream', 'what changed upstream', 'rebase', or 'pull in updates'. This skill operates on the womm-skills repo at ~/.womm-skills/, not the user's project."
metadata:
  version: 1.0.0
---

# Upstream Sync

Selectively pull upstream changes into a forked WOMM Skills repo without blowing away customizations. Git's merge tools operate on lines — this skill operates on meaning.

**This skill operates on `~/.womm-skills/`, not the user's current project.**

## Conventions

Everything is discoverable — no config file needed:

- **Repo location:** `~/.womm-skills/`
- **Upstream remote:** `upstream` (standard git convention for forks)
- **Sync marker:** `.womm-upstream-sync` in repo root — contains the SHA of the last upstream commit synced
- **Golden specs:** `~/.womm-skills/golden-specs/` (read directly, not cached)
- **Skills:** `~/.womm-skills/skills/` (cached by plugin — reinstall after sync if skills changed)

## Workflow

### 1. Check Prerequisites

- Verify `~/.womm-skills/` exists and is a git repo (look for `.claude-plugin/marketplace.json`)
- Check if `upstream` remote exists. If not, help the user add it:
  ```
  cd ~/.womm-skills
  git remote add upstream https://github.com/works-on-your-machine/womm-skills
  ```
- Run `git fetch upstream`

### 2. Find the Baseline

- Read `.womm-upstream-sync` for the last-synced commit SHA
- If the file doesn't exist (first sync), find the original fork point:
  ```
  git merge-base upstream/main HEAD
  ```

### 3. Discover Upstream Changes

Run `git diff --name-status <baseline>..upstream/main` and categorize each change:

- **New skills** — new directories in `skills/`
- **Modified skills** — changes to existing `skills/*/SKILL.md` or `skills/*/references/*`
- **New golden specs** — new files in `golden-specs/`
- **Modified golden specs** — changes to existing files in `golden-specs/`
- **Config/docs changes** — `marketplace.json`, `README.md`, `CLAUDE.md`, etc.
- **Other** — anything else

### 4. Discover Local Changes

Run `git diff <baseline>..HEAD --name-only` to see what the user has modified. Flag any files changed both upstream AND locally — these are potential conflicts.

### 5. Present Changes

Group by category. For each change show:
- What changed (brief summary)
- Whether the user also modified this file (conflict risk)

Recommend:
- **New files (no conflict):** Pull in automatically
- **Files only changed upstream:** Pull in automatically
- **Conflicting files:** Show both sides, explain what each changed, ask the user to decide

### 6. Apply Selected Changes

- **New or non-conflicting files:** `git checkout upstream/main -- path/to/file`
- **Conflicting files:** Either take upstream version, keep local version, or help the user merge manually
- Stage and commit the sync

### 7. Update the Sync Marker

Write current `upstream/main` SHA to `.womm-upstream-sync`. Include in the sync commit.

### 8. Remind About Plugin Reinstall

- **If skills changed:** Tell the user to run `cd ~/.womm-skills && claude /plugin install .`
- **If only golden specs changed:** No reinstall needed — golden specs are read directly from disk
