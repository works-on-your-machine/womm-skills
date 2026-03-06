# Skill Ideas — Future Possibilities

Ideas for additional skills or skill categories. Not committed — just captured for when the need becomes clear.

---

## Ideation / Interview Skill

The deep interview flow is already baked into design-doc and implementation-plan. But there might be a lighter-weight version for when you're just brainstorming — exploring an idea without committing to a full design doc. Could produce a quick exploration doc or just structured notes.

Could also be the basis for a "new skill" creation flow — interview the user about what process they keep repeating, then draft a SKILL.md from the conversation.

---

## Thinking Technique Skills

Approaches from [The Different Shapes of Think Before You Build](https://worksonmymachine.ai/p/the-different-shapes-of-think-before). Deferred because each is a one-sentence prompt rather than a multi-step process. Could become skills if worked examples and templates add enough value beyond just asking Claude directly.

- **Contrapositive Spec** — Define a feature by listing all failure modes first, then derive requirements from what's left. Useful when positive requirements are vague but you know exactly what shouldn't happen.
- **DSL Scaffold** — Create vocabulary (nouns, verbs, relationships) for a problem domain before solving it. Forces precision and makes future conversations clearer.
- **Inversion** — "What if we did the opposite of the obvious approach?" Useful for finding non-obvious solutions.
- **Abductive Review** — Before copying a pattern, explain WHY it works. Prevents cargo-culting.

If any of these turn out to need specific templates, worked examples, or multi-step workflows to be useful, they're worth revisiting as skills.

---

## Automation / Meta Skills

Skills that operate on the skill suite or project docs themselves. Deferred because the underlying actions are simple enough to just ask Claude to do.

- **Kickoff** — Chains design-doc → epic-breakdown → story-slice in sequence. You can just run them one after another.
- **Skill Grower** — Reads golden specs and proposes new skills when patterns repeat. In practice, you already know when something should be a skill.
- **Spec Status** — Dashboard showing doc status across epics. Just ask Claude "what's the status of my epics?"
- **Archive Sweep** — Moves completed/stale docs to archive. The judgment of when to archive is the hard part, not the `mv` command.
- **Dep Check** — Audits epic dependency graph for circular deps or ordering issues. Rare problem, epics.md is readable.

---

## Capture / Extraction Skills

Skills that close the feedback loop by capturing learnings. Deferred because users can just run `/golden-spec-create` directly when they notice a pattern worth documenting.

- **`problem-extractor`** — Scans conversations for solved problems, drafts golden specs automatically. The automation is nice but the manual trigger (`/golden-spec-create`) covers the same ground. Epic spec exists at `docs/epics/01-problem-extractor.md`.
- **`retrospective`** — Structured post-project retro that feeds findings back into golden specs. Useful process but not essential for launch — users can just reflect and write specs. Epic spec exists at `docs/epics/03-retrospective.md`.

---

## Knowledge Skills

Skills that wrap golden specs with guided workflows. Deferred because the golden specs themselves (read directly from `~/.womm-skills/golden-specs/`) provide the same value — Claude can read and apply them without a dedicated skill.

- **`rails-aws-deploy`** — Guided Rails 8 + Kamal + AWS + Terraform setup. The golden spec `RAILS_AWS_TERRAFORM_CONFIGURATION.md` covers this. Epic spec exists at `docs/epics/02-rails-aws-deploy.md`.
- **`stripe-subscriptions`** — Guided Stripe subscription billing integration. The golden spec `RAILS_STRIPE_INTEGRATION.md` covers this. Epic spec exists at `docs/epics/02-stripe-subscriptions.md`.
- **`terraform-scaffold`** — Scaffold terraform directory structure with naming conventions. Better as a golden spec — the conventions are the value. Epic spec exists at `docs/epics/02-terraform-scaffold.md`.

---

## Infrastructure / Operations Skills

Skills for specific infrastructure patterns beyond the starter golden specs.

- **Infra Audit** — Audit terraform against naming conventions and patterns. Could be useful once the conventions are well-established.
- **Kamal Deploy Checklist** — Step-by-step guided deployment with Kamal. Might overlap with rails-aws-deploy golden spec.
- **Cost Estimator** — Estimate AWS costs for a given architecture. Could pull from golden specs' cost sections.

---

## Community / Fork Skills

Skills related to the fork constellation and community features.

- **Fork Map** — Local skill that queries the constellation API and shows what other forks have done. Could be useful once the constellation has enough forks to browse.

---

## New Knowledge Skill Categories

The starter golden specs cover Rails+AWS+Terraform and Stripe. Other categories that could ship as knowledge skills if golden specs are written for them:

- **CI/CD** — GitHub Actions patterns, deployment pipelines
- **Auth** — Authentication/authorization patterns (Devise, OAuth, passkeys)
- **Testing** — Test strategy patterns, what to test at each level
- **Monitoring** — Observability setup, alerting, error tracking
- **Database** — Migration patterns, indexing strategies, backup/restore
