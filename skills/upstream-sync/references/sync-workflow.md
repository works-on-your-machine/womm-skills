# Sync Workflow Reference

Detailed git commands and edge case handling for the `/upstream-sync` skill.

## Git Commands by Step

### Fetch upstream
```bash
cd ~/.womm-skills
git fetch upstream
```

### Find the baseline
```bash
# From sync marker (preferred)
cat .womm-upstream-sync
# Output: abc1234

# From fork point (first sync fallback)
git merge-base upstream/main HEAD
# Output: abc1234
```

### List upstream changes
```bash
git diff --name-status <baseline>..upstream/main
# Output:
# A  skills/new-skill/SKILL.md          (Added)
# M  skills/design-doc/SKILL.md         (Modified)
# A  golden-specs/NEW_SPEC.md           (Added)
# M  .claude-plugin/marketplace.json    (Modified)
# D  some-old-file.md                   (Deleted)
```

### List local changes
```bash
git diff <baseline>..HEAD --name-only
# Output:
# skills/design-doc/SKILL.md           (user also modified this!)
# golden-specs/MY_CUSTOM_SPEC.md
# CLAUDE.md
```

### Pull in non-conflicting files
```bash
git checkout upstream/main -- skills/new-skill/SKILL.md
git checkout upstream/main -- golden-specs/NEW_SPEC.md
git checkout upstream/main -- .claude-plugin/marketplace.json
```

### Update the sync marker
```bash
git rev-parse upstream/main > .womm-upstream-sync
```

### Commit the sync
```bash
git add -A
git commit -m "Sync with upstream (abc1234..def5678)"
```

## Edge Cases

### First-Time Sync
No `.womm-upstream-sync` file exists. Use `git merge-base upstream/main HEAD` to find where the fork diverged. This works even if the user has made many commits since forking.

### Heavily Diverged Fork
If the fork has diverged significantly (many files changed on both sides), present a summary first before offering to pull anything in. The user may prefer to review changes manually.

### Deleted Files Upstream
If upstream deletes a file the user hasn't modified, it's safe to delete locally too. If the user has modified a deleted file, flag it — they may want to keep their version.

### New Skills in Upstream
When upstream adds new skills, they also update `marketplace.json`. Pull both the skill directory AND the marketplace.json update. Remind the user to reinstall the plugin.

### Merge Conflicts on marketplace.json
This file changes whenever skills are added. If both upstream and the user have modified it, the safest approach is to take the upstream version and re-add any local skill entries the user created.

## Example Output

```
Upstream changes since last sync (abc1234):

New skills:
  + skills/retrospective/  — New skill for post-project retros

Modified skills:
  ~ skills/design-doc/SKILL.md — Updated interview questions
    ⚠ You also modified this file

New golden specs:
  + golden-specs/DOCKER_COMPOSE_DEV.md — Docker Compose dev environment patterns

Config changes:
  ~ .claude-plugin/marketplace.json — Added retrospective skill
  ~ README.md — Updated skill list

Recommended actions:
  ✓ Pull in: skills/retrospective/, golden-specs/DOCKER_COMPOSE_DEV.md, README.md
  ✓ Pull in: marketplace.json (will re-add your local skills)
  ? Review: skills/design-doc/SKILL.md (both sides changed)
```

## The .womm-upstream-sync File

Contains a single line: the SHA of the last upstream commit that was synced. This is simpler and more reliable than tracking by date or tag:
- Always points to an exact commit
- Works even if upstream rebases or force-pushes (the SHA just won't be found, triggering fallback to merge-base)
- Updated atomically as part of the sync commit
