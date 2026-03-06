# Epic: `stripe-subscriptions` Knowledge Skill

**Status:** Backlog
**Phase:** 2 — Knowledge Skills
**Depends on:** Starter Golden Specs

## Goal

Turn the Stripe golden spec into an executable skill that guides the user through adding subscription billing to a Rails app. Covers the full flow: Stripe account setup, checkout, webhooks, billing portal, invoice sync.

## SKILL.md Structure

```
skills/stripe-subscriptions/
├── SKILL.md
└── references/
    ├── stripe-setup-checklist.md
    ├── webhook-handling.md
    └── common-gotchas.md
```

### Frontmatter
```yaml
---
name: stripe-subscriptions
description: "When the user wants to add Stripe subscription billing to a Rails application. Also use when the user mentions 'stripe integration', 'subscription billing', 'stripe webhooks', 'billing portal', or 'payment integration'. For deploying the Rails app itself, see rails-aws-deploy."
metadata:
  version: 1.0.0
---
```

### Workflow

1. **Stripe account setup** — Products, Prices, webhook endpoints, Customer Portal config
2. **Secrets management** — 4 keys per environment into AWS Secrets Manager
3. **Models & migrations** — User model additions, Subscription model, Invoice model
4. **Checkout flow** — Stripe::Customer creation, Checkout Session, success redirect
5. **Webhook controller** — 5 events, signature verification, idempotent processing
6. **Billing portal** — Session creation, linking from user dashboard
7. **Invoice sync** — Background job to fetch and store invoice data
8. **Testing** — Webhook signature generation, API stubbing

### Reference Files

**`references/stripe-setup-checklist.md`**
- Step-by-step Stripe Dashboard setup
- Sandbox vs Live differences
- Which webhook events to subscribe to (and why each one)
- Customer Portal configuration options

**`references/webhook-handling.md`**
- Controller setup (skip before_actions)
- Signature verification code
- Handler for each of the 5 events
- Idempotent write pattern
- Error handling (always return 200)

**`references/common-gotchas.md`**
- button_to + Turbo = CORS failure (needs `data: { turbo: false }`)
- Success redirect is backup, webhook is source of truth
- Sandbox vs Live Price IDs look similar but are different
- aws-sdk-rails v5 needs separate aws-actionmailer-ses gem
- current_period_start/end moved in API 2025-03-31
- Race condition between redirect and webhook (idempotent writes)

## Implementation Steps

1. Create SKILL.md with guided workflow
2. Create reference files from the golden spec content
3. Update marketplace.json
4. Test: walk through adding Stripe to a fresh Rails app
