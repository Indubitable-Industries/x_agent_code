# ╔══════════════════════════════════════════════════════════════════╗
# ║                 TROUBLESHOOTING STEP GUIDE                       ║
# ║              Systematic Problem Investigation Workflow           ║
# ╚══════════════════════════════════════════════════════════════════╝

---

## Purpose

Structured approach for investigating and documenting issues. Prevents context loss through disciplined information capture during complex troubleshooting.

**Core Principle**: Store information as you gather and process it. This prevents context loss from compacting.

---

## Pre-Flight Checklist

**Before any troubleshooting work, review these documents in order:**

1. `BaseResearchInstructions.md` - Tool usage fundamentals
2. `TROUBLESHOOTING_STEP_GUIDE.md` - This document (investigation workflow)
3. `ISSUE_FIXING_STEP_GUIDE.md` - Implementation workflow (separate phase)

---

## Phase 0: Setup

### Create Plan File

Create a plan file in `claudedocs/plans/` with format:
```
YYYYMMDD_<issue_description>.md
```

**Required sections**:
- Problem Statement
- Research (tool outputs go here)
- Conclusions (synthesis goes here)
- Status

> **STOP POINT #1**: This is one of only two places to ask the user clarifying questions. Ask now if anything is unclear about the problem.

### Create TODOWRITE

Break down investigation into granular chunks. **More granular = better**.

```yaml
- "Sequential 5: Initial problem understanding"
- "Identify log file locations"
- "Review recent log entries"
- "Tavily: Search for similar issues"
- "Context7: Check library documentation"
- "Morph: Search codebase for error source"
# ... etc
```

---

## Phase 1: Initial Understanding

### Step 1: Sequential Thinking (5 thoughts minimum)

Before any tool use, run Sequential Thinking to frame the problem.

**Output**: Update plan file with base understanding.

### Step 2: Log Investigation

1. **Identify** log file locations
2. **Document** locations in plan file (for future reference)
3. **Review** relevant log entries
4. **Extract** error messages, timestamps, patterns

---

## Phase 2: Research

### Tool Concurrency Rules

| Tool | Max Concurrent | Dump Location |
|------|----------------|---------------|
| Morph WarpGrep | 2 | Plan → Research section |
| Tavily | 3 | Plan → Research section |
| Context7 | 3 | Plan → Research section |

### Tool Selection by Need

| Need | First Choice | Then |
|------|--------------|------|
| Local codebase | Morph WarpGrep | - |
| Online/general | Tavily | - |
| CLI/OS/Commercial | Context7 | Tavily |
| Known libraries | Context7 | Tavily |

### Research Workflow

```
For each TODOWRITE research item:
    1. Run tool (respect concurrency limits)
    2. IMMEDIATELY dump results to plan file Research section
    3. Mark TODOWRITE item complete
    4. Continue to next item
```

**Critical Rule**: Complete ALL research before any "investigation" or code trace steps. Code trace when in doubt.

---

## Phase 3: Documentation Standards

### Append-Only Updates

> Never remove lines when updating. Mark outdated sections instead:
> ```
> [OUTDATED - 2026-01-06: See Conclusions section]
> ```

### Documentation by Phase

| Phase | Standard |
|-------|----------|
| **Pre-retrieval** | Be explicit about issues, plans, methods. Expand FULLY on what you have. DO NOT INVENT anything not discussed. |
| **Post-retrieval** | Enter information EXACTLY as seen. No paraphrasing. Include line references, code snippets. |
| **Implementation** | **STOP** - See below |

### What "Be Explicit" Means

- **Pre-retrieval**: Expand fully on what you know. Don't invent anything not yet discussed.
- **Post-retrieval**: Document exact meanings of items seen in returns. No paraphrasing - enter information exactly.
- **Code snippets**: If referencing code not in our codebase, provide limited previews in documents.
- **Line references**: Always include file paths and line numbers when available.

### Implementation Boundary

> **If you're about to add implementation code to a plan doc: STOP.**
>
> Plan docs are for:
> - Planning
> - Pseudocode (enough to convey intent)
>
> For implementation:
> 1. Create separate `YYYYMMDD_<topic>_implementation.md`
> 2. Cross-link both documents
> 3. Follow `ISSUE_FIXING_STEP_GUIDE.md`

---

## Phase 4: Synthesis

### Unlock Results

To process all research findings, analyze each tool's output individually first.

### Per-Tool Analysis

For EACH tool/MCP that returned data:

```
Sequential Thinking: 12 thoughts (fixed)
Input: That tool's output from Research section
Output: Save to Conclusions section
```

**Total**: 12 thoughts × N tools (where N = number of tools that returned data)

### Cross-Tool Synthesis

If multiple tools returned data, run additional synthesis:

```
Sequential Thinking: 18 thoughts (fixed)
Input: All individual tool conclusions from above
Output: Final married conclusions - unifying individually researched items
```

### Update Plan Status

After synthesis complete, update plan file status.

---

## Phase 5: Report to User

> **STOP POINT #2**: This is the second and final stop point.

**After synthesis, ALWAYS report findings to user and await decision.**

### Report Format

```markdown
## Synthesis Complete

**Status**: [FINDINGS_READY | STUCK, USER]

### What I Found
[Summary of conclusions from synthesis]

### Key Evidence
[Most important findings with references]

### Options (if applicable)
1. [Option A with trade-offs]
2. [Option B with trade-offs]

### Recommendation (if clear)
[Your suggested path, if one emerges]

**Awaiting your decision on how to proceed.**
```

### Decision Outcomes

| User Decision | Next Action |
|---------------|-------------|
| Proceed with fix | Create implementation plan per `ISSUE_FIXING_STEP_GUIDE.md` |
| Need more research | Return to Phase 2 with new TODOWRITE items |
| Different direction | Update plan file, restart as needed |
| Close issue | Mark plan `STATUS: RESOLVED` or `STATUS: WONTFIX` |

---

## Status Values

| Status | Meaning |
|--------|---------|
| `INVESTIGATING` | Active research in progress |
| `RESEARCH_COMPLETE` | All tools run, awaiting synthesis |
| `SYNTHESIZING` | Running sequential analysis |
| `FINDINGS_READY` | Synthesis complete, awaiting user decision |
| `STUCK, USER` | Conflicting/unclear findings, need user guidance |
| `APPROVED_FOR_IMPL` | User approved, ready for implementation planning |
| `RESOLVED` | Issue fixed and verified |
| `WONTFIX` | Intentionally not fixing |

---

## Quick Reference

```
SETUP:    Plan file + TODOWRITE (granular)
          → Ask user questions NOW if needed (STOP POINT #1)

INITIAL:  Sequential 5 → Logs → Document locations

RESEARCH: Tools (2-3 concurrent) → IMMEDIATE dump to Research
          Complete ALL research before proceeding
          Code trace when in doubt

DOCUMENT: Append-only, exact quotes, no implementation code
          Implementation → separate doc + cross-link

SYNTH:    12-step per tool (fixed) → 18-step cross-tool (fixed)
          Save all to Conclusions section

REPORT:   Present findings to user (STOP POINT #2)
          → User decides next steps
          → NEVER proceed to implementation without user approval
```

---

*See also: `BaseResearchInstructions.md`, `ISSUE_FIXING_STEP_GUIDE.md`*
