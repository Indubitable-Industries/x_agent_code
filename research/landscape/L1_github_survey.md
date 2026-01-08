# L1: GitHub Landscape Survey

**Task ID**: L1
**Started**: 2026-01-08
**Status**: COMPLETE

---

## Research Questions

1. Is anyone already doing what we're building?
2. Are there patterns we should adopt?
3. Are there abandoned projects with post-mortems?
4. What's the state of MCP multi-agent ecosystem?
5. Should we pivot, integrate, or continue as planned?

---

## Raw Results - Tavily

### Search: "MCP multi-agent orchestration GitHub"

**lastmile-ai/mcp-agent** - https://github.com/lastmile-ai/mcp-agent
- "Build effective agents using Model Context Protocol and simple workflow patterns"
- Tools from any MCP server plug into agents
- Multi-agent handoffs compatible with OpenAI Swarm
- Temporal execution engine for pause/resume, retries, human input

**rinadelph/Agent-MCP** - https://github.com/rinadelph/Agent-MCP
- Framework for multi-agent systems via MCP
- Multiple specialized agents working in parallel
- For experienced AI developers, requires MCP protocol familiarity

**ruvnet/claude-flow** - https://github.com/ruvnet/claude-flow
- "Leading agent orchestration platform for Claude"
- 87 specialized MCP tools for AI orchestration
- Multi-agent swarms, autonomous workflows
- Enterprise-grade architecture

**evalstate/fast-agent** - https://github.com/evalstate/fast-agent
- Create multimodal Agents and Workflows in minutes
- Complete MCP feature support including Sampling and Elicitations
- Routing, parallelization, orchestrator-workers patterns

**kyegomez/swarms** - https://github.com/kyegomez/swarms
- Enterprise-Grade Multi-Agent Orchestration Framework
- Production-scale deployments
- MCP server integration for dynamic tool discovery

**agentscope-ai/agentscope** - https://github.com/agentscope-ai/agentscope
- Multi-Agent Oriented programming
- Explicit message passing and workflow orchestration

### Search: "Claude Code extension multi-agent"

**steipete/claude-code-mcp** - https://github.com/steipete/claude-code-mcp
- Claude Code as one-shot MCP server
- "Agent in your agent" pattern
- Handles complex multi-step tasks (branch, edit, commit, PR)
- Permissions bypassed automatically

**grahama1970/claude-code-mcp-enhanced** - https://github.com/grahama1970/claude-code-mcp-enhanced
- Enhanced Claude Code MCP server with orchestration
- "Boomerang pattern" for breaking complex tasks into subtasks
- Robust error handling

**zilliztech/claude-context** - https://github.com/zilliztech/claude-context
- Semantic code search MCP for Claude Code
- Entire codebase as context
- No multi-round discovery needed

**Official: Claude Code Plugins** (Anthropic)
- Slash commands, subagents, MCP servers, hooks
- Single command installation
- June 2025: Remote MCP Server Support, GitHub Actions Integration

### Search: "OpenCode plugin integration"

**opencode-ai/opencode** - https://github.com/opencode-ai/opencode
- Go-based CLI with TUI
- Plugin system via JS/TS files or npm packages
- MCP server management built-in
- GitHub Actions integration

**Kevinjulu/oh-my-opencode-CLI** - https://github.com/Kevinjulu/oh-my-opencode-CLI
- "#1 OpenCode Plugin"
- ASYNC SUBAGENTS (like Claude Code)
- LSP/AST tools included
- **Claude Code Compatible Layer**
- Curated MCPs

**awesome-opencode/awesome-opencode** - https://github.com/awesome-opencode/awesome-opencode
- Curated list of plugins, themes, agents
- Official SDKs: JavaScript/TypeScript, Go, Python

### Search: "AI agent middleware CLI orchestration"

**crewAIInc/crewAI** - https://github.com/crewAIInc/crewAI
- Role-playing, autonomous AI agents framework
- Collaborative intelligence, complex tasks
- 100,000+ developers certified
- Enterprise-ready AI automation

**microsoft/agent-framework** - https://github.com/microsoft/agent-framework
- Multi-language (Python, .NET)
- Flexible middleware system for request/response processing
- Graph-based workflows with streaming, checkpointing, human-in-the-loop

**basnijholt/agent-cli** - https://github.com/basnijholt/agent-cli
- Middleware between chat client and OpenAI-compatible LLM
- Markdown files for memory storage
- Built-in Git integration
- Proxy middleware for any /chat/completions endpoint

**MedChaouch/Puzld.ai** - https://github.com/MedChaouch/Puzld.ai
- Terminal-native multi-LLM orchestration
- "Agentic Mode" - like Claude Code but for any LLM
- File edits with permission prompts

**awslabs/agent-squad** - https://github.com/awslabs/agent-squad
- AWS's multi-agent orchestration
- Intent classification for dynamic query routing
- Python and TypeScript support
- Universal deployment (Lambda, local, any cloud)

**block/goose** - https://github.com/block/goose
- On-machine AI agent (Block/Square)
- Works with any LLM, multi-model configuration
- MCP server integration
- Desktop app and CLI

### Search: "cross-agent communication LLM tools"

**langroid/langroid** - https://github.com/langroid/langroid
- CMU/UW-Madison researchers
- Agents exchange messages collaboratively
- MCP Support via tool adapter
- Lightweight, extensible Python framework

**massgen/MassGen** - https://github.com/massgen/MassGen
- Multi-agent scaling system in terminal
- Autonomously orchestrates frontier models
- Agents collaborate and reason together
- Presented at Berkeley Agentic AI Summit 2025

**X-PLUG/Multi-LLM-Agent** - https://github.com/X-PLUG/Multi-LLM-Agent
- α-UMi: Multi-LLM collaborated agent
- Decomposes LLM into planner, caller, summarizer
- Small LLMs collaborate to outperform large LLMs

**Research Papers (2025)**:
- "Augmenting Multi-Agent Communication with State Delta Trajectory" (June 2025)
- "Cache-to-Cache (C2C)" - direct semantic communication between LLMs using KV-cache
- NMAI (Native Multi-Agent Intelligence) blueprint for Foundation Models

---

## Raw Results - GitHub

*Projects discovered will be catalogued here with:*
- Repository name and URL
- Stars/forks (activity indicator)
- Description
- Relevance assessment

---

## Synthesis

### Categories Discovered

1. **MCP-native multi-agent frameworks**: mcp-agent, Agent-MCP, claude-flow, fast-agent
2. **Claude Code extensions**: claude-code-mcp, claude-code-mcp-enhanced, claude-context
3. **OpenCode ecosystem**: oh-my-opencode with Claude Code compatible layer
4. **General agent orchestration**: CrewAI, Microsoft Agent Framework, Swarms, agent-squad
5. **Cross-agent communication tools**: Langroid, MassGen, Multi-LLM-Agent

### Key Discovery

**oh-my-opencode-CLI** has a "Claude Code Compatible Layer" - directly relevant to our goal of bridging Claude Code and OpenCode.

### Most Relevant Projects for x_agent_code

| Project | Why It Matters |
|---------|----------------|
| steipete/claude-code-mcp | "Agent in your agent" - Claude Code as MCP server, one-shot execution |
| Kevinjulu/oh-my-opencode-CLI | Claude Code compatibility layer for OpenCode, async subagents |
| lastmile-ai/mcp-agent | Simple workflow patterns, Swarm-compatible handoffs, Temporal engine |
| ruvnet/claude-flow | Enterprise patterns, 87 MCP tools (reference architecture) |

### Patterns to Adopt

1. **From mcp-agent**: Temporal execution engine for pause/resume, Swarm-compatible handoffs
2. **From claude-code-mcp**: One-shot execution with permissions bypass, "Boomerang pattern" for subtasks
3. **From oh-my-opencode**: Claude Code compatible layer architecture, async subagents
4. **From Langroid**: Message exchange between agents, MCP tool adapter pattern
5. **From agent-cli**: Middleware proxy pattern, Markdown memory storage

---

## Conclusions

### Research Questions Answered

| Question | Answer |
|----------|--------|
| Is anyone already doing what we're building? | **PARTIALLY** - Pieces exist but no one combines them for specific Claude↔GPT bridging with interaction modes |
| Are there patterns we should adopt? | **YES** - One-shot MCP, compatibility layers, workflow engines, message exchange |
| Are there abandoned projects with post-mortems? | **NOT FOUND** - Space is young, most projects actively maintained |
| What's the state of MCP multi-agent ecosystem? | **THRIVING** - Multiple frameworks, active development, enterprise adoption |
| Should we pivot, integrate, or continue as planned? | **CONTINUE** with selective integration after deeper study |

### Our Unique Niche

What x_agent_code offers that doesn't exist:

1. **Specific CLI-to-CLI bridging** - Not framework-level, actual Claude Code ↔ OpenCode
2. **Interaction Modes** - challenge/agree/collaborate/deduce patterns (novel)
3. **User-facing simplicity** - "ask GPT" and it works, not a developer framework
4. **Cross-vendor focus** - Specifically bridges Claude ↔ OpenAI/GPT ecosystems
5. **YAML Pipeline config** - Routing playbooks as config, not code

### Risk Assessment

| Risk | Level | Notes |
|------|-------|-------|
| Market | LOW | Active space but no direct competitor |
| Technical | MEDIUM-LOW | Patterns exist, MCP proven, OpenCode API tested |
| Adoption | MEDIUM | Many choices exist, our differentiator is simplicity |
| Dependency | LOW | Building on stable standards (MCP, open source) |
| Obsolescence | MEDIUM | Fast-moving space, mitigate by moving fast |

---

## Project Evaluations

| Project | Relevance | Maturity | Architecture | Learn/Use/Compete |
|---------|-----------|----------|--------------|-------------------|
| steipete/claude-code-mcp | HIGH | Active | One-shot MCP server | **STUDY** - potential base for hub |
| oh-my-opencode-CLI | HIGH | Active/Community | Claude compat layer | **STUDY** - potential spoke solution |
| lastmile-ai/mcp-agent | MEDIUM | Active | Workflow patterns | LEARN patterns |
| ruvnet/claude-flow | MEDIUM | Active/v2.7 | Enterprise swarms | LEARN patterns |
| langroid/langroid | MEDIUM | Mature | Message exchange | LEARN architecture |
| crewAIInc/crewAI | LOW | Enterprise/100k+ | Role-based agents | AWARENESS only |
| microsoft/agent-framework | LOW | Enterprise | Graph workflows | AWARENESS only |
| block/goose | LOW | Active | Multi-model config | AWARENESS only |

---

## Next Actions

### Immediate (Before Implementation)

1. **Deep-dive on claude-code-mcp**
   - Clone and study one-shot pattern
   - Understand permission bypass mechanism
   - Evaluate: use directly vs learn from?

2. **Deep-dive on oh-my-opencode**
   - What's in the Claude Code compatibility layer?
   - Async subagent implementation details
   - Evaluate: integrate vs build own?

3. **Document patterns to adopt**
   - Create patterns reference doc
   - Decide which patterns to implement

### Then (Implementation Phase)

4. Build middleware with informed decisions
5. Consider contributing back to ecosystem
