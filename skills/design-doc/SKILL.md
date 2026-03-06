---
name: design-doc
description: "When the user wants to create a structured design document from a raw idea, conversation, or rough notes. Also use when the user mentions 'design doc', 'spec out', 'architecture doc', 'write up the design', 'plan this out', 'brainstorm this', or 'help me think through this'. Includes a deep interview phase before writing. For breaking a design doc into epics, see epic-breakdown. For capturing solved problems, see golden-spec-create."
metadata:
  version: 1.0.0
---

# Design Doc

Create a structured design document from a raw idea, conversation, or rough notes. This is the first step in the build workflow — turning vague intent into something concrete enough to break into epics.

## Before Starting

1. **Check golden specs access.** Try to list files in `~/.womm-skills/golden-specs/`. If not accessible, tell the user:
   - Add `~/.womm-skills/` to allowed paths in `~/.claude/settings.json`
   - Add a reference in `~/.claude/CLAUDE.md` pointing to the golden specs directory

2. **Read the project** to understand stack and conventions:
   - `CLAUDE.md` for project-specific instructions
   - `Gemfile`, `package.json`, `go.mod`, `Cargo.toml`, etc. for language and framework
   - Existing `docs/` directory for format conventions
   - Golden specs at `~/.womm-skills/golden-specs/` for context on past decisions

3. **Check for existing design docs** in the project (look in `docs/`, `docs/design/`, etc.). If found, match their format and level of detail.

## Process

### 1. Get the Idea

Ask the user to describe what they want to build — or read a conversation, notes, or rough spec they provide.

### 2. Deep Interview

Interview the user before writing anything. Use the following areas as a guide, but adapt to the specific idea. Questions should be specific and non-obvious — don't ask things you can infer from the project files.

**Problem space:**
- What specific problem does this solve? Who has this problem?
- Why now? What changed that makes this worth building?
- What happens if we don't build this?

**Technical implementation:**
- How does this fit into the existing system?
- What are the key data models or state changes?
- What external services or integrations are involved?
- Are there concurrency, performance, or scale concerns?

**UI & UX** (if applicable):
- What are the primary user flows?
- What are the edge cases and error states?
- What does the user see when things go wrong?

**Concerns and tradeoffs:**
- What could go wrong?
- What are we explicitly choosing not to build?
- What's the simplest version that would be useful?

**Constraints:**
- Budget, timeline, team size?
- Existing systems we can't change?
- Technical limits (hosting, language, framework)?

Keep interviewing until you've covered all major areas. Don't stop after 3-4 questions. Synthesize answers as you go to ask better follow-up questions.

### 3. Draft

Write the design doc using the template in `references/design-doc-template.md`. Fill every section based on what you learned from the interview and the project files.

### 4. Review

Present the draft to the user. Iterate based on feedback until they're satisfied.

### 5. Write

Save the final doc to the appropriate location:
- Respect the project's existing docs structure
- If no convention exists, write to `docs/`
- Filename: descriptive, lowercase, hyphens (e.g., `user-authentication.md`)

## Cross-References

- After writing: suggest `/epic-breakdown` to break the design doc into implementable work
- If the user describes a solved problem instead of a new idea: redirect to `/golden-spec-create`
