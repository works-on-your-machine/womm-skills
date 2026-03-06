---
name: fork-share
description: "When the user wants to share what they've built in their fork with the WOMM Skills community. Also use when the user mentions 'share my fork', 'contribute back', 'show what I changed', 'update manifest', or 'tell others about my fork'. This skill operates on the womm-skills repo at ~/.womm-skills/, not the user's project."
metadata:
  version: 1.0.0
---

# Fork Share

Generate a structured summary of what your fork changed and why. The diff tells you *what* — this skill captures the *why*.

**This skill operates on `~/.womm-skills/`, not the user's current project.**

## Workflow

### 1. Check Prerequisites

- Verify `~/.womm-skills/` exists and is a git repo
- Check if `upstream` remote exists (needed to diff against original). If not, help the user add it:
  ```
  cd ~/.womm-skills
  git remote add upstream https://github.com/works-on-your-machine/womm-skills
  ```
- Run `git fetch upstream`

### 2. Discover What Changed

Find the fork point and diff against it:
```
git merge-base upstream/main HEAD
git diff --name-status <fork-point>..HEAD
```

Categorize changes:
- **New golden specs** — files added to `golden-specs/`
- **New skills** — new directories in `skills/`
- **Modified skills** — changes to existing skills
- **CLAUDE.md changes** — stack/preference updates
- **Other modifications**

### 3. Ask the User to Annotate

For each change, ask for a one-sentence description:
- **New golden spec:** "What problem does this solve?"
- **New skill:** "What does this skill do?"
- **Modified skill:** "Why did you change this?"
- **CLAUDE.md changes:** "What's your stack?" (parse automatically from the file if possible)

### 4. Generate `womm-manifest.json`

Write to the `~/.womm-skills/` repo root. See `references/manifest-format.md` for the full schema.

### 5. Format for Sharing (Optional)

Ask the user if they'd like a message formatted for Discord (`#show-your-machine`):

```
Just updated my WOMM fork:
  Stack: Next.js + Vercel + Planetscale
  Added golden spec: NEXTJS_VERCEL_DEPLOY.md
    → Next.js App Router deployment to Vercel with edge runtime gotchas
  Modified: design-doc (added frontend-specific architecture sections)
```

### 6. Commit the Manifest

Stage and commit `womm-manifest.json`. The constellation pipeline picks this up automatically on next poll.
