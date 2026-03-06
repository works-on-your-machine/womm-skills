# Epic: Starter Golden Specs

**Status:** Backlog
**Phase:** 2 — Knowledge Skills
**Depends on:** Phase 0

## Goal

Package the existing Rails+AWS+Terraform and Rails+Stripe golden specs into the repo's `golden-specs/` directory as starter content. When users clone to `~/.womm-skills/`, these are immediately available in every project via the `~/.claude/CLAUDE.md` reference. They serve as examples of what golden specs look like and provide immediate value for anyone with a similar stack.

## Deliverables

### `golden-specs/RAILS_AWS_TERRAFORM_CONFIGURATION.md`
Adapted from `/Users/swerner/Development/golden_specs/RAILS_AWS_TERRAFORM_CONFIGURATION.md`.

Changes needed:
- Remove references to local paths (`~/Development/shipyard/fleet/`, etc.)
- Replace with generic "your project" language
- Add a preamble: "This is a starter golden spec. It documents a proven pattern for deploying Rails 8 apps to AWS. Replace or modify it for your own setup."
- Keep all technical content intact — the gotchas and implementation details are the value

### `golden-specs/RAILS_STRIPE_INTEGRATION.md`
Adapted from `/Users/swerner/Development/golden_specs/RAILS_STRIPE_INTEGRATION.md`.

Same changes: remove local paths, add preamble, keep technical content.

### `golden-specs/README.md`
Update the existing README to:
- Explain that these are starter specs
- List the included specs with a one-line summary each
- Explain how to add your own (naming convention, format reference)
- Link to `/golden-spec-create` skill for guided creation

## Implementation Steps

1. Copy and adapt RAILS_AWS_TERRAFORM_CONFIGURATION.md
2. Copy and adapt RAILS_STRIPE_INTEGRATION.md
3. Update golden-specs/README.md
4. Review: make sure no personal paths or project-specific references remain
