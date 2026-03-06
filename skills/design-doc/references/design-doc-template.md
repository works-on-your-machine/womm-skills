# Design Doc Template

Use this template when writing a design doc. Every section should be filled in — if something is unknown, say so explicitly in Open Questions rather than leaving the section empty.

---

## Problem

What need does this solve? Why now? Who has this problem and how are they currently dealing with it?

**Good:** "Users currently export data as CSV and manually import it into their accounting software. This takes 20 minutes per week and frequently introduces errors in the currency conversion step."

**Bad:** "We need a better export feature."

The problem statement should make it obvious why this is worth building. If it doesn't, the idea might not be ready for a design doc yet.

## Solution

High-level approach. What are we building? One or two paragraphs, not implementation details. A reader should understand the shape of the solution without knowing the codebase.

## Architecture

How it fits into the existing system. Cover:
- Key components and how they interact
- Data flow (what gets created, read, updated, deleted)
- Integrations with external services
- Where new code lives relative to existing code

A diagram or flow description is helpful here. Keep it at the level where someone could sketch it on a whiteboard.

## Constraints

What we can't change. Be explicit about:
- Budget and timeline
- Existing dependencies we must work with
- Technical limits (hosting, language version, framework constraints)
- Organizational constraints (team size, review process, compliance)
- Things we're explicitly choosing not to build (and why)

## Implementation Approach

Rough ordering of work. What gets built first and why? This isn't a full epic breakdown — it's enough to see the general sequence and understand the reasoning behind it.

Consider:
- What has the most technical risk? (Build that early to learn fast)
- What unblocks other work? (Build dependencies first)
- What's the smallest useful slice? (Ship value incrementally)

## Open Questions

Things we don't know yet that could change the approach. Each question should include:
- Why it matters (what decision does the answer affect?)
- Who might know the answer or how to find out
- What we'd do if we had to decide right now

This section should shrink over time as questions get answered. When it's empty, the design is ready for epic breakdown.

## Related Golden Specs

Link to any golden specs at `~/.womm-skills/golden-specs/` that are relevant to this work. If none exist yet, note any areas where a golden spec would be useful after implementation.

---

## When to Stop

A design doc is ready for `/epic-breakdown` when:
- The Problem and Solution sections are clear and agreed upon
- Architecture covers enough detail to identify the major pieces of work
- Open Questions don't contain anything that would change the overall approach
- Constraints are documented so the epic breakdown can respect them

A design doc is NOT a full implementation spec. Don't try to specify every API endpoint, database column, or UI element. That level of detail belongs in `/implementation-plan`.
