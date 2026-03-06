# Epic: `upstream-sync` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Let users selectively pull in upstream changes without blowing away their customizations. Regular `git rebase` or `git merge` from upstream would be a mess once someone has modified skills or added their own golden specs. This skill adds a semantic layer — Claude understands what changed and helps the user decide what to take.

## The Problem

After forking and customizing:
- Upstream adds a new skill → user wants it
- Upstream fixes a gotcha in a golden spec → user wants it
- Upstream rewrites a skill the user already customized → user probably doesn't want a blind overwrite

Git's merge tools operate on lines. This skill operates on meaning.

## SKILL.md Structure

```
skills/upstream-sync/
├── SKILL.md
└── references/
    └── sync-workflow.md
```

### Frontmatter
```yaml
---
name: upstream-sync
description: "When the user wants to check for and pull in upstream changes to their forked WOMM Skills. Also use when the user mentions 'sync', 'update skills', 'check upstream', 'what changed upstream', 'rebase', or 'pull in updates'. This skill operates on the womm-skills repo at ~/.womm-skills/, not the user's project."
metadata:
  version: 1.0.0
---
```

### Conventions (No Config File Needed)

Everything is discoverable:

- **Repo location**: `~/.womm-skills/` (clone destination)
- **Upstream remote**: `upstream` (standard git convention for forks). If missing, prompt user to add it: `git remote add upstream https://github.com/worksonmymachine/womm-skills`
- **Sync marker**: `.womm-upstream-sync` file in repo root. Contains the commit SHA of the last upstream commit the user synced with. If missing (first run), use `git merge-base upstream/main HEAD` to find the original fork point.
- **Golden specs**: `~/.womm-skills/golden-specs/` (read directly, not cached by plugin)
- **Skills**: `~/.womm-skills/skills/` (cached by plugin — reinstall after sync)
- **What the user customized**: `git diff` against the sync marker or fork point

### Workflow

1. **Check prerequisites**
   - Verify `~/.womm-skills/` exists and is a git repo (look for `.claude-plugin/marketplace.json`)
   - Verify `upstream` remote exists. If not, help the user add it.
   - Run `git fetch upstream`

2. **Find the baseline**
   - Read `.womm-upstream-sync` for the last-synced commit SHA
   - If file doesn't exist (first sync), use `git merge-base upstream/main HEAD`

3. **Discover upstream changes**
   - `git diff --name-status <baseline>..upstream/main` to get all changed files
   - Categorize each change:
     - **New skills** — new directories in `skills/`
     - **Modified skills** — changes to existing `skills/*/SKILL.md` or `skills/*/references/*`
     - **New golden specs** — new files in `golden-specs/`
     - **Modified golden specs** — changes to existing files in `golden-specs/`
     - **Config/docs changes** — marketplace.json, README, CLAUDE.md, etc.
     - **Other** — anything else

4. **Discover user's local changes**
   - `git diff <baseline>..HEAD --name-only` to see what the user has modified
   - Flag any files changed both upstream AND locally (potential conflicts)

5. **Present changes to user**
   - Group by category
   - For each change: show what changed (brief summary), whether the user also modified it
   - Recommend: pull in new files automatically, review modified files individually
   - For conflicts: show both diffs, explain what each side changed, ask user to decide

6. **Apply selected changes**
   - For new files (no conflict): `git checkout upstream/main -- path/to/file`
   - For files only changed upstream (user hasn't touched): same
   - For conflicting files: either take upstream version, keep local version, or help user merge manually
   - Stage and commit the sync

7. **Update the marker**
   - Write current `upstream/main` SHA to `.womm-upstream-sync`
   - Include in the sync commit

8. **Remind about plugin reinstall (only if skills changed)**
   - If upstream changes include new or modified skills: tell user to run `cd ~/.womm-skills && claude /plugin install .`
   - If only golden specs changed: no reinstall needed — golden specs are read directly from `~/.womm-skills/golden-specs/`

### Reference Files

**`references/sync-workflow.md`**
- Detailed git commands for each step
- How to handle edge cases: first-time sync, heavily diverged forks, deleted files
- What `.womm-upstream-sync` contains and why
- Example output showing a categorized change summary

## Implementation Steps

1. Create SKILL.md with the workflow above
2. Create reference file with git command details
3. Update marketplace.json
4. Test: fork the repo, make changes to both fork and upstream, run sync
