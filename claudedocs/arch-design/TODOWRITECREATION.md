# ╔══════════════════════════════════════════════════════════════════╗
# ║                    TODOWRITE CREATION GUIDE                      ║
# ║                  Task Management for Agent Workflows             ║
# ╚══════════════════════════════════════════════════════════════════╝

---

## Purpose

Guide for when and how agents should create and manage TODOWRITE task lists to maintain progress visibility and session continuity.

---

## When to Create a TODOWRITE

**Required**:
- Any research task with >2 steps
- Before starting any plan file work
- When picking up an existing plan mid-session
- After receiving multi-part instructions
- Beginning any implementation work

**Optional but Recommended**:
- Complex single-step tasks (safety net)
- When context window is getting full (tracking prevents re-work)

---

## Task Granularity

| Task Type | Granularity | Example |
|-----------|-------------|---------|
| Research | Per-source or per-question | "Search Tavily for MCP patterns" |
| Implementation | Per-function or per-file | "Implement spawn_subagent()" |
| Documentation | Per-section | "Write Architecture section" |
| Investigation | Per-hypothesis | "Test if Codex blocks subprocess" |

### Breakdown Rule

> If a task takes >15 min or spans multiple tools, break it down further

### Max Task Count

> **10 tasks maximum per TODOWRITE**
>
> If you need >10 tasks, create sub-plans in `claudedocs/plans/` and reference them:
> ```
> "Execute sub-plan: 20260106_codex_sandbox_investigation.md"
> ```

---

## Naming Convention

```yaml
content: Verb + Object + Context
activeForm: Verbing + Object + Context
```

| Quality | content | activeForm |
|---------|---------|------------|
| Good | "Research Codex sandbox restrictions" | "Researching Codex sandbox restrictions" |
| Good | "Implement context sharing protocol" | "Implementing context sharing protocol" |
| Bad | "Codex stuff" | "Doing Codex stuff" |
| Bad | "Do the thing" | "Doing the thing" |

**Verbs to use**: Research, Implement, Document, Test, Investigate, Create, Update, Review, Fix

---

## Status Flow

```
pending ──────→ in_progress ──────→ completed
                     │
                     ↓
               (blocked)
                     │
                     ↓
         Document in issues/
                     │
                     ↓
         Resume when unblocked
```

### Status Rules

| Rule | Requirement |
|------|-------------|
| Single Active | Only ONE task `in_progress` at a time |
| Immediate Completion | Mark complete **immediately** after finishing, not batched |
| No Skipping | Don't mark complete if partially done |

---

## Task Dependencies

When tasks have dependencies, document them explicitly:

### In TODOWRITE Content

```yaml
content: "Implement protocol handler (blocked by: Define protocol spec)"
```

### Dependency Patterns

| Pattern | Notation |
|---------|----------|
| Hard block | `(blocked by: Task Name)` |
| Soft dependency | `(after: Task Name)` |
| Parallel OK | No notation needed |

### Dependency Example

```
1. "Define message protocol spec" - in_progress
2. "Implement protocol handler (blocked by: #1)" - pending
3. "Research existing MCP patterns" - pending (can run parallel to #1)
4. "Write protocol documentation (after: #1, #2)" - pending
```

---

## Plan File Integration

TODOWRITE tasks map to specific plan file sections:

### Plan File Section → Task Type

| Plan Section | TODOWRITE Tasks |
|--------------|-----------------|
| `## Research Objectives` | Research tasks |
| `## Research Phases` | Phase-level tasks |
| `## Implementation` | Implementation tasks |
| `## Outstanding Questions` | Investigation tasks |

### Sync Requirements

1. **Plan checkboxes** must stay in sync with TODOWRITE status
2. **Findings** go into plan's Research section immediately after tool use
3. **Blockers** get documented in both TODOWRITE and `claudedocs/issues/`

### Example Sync

**Plan file** (`20260106_cross_cli_agent_control.md`):
```markdown
## Research Phases
### Phase 1: Landscape Survey
- [x] Catalog existing cross-agent projects
- [ ] Document Codex sandbox restrictions  ← current
- [ ] Document OpenCode capabilities
```

**TODOWRITE**:
```yaml
- content: "Catalog existing cross-agent projects"
  status: completed
- content: "Document Codex sandbox restrictions"
  status: in_progress  # ← matches plan
- content: "Document OpenCode capabilities"
  status: pending
```

---

## Session Handoff

When ending a session with incomplete work:

### Checklist

1. **TODOWRITE current** - Reflects actual state (not stale)
2. **Serena memory** - Summary of progress and blockers
3. **Plan file** - Next action noted in relevant section
4. **Issues documented** - Any blockers in `claudedocs/issues/`

### Handoff Memory Template

```markdown
## Session Handoff: [Date]

**Completed**: [List completed tasks]
**In Progress**: [Current task and state]
**Blocked**: [Blockers and issue references]
**Next**: [Immediate next action]
**Plan File**: [Which plan, which section]
```

---

## Anti-Patterns

| Anti-Pattern | Correct Behavior |
|--------------|------------------|
| >10 tasks in one TODOWRITE | Break into sub-plans |
| Multiple `in_progress` | Only one active at a time |
| Batching completions | Mark complete immediately |
| Vague task names | Verb + Object + Context |
| Ignoring dependencies | Document with `(blocked by:)` |
| Stale TODOWRITE | Update before every tool switch |
| No session handoff | Always document state before ending |

---

## Quick Reference

```
CREATE: >2 steps, any plan work, multi-part instructions
GRANULARITY: 15 min or multi-tool = break down
MAX: 10 tasks, then sub-plan
NAMING: Verb + Object + Context
STATUS: One in_progress, immediate completion
DEPS: (blocked by: X) or (after: X)
SYNC: Plan checkboxes ↔ TODOWRITE status
HANDOFF: TODOWRITE + Memory + Plan note + Issues
```

---

*See also: `BaseResearchInstructions.md` for tool usage during task execution*
