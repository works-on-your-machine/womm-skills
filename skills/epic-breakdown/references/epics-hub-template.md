# Epics Hub Template

Use this format for the `epics.md` tracking hub.

---

```markdown
# [Project Name] — Epics

## Status Key
- **Done** — Shipped and working
- **In Progress** — Currently being built
- **Ready** — Spec'd and ready to implement
- **Backlog** — Planned, not yet spec'd

---

## Phase 1: [Phase Name]
[One sentence describing what this phase accomplishes.]

| Epic | Status | Depends On | File |
|------|--------|------------|------|
| [Epic name] | Ready | Nothing | [docs/epics/01-epic-name.md](docs/epics/01-epic-name.md) |
| [Epic name] | Ready | [Other epic] | [docs/epics/01-epic-name.md](docs/epics/01-epic-name.md) |

## Phase 2: [Phase Name]
[One sentence describing what this phase accomplishes.]

| Epic | Status | Depends On | File |
|------|--------|------------|------|
| [Epic name] | Backlog | Phase 1 | [docs/epics/02-epic-name.md](docs/epics/02-epic-name.md) |
```

---

## Guidelines

- **File naming:** `{phase number}-{epic-name}.md` (e.g., `01-user-auth.md`, `02-billing.md`)
- **Status updates:** Update the hub table when a story or epic status changes. The hub is the single source of truth for project progress.
- **Phases** group related epics. Within a phase, order by dependency then by risk (lowest risk first).
- **Links** should be relative paths so they work in any git host.
- Keep the hub concise — details belong in the individual epic files.
