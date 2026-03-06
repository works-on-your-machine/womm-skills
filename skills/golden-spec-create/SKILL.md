---
name: golden-spec-create
description: "When the user wants to document a solution, pattern, or configuration they've figured out so they don't have to figure it out again. Also use when the user mentions 'golden spec', 'write this down', 'document this pattern', 'capture what we learned', or 'save this for next time'. For automatically extracting problems from a conversation, see problem-extractor."
metadata:
  version: 1.0.0
---

# Golden Spec Create

Structure a new golden spec from a conversation, debug session, or manual input. Golden specs capture hard-won knowledge — what worked, what didn't, and the gotchas — so you don't have to figure things out twice.

## Before Starting

1. **Check golden specs access.** Try to list files in `~/.womm-skills/golden-specs/`. If not accessible, tell the user:
   - Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - Add a reference in `~/.claude/CLAUDE.md`:
     ```
     Reference golden specs at ~/.womm-skills/golden-specs/ for battle-tested
     patterns and gotchas. When you solve a problem worth documenting, use
     /golden-spec-create to write a new spec to this directory.
     ```

2. **Read existing golden specs** in `~/.womm-skills/golden-specs/` to match format and depth. If no golden specs exist yet, use the template in `references/golden-spec-template.md`.

## Process

### 1. Understand What Was Solved

Ask the user what they solved, or read from the conversation context. Get the full picture: what they were trying to do, what went wrong, and what finally worked.

### 2. Draft Each Section

For each section in the golden spec format, draft based on what you know and ask for clarification where needed:

1. **Problem** — What were you trying to do? What made it hard?
2. **Why This Approach** — What alternatives exist? Why is this one better? Include a comparison table or brief pros/cons if relevant.
3. **Architecture** — How the pieces fit together. Diagram or description of the flow.
4. **Implementation** — Complete, copy-paste-ready code and configuration. Not abbreviated. Include file paths.
5. **Common Gotchas** — Things that went wrong or were surprising. Each one should include: what happens, why it happens, how to fix it.
6. **Cost / Tradeoffs** — Monthly costs, performance tradeoffs, maintenance burden. Numbers where possible.
7. **Reference Implementations** — Links or paths to working projects that use this pattern.

### 3. Focus on Gotchas

The gotchas section is the most valuable part. Push for specifics:
- Not "be careful with configuration" but "setting `pool_size` below 5 causes connection timeouts under load because Sidekiq defaults to 5 threads"
- Not "testing can be tricky" but "Stripe webhook tests need `OpenSSL::HMAC.hexdigest` to generate valid signatures — the Stripe gem's test helpers don't work in controller tests"

### 4. Make Implementation Complete

Code should be copy-paste-ready:
- Include full file paths
- Don't abbreviate with `...` or "add your config here"
- Show complete files when the context matters, not just the changed lines
- Include environment variables, configuration files, and migration files

### 5. Strip Project-Specific Details

Golden specs should be generic and reusable across any project. Before finalizing:
- Replace real project names, domains, and URLs with placeholders (`myapp`, `example.com`)
- Replace real S3 bucket names, database names, and secret IDs with `{project}-{environment}-{resource}` patterns
- Remove references to specific local paths (`~/Development/...`) — use generic descriptions instead
- Replace real API keys, account IDs, and org-specific values with placeholder formats
- Keep specific version numbers, gem names, and AWS instance sizes — those are the useful parts

The goal: someone with a completely different project should be able to follow the spec by substituting their own names.

### 6. Review

Present the draft for review. Check:
- Could someone follow this without any other context?
- Are the gotchas specific (not generic warnings)?
- Is the code complete (not placeholder)?
- Are costs actual numbers, not "it depends"?
- Are project-specific identifiers replaced with generic placeholders?

### 7. Write

Save to `~/.womm-skills/golden-specs/`. Available in every project immediately.

## Naming

`SCREAMING_SNAKE_CASE.md` matching the topic:
- `RAILS_AWS_TERRAFORM_CONFIGURATION.md`
- `NEXTJS_VERCEL_DEPLOY.md`
- `STRIPE_SUBSCRIPTION_BILLING.md`
- `POSTGRES_FULL_TEXT_SEARCH.md`

## Cross-References

- If the user describes a problem they haven't solved yet: suggest `/design-doc`
- If the user wants to scan a conversation for multiple learnings: suggest `/problem-extractor` (when available)
