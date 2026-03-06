# Golden Specs

Golden specs are documents written after solving a real problem. They capture what worked, what didn't, and the gotchas — so you don't have to figure things out twice.

## How They Work

Golden specs in this directory are read directly from `~/.womm-skills/golden-specs/` by any Claude Code project. They're not cached by the plugin system — write a new one and it's available everywhere immediately.

Your `~/.claude/CLAUDE.md` should reference this directory:

```
Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested patterns
and gotchas. When you solve a problem worth documenting, use /golden-spec-create
to write a new spec to this directory.
```

## Writing a Golden Spec

Run `/golden-spec-create` after solving a problem. The skill walks you through the format and writes the file here.

### Naming

Use `SCREAMING_SNAKE_CASE.md` matching the topic:
- `RAILS_AWS_TERRAFORM_CONFIGURATION.md`
- `NEXTJS_VERCEL_DEPLOY.md`
- `STRIPE_SUBSCRIPTION_BILLING.md`

### Sections

1. **Problem** — What were you trying to do? What made it hard?
2. **Why This Approach** — Alternatives and why this one wins
3. **Architecture** — How the pieces fit together
4. **Implementation** — Complete, copy-paste-ready code and config
5. **Common Gotchas** — What went wrong, why, and how to fix it
6. **Cost / Tradeoffs** — Numbers where possible
7. **Reference Implementations** — Links to working projects

The gotchas and implementation sections are the most valuable parts. Make them specific and complete.
