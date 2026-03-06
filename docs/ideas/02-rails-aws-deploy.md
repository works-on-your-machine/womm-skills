# Epic: `rails-aws-deploy` Knowledge Skill

**Status:** Backlog
**Phase:** 2 — Knowledge Skills
**Depends on:** Starter Golden Specs

## Goal

Turn the Rails+AWS+Terraform golden spec into an executable skill that walks through implementation step by step. Unlike the golden spec (which is a reference document), this skill actively guides the user through setup, adapting to their specific project.

## SKILL.md Structure

```
skills/rails-aws-deploy/
├── SKILL.md
└── references/
    ├── architecture-overview.md
    ├── terraform-resources.md
    ├── kamal-configuration.md
    └── common-gotchas.md
```

### Frontmatter
```yaml
---
name: rails-aws-deploy
description: "When the user wants to deploy a Rails 8 application to AWS using Kamal and Terraform. Also use when the user mentions 'deploy rails to aws', 'set up kamal', 'terraform for rails', 'ec2 deployment', or 'aurora setup'. For scaffolding just the terraform directory, see terraform-scaffold. For Stripe integration, see stripe-subscriptions."
metadata:
  version: 1.0.0
---
```

### Workflow

The skill should walk through these stages in order:

1. **Check prerequisites** — Rails 8 app exists, AWS CLI configured, Terraform installed, Docker running
2. **Terraform setup** — Scaffold terraform directory (suggest `/terraform-scaffold`), configure state backend, create resources
3. **Kamal setup** — Generate deploy.yml, environment configs, secrets files
4. **IAM & security** — Instance profile, security groups, least-privilege policies
5. **Database** — Aurora Serverless v2 configuration, additional databases for Solid Queue/Cache/Cable
6. **Deploy** — First deploy walkthrough, health check verification
7. **Post-deploy** — SES setup, monitoring, backup verification

### Reference Files

**`references/architecture-overview.md`**
- Architecture diagram: Cloudflare → EC2 → Docker → Aurora
- Why this over ECS/Fargate (comparison table)
- Cost breakdown per environment

**`references/terraform-resources.md`**
- Complete list of terraform resources created
- Naming conventions
- Minimum sizing (t3.small, 0.5 ACU, 20GB EBS) with explanations

**`references/kamal-configuration.md`**
- deploy.yml structure
- Environment-specific overrides (staging/production)
- Secrets management (Kamal secrets files + AWS Secrets Manager)
- Pre-build hooks (Tailwind on ARM Mac)

**`references/common-gotchas.md`**
- All gotchas from the golden spec, expanded with "how to diagnose" for each
- t3.micro OOM, Docker bypassing UFW, Kamal health check + Rack Attack, Aurora cold starts, etc.

## Implementation Steps

1. Create SKILL.md with the guided workflow
2. Extract reference content from the golden spec into the 4 reference files
3. Update marketplace.json
4. Test: walk through the full setup in a new Rails project
