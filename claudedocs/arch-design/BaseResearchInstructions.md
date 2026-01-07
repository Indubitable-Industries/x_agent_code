# ╔══════════════════════════════════════════════════════════════════╗
# ║               BASE RESEARCH INSTRUCTIONS                         ║
# ║                     Agent Tool Usage Guide                       ║
# ╚══════════════════════════════════════════════════════════════════╝

---

## Purpose

**The purpose of this guide is to instruct on tool use: what is used for what, when, and how.**

Agent is allowed tool use for research but is required to use tools in very specific ways.

---

## Tool Reference

### Tavily

**What**: Online searches for technical and other topics, very well formatted for agent consumption.

**When**:
- External documentation lookup
- Current events / recent releases
- General technical research
- API documentation not in Context7

**How**: Direct queries, results are clean and digestible.

---

### Context7

**What**: System tools, commercial applications, and professional software documentation. An online man page built to be digestible.

**When**:
- Library/framework documentation
- CLI tool references
- Commercial software APIs
- Professional tooling guides

**How**: Query with library name + specific topic.

---

### Morph WarpGrep

**What**: Code context search with semantic understanding. Can consume significant context window.

**When**:
- Unsure where to start looking in codebase
- Need semantic understanding of code relationships
- Exploring unfamiliar code structure

**Constraints**:
- **Run at most 3x Morph searches per research cycle**
- **Immediately document findings in plan file after searches**
- Can occasionally produce incorrect results - watch diff output carefully

**How**: Targeted semantic queries about code behavior/location.

---

### Morph Fast Replace

**What**: High-speed find/replace workflow for code modifications.

**When**:
- Changing functions, lines, blocks
- Bulk replacements across files
- Refactoring with known patterns

**How**: Provide clear before/after patterns with sufficient context.

---

### Serena Memories

**What**: Critical cross-session reference system. Stores important notes about structure, usability, and change history.

**When**:
- **FREQUENTLY** - this is a primary tool
- When Morph won't get you there faster than a memory reference
- Storing architectural decisions
- Recording change history
- Documenting structural patterns

**How**:
- Short blurbs referencing larger documents
- Memories managed under `.serena/` directory
- Interaction is largely user-driven
- Check existing memories before deep code searches

---

### Serena Find/Replace

**What**: Symbol-aware find/replace. Works only on symbols as recognized by Serena.

**When**:
- Renaming symbols (functions, classes, variables)
- Refactoring with symbol awareness
- Changes requiring LSP-level understanding

**Accuracy Note**:
> Serena is FAR more accurate in replacements than Morph (0 failures vs ~dozen over extended use)

**How**: Reference symbols by name path, provide replacement.

---

### Sequential Thinking

**What**: The crowning jewel of agent problem-solving workflow. Combats circular reasoning, hallucination potential, and misunderstandings.

**When**:
| Symptom | Action |
|---------|--------|
| Model being "brief" | Consider clearing session |
| Model being obtuse | Time for Sequential Reasoning |
| Complex technical problem | Sequential with appropriate depth |

**Depth Guide**:

| Thoughts | Use Case |
|----------|----------|
| 3-5 | Minimal - simple clarifications |
| 10-15 | Medium-weight technical issues, moderate integrations |
| 20-30 | Large research synthesis, advanced implementations (e.g., refactoring core components) |

**How**: Invoke with thought count matching problem complexity.

---

## Workflow Requirements

### After Every Tool Use

1. **Document results** → Add to current plan file "Research" section
2. **Progress tracking** → Check off completed TODOWRITE items
3. **No TODOWRITE?** → Create one per `TODOWRITECREATION` guide in `claudedocs/arch-design/`

### Research Flow

```
Tool Use → Document in Plan → Update TODOWRITE → Next Tool
    ↑                                              ↓
    └──────────── Repeat until complete ───────────┘
```

---

## Tool Selection Matrix

| Need | First Choice | Fallback |
|------|--------------|----------|
| External docs | Tavily | Context7 |
| Library docs | Context7 | Tavily |
| Code location | Serena Memory | Morph WarpGrep |
| Code understanding | Morph WarpGrep | Serena symbols |
| Symbol refactor | Serena Find/Replace | Morph Fast Replace |
| Bulk text replace | Morph Fast Replace | Serena Find/Replace |
| Complex reasoning | Sequential Thinking | - |

---

## Anti-Patterns

- Running >3 Morph searches without documenting findings
- Skipping TODOWRITE updates
- Using Morph when a Serena memory would be faster
- Under-scoping Sequential Thinking depth for complex problems
- Not watching Morph diff output for errors

---

*See also: `TODOWRITECREATION.md` in this directory*
