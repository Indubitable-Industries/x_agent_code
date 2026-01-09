# F9 PoC Test Harness

**Test ID**: F9-POC
**Created**: 2026-01-08
**Status**: PARTIAL (T1-T4 PASS via curl, T5-T7 pending Claude Code client)
**Depends On**: F9 research, PoC implementation
**PoC Location**: [`poc/f9_long_polling/`](../../poc/f9_long_polling/)

---

## Objective

Empirically validate the long-polling keep-alive pattern with Claude Code to determine if child agents can wait in open tool calls while receiving heartbeat progress notifications.

---

## Test Environment

### Prerequisites

| Requirement | Version/Details | Verified |
|-------------|-----------------|----------|
| Python | 3.10.12 | [x] |
| uv | Latest | [x] |
| Claude Code | 2.1.2 | [x] |
| Port 8000 | Available | [x] |

### Setup Commands

```bash
# 1. Navigate to PoC directory
cd /home/phaze/PycharmProjects/x_agent_code/poc/f9_long_polling

# 2. Install dependencies
uv sync

# 3. Start server (Terminal 1)
uv run server.py

# 4. Register MCP server with Claude Code (Terminal 2)
claude mcp add --transport http f9-poc http://localhost:8000/mcp

# 5. Verify registration
claude mcp list
```

---

## Test Cases

### T1: Basic Tool Registration

**Objective**: Verify MCP server connects and tools are visible.

**Steps**:
1. Start server: `uv run server.py`
2. In Claude Code: "List available MCP tools"

**Expected**:
- `register_and_wait` tool visible
- `send_message` tool visible
- `check_status` tool visible

**Pass Criteria**: All 3 tools listed
**Fail Criteria**: Tools not found or connection error

**Result**: `[x] PASS  [ ] FAIL  [ ] SKIP`
**Notes**: Tested via curl. All 3 tools (register_and_wait, send_message, check_status) visible.

---

### T2: Short Wait with Timeout

**Objective**: Verify agent waits for tool completion and receives timeout response.

**Steps**:
1. Start server
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 30"
3. Wait for response (do NOT send any message)

**Expected**:
- Server logs show heartbeat at ~30s mark
- Tool returns after ~30s with `{"no_work": true}`
- Claude Code displays the response (not an error)

**Pass Criteria**:
- [x] Tool returns cleanly (not timeout error)
- [x] Response contains `no_work: true`
- [x] Server logged at least 1 heartbeat

**Fail Criteria**:
- Claude Code shows "tool timeout" error before server returns
- Connection drops during wait
- No heartbeat logged

**Result**: `[x] PASS  [ ] FAIL  [ ] SKIP`
**Notes**: Tested via curl with 15s timeout. Response: `{"no_work": true, "waited_seconds": 15, "heartbeats_sent": 1}`

---

### T3: Message Delivery During Wait

**Objective**: Verify message can be delivered to waiting agent.

**Steps**:
1. Start server
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 120"
3. In Terminal 3 (within 30 seconds):
   ```bash
   cd /home/phaze/PycharmProjects/x_agent_code/poc/f9_long_polling
   uv run send_message.py "Test message from harness" challenge
   ```
4. Observe Claude Code response

**Expected**:
- Server logs "Message found and consumed"
- Tool returns with `no_work: false` and message content
- Claude Code receives and displays the message

**Pass Criteria**:
- [x] Message delivered to waiting agent
- [x] Response contains message content
- [x] Mode is "challenge"
- [x] Delivery time < 5 seconds after message sent

**Fail Criteria**:
- Message not received
- Timeout before message delivery
- Wrong message content

**Result**: `[x] PASS  [ ] FAIL  [ ] SKIP`
**Notes**: Tested via curl. Message delivered in ~3 seconds. Response: `{"no_work": false, "agent_name": "t3b-agent", "message": {"content": "T3 test message", "mode": "challenge", "timestamp": "2026-01-08", "from": "test"}, "waited_seconds": 3, "heartbeats_sent": 1}`

---

### T4: Multiple Heartbeats

**Objective**: Verify multiple heartbeats are sent during extended wait.

**Steps**:
1. Start server
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 90"
3. Wait for full duration (do NOT send message)

**Expected**:
- Server logs 3 heartbeats (at ~30s, ~60s, ~90s)
- Progress notifications visible in server logs
- Tool returns with `heartbeats_sent: 3`

**Pass Criteria**:
- [x] 3 heartbeats logged
- [x] No connection drops
- [x] Response shows correct heartbeat count

**Fail Criteria**:
- Fewer than 3 heartbeats
- Connection drops during wait
- Claude Code timeout error

**Result**: `[x] PASS  [ ] FAIL  [ ] SKIP`
**Notes**: Tested via curl with 90s timeout. Response: `{"no_work": true, "waited_seconds": 90, "heartbeats_sent": 3}`. Server logged heartbeats at ~30s intervals.

---

### T5: User Interrupt (Ctrl+C)

**Objective**: Verify user can interrupt a waiting agent.

**Steps**:
1. Start server
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 300"
3. After ~15 seconds, press Ctrl+C in Claude Code
4. Attempt to use Claude Code normally

**Expected**:
- Wait is interrupted
- Claude Code remains responsive
- No hung state or error cascade

**Pass Criteria**:
- [ ] Ctrl+C interrupts the wait
- [ ] Claude Code usable after interrupt
- [ ] Server handles disconnection gracefully

**Fail Criteria**:
- Cannot interrupt
- Claude Code becomes unresponsive
- Requires restart

**Result**: `[ ] PASS  [ ] FAIL  [ ] SKIP`
**Notes**:

---

### T6: Token Usage Measurement

**Objective**: Measure token overhead of waiting.

**Steps**:
1. Note Claude Code token count before test
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 60"
3. Wait for timeout
4. Note Claude Code token count after test
5. Calculate tokens used

**Expected**:
- Minimal token usage (< 100 tokens per minute of waiting)
- Progress notifications don't add to model context

**Pass Criteria**:
- [ ] Token usage < 200 tokens for 60s wait
- [ ] Acceptable overhead for production use

**Fail Criteria**:
- Token usage > 500 tokens for 60s wait
- Exponential token growth with wait time

**Result**: `[ ] PASS  [ ] FAIL  [ ] SKIP`
**Tokens Before**:
**Tokens After**:
**Delta**:
**Notes**:

---

### T7: Extended Duration (Claude Code Timeout Test)

**Objective**: Test if Claude Code's internal timeout can handle longer waits.

**Steps**:
1. Start server
2. In Claude Code: "Use register_and_wait with agent_name 'test' and timeout_seconds 180"
3. At ~150 seconds, send message:
   ```bash
   uv run send_message.py "Extended wait test" info
   ```

**Expected**:
- Claude Code waits full 150 seconds
- Message delivered successfully
- No timeout errors from Claude Code

**Pass Criteria**:
- [ ] Wait exceeds 2 minutes (Claude Code default timeout)
- [ ] Message delivered successfully
- [ ] Progress notifications kept connection alive

**Fail Criteria**:
- Claude Code timeout at 2 minutes
- Connection dropped
- Message not delivered

**Result**: `[ ] PASS  [ ] FAIL  [ ] SKIP`
**Actual Wait Duration**:
**Notes**:

---

## Test Execution Log

| Test | Date | Executor | Result | Notes |
|------|------|----------|--------|-------|
| T1 | 2026-01-08 | curl | PASS | All 3 tools visible |
| T2 | 2026-01-08 | curl | PASS | 15s timeout, 1 heartbeat |
| T3 | 2026-01-08 | curl | PASS | Message delivered in 3s |
| T4 | 2026-01-08 | curl | PASS | 90s, 3 heartbeats |
| T5 | | Claude Code | PENDING | Requires client testing |
| T6 | | Claude Code | PENDING | Requires client testing |
| T7 | | Claude Code | PENDING | Requires client testing |

---

## Environment Capture

Record before testing:

```
Date: 2026-01-08
Claude Code Version: 2.1.2
Python Version: 3.10.12
OS: Linux 6.9.3-76060903-generic
MCP SDK Version: mcp>=1.9.0

Claude Code Settings (if modified):
- MCP_TIMEOUT: default
- BASH_DEFAULT_TIMEOUT_MS: default (120000)
- Other: f9-poc server registered via `claude mcp add`
```

---

## Summary & Conclusions

### Overall Result

`[ ] PASS - Pattern validated, proceed to implementation`
`[x] PARTIAL - Server tests pass, Claude Code client tests pending`
`[ ] FAIL - Pattern does not work as expected`

### Key Findings

1. **Server-side pattern works**: MCP server correctly holds connections, sends heartbeats, and delivers messages
2. **Heartbeat mechanism validated**: Progress notifications sent every 30s as expected
3. **Message delivery fast**: <5s latency from message queue to waiting agent response
4. **SSE streaming functional**: Streamable HTTP transport working correctly

### Blockers Identified

1. T5-T7 require testing with Claude Code as actual MCP client (not curl)
2. Need fresh Claude Code session to test MCP tool integration

### Recommendations

1. Complete T5-T7 in fresh Claude Code session with f9-poc MCP server active
2. If T5-T7 pass, proceed to Phase 2 optimization (adaptive timeouts, multi-agent)

---

## Post-Test Actions

- [ ] Update F9 research doc with results
- [ ] Update TRACKER.md status
- [ ] If PASS: Create implementation task
- [ ] If FAIL: Document failure mode, explore alternatives
- [ ] Commit test results

---

## References

- [F9 Research](../features/F9_long_polling_keepalive.md)
- [F8 Notification Feasibility](F8_notification_feasibility.md) (negative result that led to F9)
- [PoC README](../../poc/f9_long_polling/README.md)
