# Golden Spec Template

Use this template when writing a golden spec. Every section should be filled in with specific, actionable detail.

---

```markdown
# [Topic Name]

## Problem

What were you trying to do? What made it hard? Be specific about the context — what stack, what scale, what constraints.

## Why This Approach

What alternatives exist? Why is this one better for this context?

| Approach | Pros | Cons |
|----------|------|------|
| [This approach] | [Why it wins] | [What you give up] |
| [Alternative A] | [Its strengths] | [Why you didn't choose it] |
| [Alternative B] | [Its strengths] | [Why you didn't choose it] |

## Architecture

How the pieces fit together. Include a flow description or diagram:

```
[Component A] → [Component B] → [Component C]
                      ↓
              [Component D]
```

Explain why the architecture looks this way — what constraints shaped it.

## Implementation

Complete, copy-paste-ready code and configuration. Include file paths.

### [First file or component]

**`path/to/file.ext`**
```language
[Complete file contents]
```

### [Second file or component]

**`path/to/other_file.ext`**
```language
[Complete file contents]
```

### [Environment / Configuration]

**Environment variables:**
```
KEY=value  # What this does
```

**Configuration files:**
```yaml
# path/to/config.yml
[Complete config]
```

## Common Gotchas

### [Gotcha title]
**What happens:** [Specific symptom]
**Why:** [Root cause]
**Fix:** [Exact solution]

### [Gotcha title]
**What happens:** [Specific symptom]
**Why:** [Root cause]
**Fix:** [Exact solution]

## Cost / Tradeoffs

| Resource | Monthly Cost | Notes |
|----------|-------------|-------|
| [Resource] | $XX | [Context] |
| **Total** | **$XX** | |

Performance tradeoffs:
- [Tradeoff and its impact]

Maintenance burden:
- [What needs ongoing attention]

## Reference Implementations

- [Project name](link) — [Brief description of how it uses this pattern]
```

---

## Quality Checklist

Before finalizing a golden spec, verify:

- [ ] **Self-contained** — Could someone follow this without any other context?
- [ ] **Gotchas are specific** — Each one describes a symptom, cause, and fix (not "be careful with X")
- [ ] **Code is complete** — No `...`, no "add your config here", no abbreviated files
- [ ] **Costs are numbers** — Actual dollar amounts or ranges, not "it depends"
- [ ] **Alternatives are compared** — Reader understands why this approach over others
- [ ] **Architecture is visual** — Flow description or diagram, not just prose
- [ ] **No project-specific identifiers** — Real project names, domains, bucket names, and local paths replaced with generic placeholders (`myapp`, `example.com`, `{project}-{resource}`)
