# L1a: Deep-Dive on Key Projects

**Task ID**: L1a (Follow-up to L1)
**Started**: 2026-01-08
**Status**: COMPLETE

---

## Overview

Two projects from the L1 GitHub survey require detailed analysis before x_agent_code implementation:

1. **steipete/claude-code-mcp** - Claude Code as one-shot MCP server
2. **Kevinjulu/oh-my-opencode-CLI** - OpenCode plugin with Claude compatibility

Both directly relate to our hub-spoke architecture design.

---

## Project 1: claude-code-mcp

**Repository**: https://github.com/steipete/claude-code-mcp
**Stars**: 992 | **Forks**: 110 | **License**: MIT

### What It Does

Exposes Claude Code as an MCP server, enabling other AI agents to invoke Claude Code in **one-shot mode** with permissions bypassed. This creates an "agent in your agent" pattern.

### Architecture

```
┌─────────────────────┐
│  Host Agent         │  (Cursor, Windsurf, etc.)
│  (any model)        │
└─────────┬───────────┘
          │ MCP Protocol
          ▼
┌─────────────────────┐
│  claude-code-mcp    │  MCP Server
│  (Node.js)          │
└─────────┬───────────┘
          │ CLI invocation
          ▼
┌─────────────────────┐
│  Claude Code CLI    │  --dangerously-skip-permissions
│  (Claude model)     │
└─────────────────────┘
```

### Single Tool Interface

The server exposes **one tool**: `claude_code`

```json
{
  "prompt": "Create a new React component",
  "options": {
    "tools": ["Bash", "Read", "Write", "Edit"]
  }
}
```

### Permission Bypass Mechanism

**Critical insight**: Claude Code supports `--dangerously-skip-permissions` flag.

1. User runs `claude --dangerously-skip-permissions` once manually
2. Accepts terms interactively (one-time)
3. MCP server can then use the flag non-interactively
4. All subsequent invocations skip permission prompts

**Implication for x_agent_code**: We can invoke Claude Code non-interactively as a subprocess using this same flag.

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Cost Offloading** | Use cheaper models (Gemini, o3) as orchestrators, delegate to Claude Code |
| **Context Preservation** | Queued commands reduce context compaction, preserving important info |
| **File Operations** | Claude Code excels at file editing vs other tools |
| **System Access** | Gains capabilities editors can't provide |

### Usage Patterns

**File Creation** (must include working directory):
```
"Your work folder is /Users/user/project\n\nCreate config.yml with database settings"
```

**Git Workflows**:
```
"Stage all changes, commit with message 'Fix auth bug', and push to origin"
```

**Complex Releases**:
```
"Bump version to 2.0.0, update CHANGELOG.md, create git tag, push with tags"
```

### Configuration

```json
{
  "claude-code-mcp": {
    "command": "npx",
    "args": ["-y", "@steipete/claude-code-mcp@latest"],
    "env": {
      "CLAUDE_CLI_NAME": "claude",
      "MCP_CLAUDE_DEBUG": "true"
    }
  }
}
```

### Relevance to x_agent_code

| Pattern | How We Use It |
|---------|---------------|
| One-shot execution | Our `ask_agent()` tool uses same pattern |
| Permission bypass | We'll use `--dangerously-skip-permissions` for non-interactive invocation |
| CWD context | We must pass working directory in prompts |
| Single tool design | Validates our simple tool interface approach |

**Key Difference**: claude-code-mcp is Claude→Claude. We're building Claude→GPT (via OpenCode).

---

## Project 2: oh-my-opencode-CLI

**Repository**: https://github.com/Kevinjulu/oh-my-opencode-CLI
**License**: Open Source

### What It Does

Enterprise-grade plugin system for OpenCode that enables:
- Multi-model orchestration with specialized agent teams
- Claude Code compatibility layer
- Async subagent execution
- Quality enforcement mechanisms

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    oh-my-opencode                        │
├─────────────────────────────────────────────────────────┤
│  Agent Teams                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │   Sisyphus   │ │    Oracle    │ │   Frontend   │     │
│  │ Claude Opus  │ │   GPT 5.2    │ │  Gemini 3    │     │
│  │  (main dev)  │ │  (debug)     │ │   (UI/UX)    │     │
│  └──────────────┘ └──────────────┘ └──────────────┘     │
│                                                          │
│  ┌──────────────┐ ┌──────────────┐                      │
│  │  Librarian   │ │   Explore    │                      │
│  │Claude Sonnet │ │ Context grep │                      │
│  │   (docs)     │ │  (search)    │                      │
│  └──────────────┘ └──────────────┘                      │
├─────────────────────────────────────────────────────────┤
│  Claude Code Compatibility Layer                         │
│  - Commands, Agents, Skills, MCPs                        │
│  - Hooks: PreToolUse, PostToolUse, UserPromptSubmit     │
│  - Subagent context inheritance                          │
├─────────────────────────────────────────────────────────┤
│  Enforcement Mechanisms                                  │
│  - Todo Continuation Enforcer                            │
│  - Comment Checker (code quality)                        │
├─────────────────────────────────────────────────────────┤
│  OpenCode Core                                           │
└─────────────────────────────────────────────────────────┘
```

### Claude Code Compatibility Layer

**Full feature parity with Claude Code**:
- Commands (slash commands)
- Agents (subagent spawning)
- Skills (reusable workflows)
- MCPs (Model Context Protocol servers)
- Hooks system for lifecycle events

**Hook Events**:
| Hook | When Fired |
|------|------------|
| `PreToolUse` | Before any tool execution |
| `PostToolUse` | After tool completion |
| `UserPromptSubmit` | When user sends message |
| `Stop` | When agent completes |

### Async Subagents Implementation

**Parallel Execution Pattern**:
```
Main Agent (Sisyphus)
    │
    ├──▶ Spawn Frontend Agent ──▶ [working on UI...]
    │         async
    │
    ├──▶ Spawn Backend Agent ──▶ [working on API...]
    │         async
    │
    └──▶ Continue own work...

    [All agents complete, results merged]
```

**Benefits**:
- Reduced token consumption (distributed context)
- Parallel work on different parts
- Specialized models for specialized tasks
- No context bloat in main agent

### Multi-Model Orchestration

**Model Assignment by Task**:
| Agent | Model | Task Type |
|-------|-------|-----------|
| Sisyphus | Claude Opus 4.5 | Core development, complex logic |
| Oracle | GPT 5.2 | Design review, debugging |
| Frontend | Gemini 3 Pro | UI/UX, interface code |
| Librarian | Claude Sonnet 4.5 | Documentation, exploration |

### Curated MCP Integrations

| MCP | Purpose |
|-----|---------|
| Exa | Web search |
| Context7 | Official documentation |
| Grep.app | GitHub code search |
| LSP/AST | Language Server Protocol, AstGrep |

### Enforcement Mechanisms

**Todo Continuation Enforcer**:
- Prevents agents from prematurely stopping
- Forces completion of all todo items
- "It doesn't stop until the job is done"

**Comment Checker**:
- Validates generated code quality
- Ensures human-indistinguishable output
- Catches placeholder/stub code

### Installation

```bash
bunx oh-my-opencode install

# With provider flags:
bunx oh-my-opencode install \
  --claude=yes \
  --chatgpt=yes \
  --gemini=yes
```

### Relevance to x_agent_code

| Feature | How We Use It |
|---------|---------------|
| Claude compat layer | Model for our middleware design |
| Async subagents | Pattern for our invisible subprocess model |
| Hooks system | Consider for our observability layer |
| Multi-model orchestration | Validates our hub-spoke with multiple spokes |
| Enforcement mechanisms | "Continue until done" pattern for our agents |

**Key Insight**: oh-my-opencode already bridges OpenCode to Claude patterns. We could potentially:
1. Use it directly as our OpenCode spoke
2. Learn from its architecture for our own implementation
3. Contribute to it rather than build from scratch

---

## Comparative Analysis

| Aspect | claude-code-mcp | oh-my-opencode |
|--------|-----------------|----------------|
| **Direction** | Other agents → Claude Code | OpenCode → Claude-like behavior |
| **Architecture** | Single MCP tool | Full plugin system |
| **Model Support** | Claude only | Multi-model (Claude, GPT, Gemini) |
| **Complexity** | Simple, focused | Feature-rich, enterprise |
| **Our Use Case** | Hub pattern model | Spoke implementation candidate |

---

## Patterns to Adopt

### From claude-code-mcp

1. **One-shot execution with permission bypass**
   ```bash
   claude --dangerously-skip-permissions -p "task description"
   ```

2. **CWD context injection**
   ```
   "Your work folder is {cwd}\n\n{actual_prompt}"
   ```

3. **Single tool interface**
   - Keep it simple: one `ask_agent()` tool
   - Let the invoked agent handle complexity

### From oh-my-opencode

1. **Async subagent pattern**
   - Spawn, don't wait
   - Collect results later
   - Merge context intelligently

2. **Hooks for observability**
   - PreToolUse / PostToolUse for logging
   - UserPromptSubmit for request tracking

3. **Enforcement mechanisms**
   - Todo continuation: don't let agents quit early
   - Quality checks: validate output before accepting

4. **Model-task matching**
   - Different models for different tasks
   - Specialized agents for specialized work

---

## Strategic Recommendations

### Option A: Build on Existing

**Use oh-my-opencode as our OpenCode spoke**:
- Already has Claude compatibility
- Multi-model support
- Active community
- Reduces our build effort

**Risk**: Dependency on external project, may not fit our exact needs.

### Option B: Learn and Build

**Build our own middleware, informed by these patterns**:
- Full control over architecture
- Tailored to our interaction modes
- Can contribute patterns back to community

**Risk**: More development effort, reinventing some wheels.

### Option C: Hybrid

**Use claude-code-mcp patterns for hub, build custom spoke integration**:
- Adopt permission bypass pattern
- Adopt one-shot execution pattern
- Build our own interaction mode layer
- Use OpenCode directly (not oh-my-opencode)

**Recommendation**: **Option C** - We want our unique interaction modes (challenge/agree/collaborate/deduce) which neither project provides. Use their patterns, not their code.

---

## Next Steps

1. Test claude-code-mcp locally to understand one-shot behavior
2. Test OpenCode's native HTTP API without oh-my-opencode
3. Design our middleware combining:
   - Permission bypass (claude-code-mcp)
   - Async execution (oh-my-opencode)
   - Interaction modes (our novel contribution)
4. Proceed to implementation phase
