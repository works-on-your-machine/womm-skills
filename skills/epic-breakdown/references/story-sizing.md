# Story Sizing Guide

Stories should be sized for a single AI coding session: 15-30 minutes, fits in one context window.

## Right-Sized Story

A story is the right size when:
- It has a single clear goal
- It touches a small number of files (1-5)
- You can describe what "done" looks like in 2-3 acceptance criteria
- An AI agent can pick it up cold with just the story description and the codebase

## Too Big

Signs a story needs splitting:
- It touches multiple concerns (API + UI + database migration in one story)
- You need multiple paragraphs to describe what to do
- The acceptance criteria list keeps growing (5+ items)
- You find yourself writing "and then also..."
- It would require holding the context of many files simultaneously

**How to split:** Find the natural boundary. Usually it's the first thing that has to be true before the rest can happen. That's story A. Everything that depends on it is story B.

## Too Small

Signs stories should be merged:
- The story is "create a file" with no logic
- It takes longer to describe than to do
- Multiple stories are always done together and there's no reason to commit between them

**How to merge:** Combine stories that are always done in sequence and touch the same files.

## Writing Context for AI Agents

Each story should be self-contained enough that an AI agent can pick it up cold. Include:

- **What files to read first** — don't make the agent guess where to look
- **What the goal is** — concrete outcome, not vague direction
- **How to verify** — specific acceptance criteria the agent can check
- **What NOT to do** — if there are tempting but wrong approaches, call them out

**Good:** "Create `app/services/stripe_webhook_handler.rb`. Read `app/controllers/webhooks_controller.rb` for the existing pattern. Handle `checkout.session.completed` by finding the User via `client_reference_id` and creating a Subscription record. Test with a fixture webhook payload."

**Bad:** "Handle Stripe webhooks."
