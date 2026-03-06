# Epic Template

Use this format for each individual epic file.

---

```markdown
# Epic: [Epic Name]

**Status:** Ready
**Phase:** [Phase number and name]
**Depends on:** [List of epic names or "Nothing"]
**Blocks:** [List of epic names or "Nothing"]

## Goal

One or two sentences. What does this epic accomplish? What's different when it's done?

## Scope

**In scope:**
- [Specific thing this epic covers]
- [Another specific thing]

**Out of scope:**
- [Thing that might seem related but isn't part of this epic]
- [Thing deferred to a later epic]

## Stories

### Story 1.1: [Story Name]

**Description:** What to do and why.

**Inputs:**
- [Files to read or context needed]

**Outputs:**
- [Files created or modified]
- [State changes]

**Acceptance criteria:**
- [ ] [Specific, verifiable condition]
- [ ] [Another condition]
- [ ] Tests pass

**Dependencies:** None

---

### Story 1.2: [Story Name]

**Description:** What to do and why.

**Inputs:**
- [Files to read or context needed]

**Outputs:**
- [Files created or modified]

**Acceptance criteria:**
- [ ] [Specific, verifiable condition]
- [ ] Tests pass

**Dependencies:** Story 1.1

---

## Implementation Order

List the stories in the order they should be built, with brief rationale:

1. **Story 1.1** — [Why this goes first]
2. **Story 1.2** — [Why this goes second]
```

---

## Guidelines

- **Goal** should make the value obvious. "Set up the database" is vague. "Create the subscription data model so the billing API has somewhere to store state" is clear.
- **Scope** prevents scope creep. Be explicit about what's out — especially things that are tempting to include.
- **Stories** are the atomic unit of work. Each one should be completable in 15-30 minutes of AI-assisted work.
- **Acceptance criteria** use checkboxes so the `/implement` skill can mark them done as it works through stories.
- **Implementation order** may differ from story numbering if dependencies or risk levels suggest a different sequence.
