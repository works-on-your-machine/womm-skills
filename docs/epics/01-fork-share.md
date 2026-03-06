# Epic: `fork-share` Skill

**Status:** Ready
**Phase:** 1 — Core Skills
**Depends on:** Phase 0

## Goal

Give users a structured way to share what they've built in their fork. The Fork Constellation already polls forks via GitHub API and sees file diffs, but it can't know *why* someone made a change. This skill generates rich metadata that makes the constellation more useful for everyone.

## The Problem

The constellation sees: "user added `golden-specs/NEXTJS_VERCEL_DEPLOY.md`"
But can't see: "I wrote this after spending 3 hours debugging Vercel's edge runtime with Next.js App Router"

The diff tells you *what*. This skill captures the *why*.

## SKILL.md Structure

```
skills/fork-share/
├── SKILL.md
└── references/
    └── manifest-format.md
```

### Frontmatter
```yaml
---
name: fork-share
description: "When the user wants to share what they've built in their fork with the WOMM Skills community. Also use when the user mentions 'share my fork', 'contribute back', 'show what I changed', 'update manifest', or 'tell others about my fork'. This skill operates on the womm-skills repo at ~/.womm-skills/, not the user's project."
metadata:
  version: 1.0.0
---
```

### Workflow

1. **Check prerequisites**
   - Verify `~/.womm-skills/` exists and is a git repo
   - Verify `upstream` remote exists (needed to diff against original)
   - Run `git fetch upstream`

2. **Discover what changed**
   - Find the fork point: `git merge-base upstream/main HEAD`
   - `git diff --name-status <fork-point>..HEAD` to see all changes
   - Categorize:
     - **New golden specs** — files added to `golden-specs/`
     - **New skills** — new directories in `skills/`
     - **Modified skills** — changes to existing skills
     - **CLAUDE.md changes** — stack/preference updates
     - **Other modifications**

3. **Ask the user to annotate**
   - For each new golden spec: "What problem does this solve? One sentence."
   - For each new skill: "What does this skill do? One sentence."
   - For each modified skill: "Why did you change this? One sentence."
   - For CLAUDE.md changes: "What's your stack?" (parse automatically if possible)

4. **Generate `womm-manifest.json`**
   - Write to repo root
   - Structure:

   ```json
   {
     "fork_name": "My WOMM Setup",
     "description": "Customized for Next.js + Vercel + Planetscale",
     "stack": ["nextjs", "vercel", "planetscale"],
     "golden_specs_added": [
       {
         "file": "NEXTJS_VERCEL_DEPLOY.md",
         "description": "Next.js App Router deployment to Vercel with edge runtime gotchas"
       }
     ],
     "skills_added": [],
     "skills_modified": [
       {
         "skill": "design-doc",
         "description": "Added frontend-specific architecture sections"
       }
     ],
     "last_updated": "2026-03-05"
   }
   ```

5. **Optionally format for Discord**
   - Generate a message the user can paste into `#show-your-machine`:
   ```
   Just updated my WOMM fork:
     Stack: Next.js + Vercel + Planetscale
     Added golden spec: NEXTJS_VERCEL_DEPLOY.md
       → Next.js App Router deployment to Vercel with edge runtime gotchas
     Modified: design-doc (added frontend-specific architecture sections)
   ```

6. **Commit the manifest**
   - Stage and commit `womm-manifest.json`
   - The constellation pipeline picks this up automatically on next poll

### Reference Files

**`references/manifest-format.md`**
- Full `womm-manifest.json` schema
- Examples of good annotations vs vague ones
- How the constellation uses this data
- Stack naming conventions (lowercase, common abbreviations)

## How This Feeds the Constellation

The constellation pipeline already diffs forks. When it finds a `womm-manifest.json`, it uses the richer metadata instead of raw diffs:
- Fork detail pages show the user's descriptions, not just file names
- Trending page can group by stack
- Feed shows meaningful summaries

Without the manifest, everything still works — just with less context.

## Implementation Steps

1. Create SKILL.md with the workflow above
2. Create reference file with manifest schema and examples
3. Update marketplace.json
4. Test: run in a fork with changes, verify manifest and Discord message look right
