# F2: Peer-to-Peer Agent Messaging

**Feature ID**: F2
**Proposed**: 2026-01-08
**Status**: IDEA
**Priority**: HIGH

---

## Problem Statement

Current architecture assumes hub-spoke with subprocess spawning:
- Claude Code spawns OpenCode as child process
- One-shot execution, ephemeral sessions
- No awareness of other running agents

**Real-world scenario**: User has multiple CLI windows open:
- Window 1: Claude Code session
- Window 2: Codex session
- Window 3: OpenCode session

Each agent should be able to communicate with the others in real-time.

---

## Proposed Solution

Transform the MCP middleware from a **subprocess spawner** to a **message broker**.

### Architecture Evolution

**Before (Hub-Spoke, Subprocess)**:
```
Claude Code (hub)
    └── MCP Server (middleware)
            └── opencode run (spawned, ephemeral)
```

**After (Peer-to-Peer, Persistent)**:
```
           ┌─────────────────────────────┐
           │   MCP Server (broker)       │
           │   - Agent Registry          │
           │   - Message Router          │
           │   - Online Tracking         │
           └─────────────┬───────────────┘
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
┌───────────┐     ┌───────────┐     ┌───────────┐
│Claude Code│     │  Codex    │     │ OpenCode  │
│ Window 1  │     │ Window 2  │     │ Window 3  │
└───────────┘     └───────────┘     └───────────┘
```

---

## Proposed MCP Tools

### Agent Lifecycle

| Tool | Purpose |
|------|---------|
| `register_agent(name, type, capabilities)` | Announce presence to broker |
| `unregister_agent()` | Clean disconnect |
| `heartbeat()` | Keep-alive signal |

### Discovery

| Tool | Purpose |
|------|---------|
| `list_agents()` | Get all online agents |
| `get_agent_info(agent_id)` | Get specific agent details |
| `query_capabilities(capability)` | Find agents with specific capability |

### Messaging

| Tool | Purpose |
|------|---------|
| `send_message(agent_id, message, mode)` | Direct message to specific agent |
| `broadcast(message, mode)` | Send to all online agents |
| `get_messages()` | Retrieve pending messages for self |
| `subscribe(topic)` | Subscribe to topic-based messages |

---

## Agent Registry Schema

```json
{
  "agent_id": "uuid-v4",
  "name": "claude-code-main",
  "type": "claude-code",
  "capabilities": ["file_read", "file_write", "bash", "mcp"],
  "status": "online",
  "connected_at": "2026-01-08T19:30:00Z",
  "last_heartbeat": "2026-01-08T19:35:00Z",
  "metadata": {
    "model": "claude-opus-4-5",
    "cwd": "/home/user/project"
  }
}
```

---

## Message Schema

```json
{
  "message_id": "uuid-v4",
  "from_agent": "agent-id-1",
  "to_agent": "agent-id-2",  // or "broadcast"
  "mode": "challenge",       // challenge/agree/collaborate/deduce
  "content": {
    "type": "request",       // request/response/notification
    "payload": "Review this code for security issues",
    "context": { ... }
  },
  "timestamp": "2026-01-08T19:35:00Z",
  "status": "pending"        // pending/delivered/read
}
```

---

## Implementation Considerations

### Connection Management

1. **MCP stdio limitation**: MCP uses stdio, which is 1:1 with the host process
2. **Solution options**:
   - Shared file-based message queue
   - Local socket server alongside MCP
   - HTTP polling endpoint
   - SQLite as shared state

### Heartbeat & Timeout

- Heartbeat interval: 30 seconds
- Timeout threshold: 90 seconds (3 missed heartbeats)
- On timeout: Mark agent offline, queue messages

### Message Delivery

- **Online agent**: Deliver immediately via MCP response
- **Offline agent**: Queue in persistent storage
- **TTL**: Messages expire after 1 hour by default

---

## Use Cases

### UC1: Code Review Handoff

```
Claude Code: "Hey Codex, review this auth implementation for security issues"
Codex: [receives message, reviews code, responds]
Claude Code: [receives response, presents to user]
```

### UC2: Collaborative Debugging

```
Claude Code: [broadcast] "Found a bug in module X, anyone have context?"
OpenCode: "I worked on that yesterday, the issue is in line 45"
Codex: "I can run the test suite to verify"
```

### UC3: Capability Delegation

```
Claude Code: [query_capabilities("gpu_inference")]
Response: [OpenCode has gpu_inference]
Claude Code: [send_message(opencode, "Run this model locally")]
```

---

## Open Questions

1. **Persistence**: Where to store registry/messages? SQLite? File-based?
2. **Security**: How to prevent rogue agents? Authentication?
3. **Ordering**: Guaranteed message ordering or best-effort?
4. **Scale**: How many concurrent agents to support?
5. **Cross-machine**: Local only, or network-capable?

---

## Relationship to Existing Features

| Existing | How F2 Extends It |
|----------|-------------------|
| `ask_agent()` | Still works for one-shot, F2 adds persistent messaging |
| Interaction modes | Applied to peer messages, not just spawned agents |
| Agent registry | Evolves from static config to dynamic discovery |

---

## Research Needed

- How do existing multi-agent systems handle peer discovery?
- MCP protocol support for multi-client scenarios
- Message broker patterns (pub/sub, point-to-point, hybrid)
- Existing implementations to learn from (MIT licensed)

---

## Success Criteria

1. Agent A can discover Agent B is online
2. Agent A can send message to Agent B
3. Agent B receives message and can respond
4. Works across Claude Code, Codex, OpenCode
5. Graceful handling of agent disconnect
