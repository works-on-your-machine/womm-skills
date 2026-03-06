# Manifest Format Reference

The `womm-manifest.json` file lives in the repo root and describes what a fork has added or changed. The Fork Constellation reads this for richer metadata than raw diffs can provide.

## Schema

```json
{
  "fork_name": "string — human-readable name for this fork",
  "description": "string — one sentence describing what this fork is customized for",
  "stack": ["string — lowercase tech/framework names"],
  "golden_specs_added": [
    {
      "file": "FILENAME.md",
      "description": "string — what problem this solves, one sentence"
    }
  ],
  "skills_added": [
    {
      "skill": "skill-name",
      "description": "string — what this skill does, one sentence"
    }
  ],
  "skills_modified": [
    {
      "skill": "skill-name",
      "description": "string — why this was changed, one sentence"
    }
  ],
  "last_updated": "YYYY-MM-DD"
}
```

## Example

```json
{
  "fork_name": "My WOMM Setup",
  "description": "Customized for Next.js + Vercel + Planetscale",
  "stack": ["nextjs", "vercel", "planetscale", "typescript"],
  "golden_specs_added": [
    {
      "file": "NEXTJS_VERCEL_DEPLOY.md",
      "description": "Next.js App Router deployment to Vercel with edge runtime gotchas"
    },
    {
      "file": "PLANETSCALE_PRISMA_SETUP.md",
      "description": "Planetscale database setup with Prisma ORM and branching workflow"
    }
  ],
  "skills_added": [],
  "skills_modified": [
    {
      "skill": "design-doc",
      "description": "Added frontend-specific architecture sections for component hierarchy and state management"
    }
  ],
  "last_updated": "2026-03-05"
}
```

## Stack Naming Conventions

Use lowercase, common abbreviations:
- `rails`, `nextjs`, `django`, `express`, `fastapi`
- `aws`, `vercel`, `fly`, `render`, `railway`
- `postgres`, `mysql`, `planetscale`, `supabase`, `redis`
- `typescript`, `ruby`, `python`, `go`, `rust`
- `terraform`, `docker`, `kamal`
- `stripe`, `resend`, `clerk`, `auth0`

## Good Annotations vs Vague Ones

**Good:**
- "Next.js App Router deployment to Vercel with edge runtime gotchas"
- "Added frontend-specific architecture sections for component hierarchy and state management"

**Vague:**
- "Deployment stuff"
- "Some changes to design-doc"

Annotations should be specific enough that someone browsing the constellation can decide whether this fork's changes are relevant to them.

## How the Constellation Uses This

When the constellation pipeline finds a `womm-manifest.json`:
- Fork detail pages show user descriptions instead of raw file names
- The trending page groups forks by stack
- The activity feed shows meaningful summaries
- Search can match on stack keywords

Without a manifest, the constellation still works — it just shows file diffs without context.
