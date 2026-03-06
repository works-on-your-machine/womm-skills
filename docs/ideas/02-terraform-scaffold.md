# Epic: `terraform-scaffold` Knowledge Skill

**Status:** Backlog
**Phase:** 2 — Knowledge Skills
**Depends on:** Starter Golden Specs

## Goal

Scaffold a new project's terraform directory following established conventions. Generates the full directory structure, module skeleton, environment configs, and state backend configuration.

## SKILL.md Structure

```
skills/terraform-scaffold/
├── SKILL.md
└── references/
    ├── directory-structure.md
    └── naming-conventions.md
```

### Frontmatter
```yaml
---
name: terraform-scaffold
description: "When the user wants to set up terraform for a new project. Also use when the user mentions 'scaffold terraform', 'new terraform project', 'terraform directory structure', or 'set up infrastructure as code'. For a full Rails+AWS deployment, see rails-aws-deploy."
metadata:
  version: 1.0.0
---
```

### Workflow

1. **Read the project** — Get project name from CLAUDE.md or repo name, check for existing terraform files
2. **Ask** — What resources does this project need? (EC2, RDS, S3, etc.)
3. **Generate structure:**
   ```
   terraform/
   ├── modules/app/
   │   ├── main.tf          # Provider config
   │   ├── variables.tf     # All variables
   │   ├── outputs.tf       # All outputs
   │   └── [resource].tf    # One file per resource type
   ├── staging/
   │   ├── main.tf          # Backend config + module call
   │   └── terraform.tfvars # Environment-specific values
   └── production/
       ├── main.tf
       └── terraform.tfvars
   ```
4. **Configure state backend** — S3 bucket key, DynamoDB table, region
5. **Set up default tags** — Project, Environment, ManagedBy: terraform
6. **Generate resource files** — Skeleton for each requested resource

### Reference Files

**`references/directory-structure.md`**
- Why modules/app + staging/production (no backend switching)
- What goes in each file
- How to add a new resource type

**`references/naming-conventions.md`**
- Resource naming: `{project}-{resource}-{environment}`
- State key pattern: `{project}/{environment}/terraform.tfstate`
- Tagging strategy
- .gitignore entries (terraform.tfvars with secrets, .terraform/)

## Implementation Steps

1. Create SKILL.md
2. Create reference files
3. Update marketplace.json
4. Test: scaffold terraform for a new project, verify structure and naming
