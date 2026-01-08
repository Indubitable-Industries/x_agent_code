# F7: OpenCode Invocation Tests

**Task ID**: F7
**Date**: 2026-01-08
**Status**: PASSED

---

## Test Environment

| Component | Version | Path |
|-----------|---------|------|
| OpenCode | 1.0.157 | `/home/phaze/.bun/bin/opencode` |
| Claude Code | 2.1.1 | `/home/phaze/.nvm/versions/node/v22.19.0/bin/claude` |

---

## Test Results

### T1: Installation Verification ✅

```bash
which opencode && opencode --version
# /home/phaze/.bun/bin/opencode
# 1.0.157
```

### T2: HTTP Server (`opencode serve`) ⚠️

```bash
opencode serve --port 3001
# Server starts but proxies to cloud (desktop.opencode.ai)
# NOT suitable for local-only use
```

**Finding**: `opencode serve` is a cloud proxy, not a local API server. Not what we need.

### T3: CLI Run Command (`opencode run`) ✅

```bash
opencode run -m opencode/gpt-5-nano "What is 2+2? Reply with just the number."
# 4
```

**Finding**: This is our invocation method. Direct CLI with model selection.

### T4: Project Context Awareness ✅

```bash
opencode run -m opencode/gpt-5-nano "What is the main purpose of this project?"
# [Reads README.md]
# A cross-CLI agent middleware that enables AI coding assistants to communicate...
```

**Finding**: OpenCode reads project files to answer questions. Full context access.

### T5: Model Selection ✅

```bash
# GPT-5 variant
opencode run -m opencode/gpt-5-nano "List 3 languages"
# ["Python","JavaScript","Go"]

# Grok variant
opencode run -m opencode/grok-code "What is the hub in this architecture?"
# Claude Code
```

**Finding**: Multiple models work via `-m provider/model` flag.

### T6: JSON Output Format ✅

```bash
opencode run --format json -m opencode/grok-code "Reply with JSON"
# {"type":"text","timestamp":...,"part":{"text":"{...}"}}
```

**Finding**: JSON format wraps response in event structure. Parse `part.text` for content.

---

## Available Models (Subset)

### Free/Built-in
- `opencode/gpt-5-nano` - GPT-5 variant
- `opencode/grok-code` - Grok variant
- `opencode/glm-4.7-free` - GLM-4 variant

### Via OpenRouter (requires API key)
- `openrouter/x-ai/grok-4-fast`
- `openrouter/openai/gpt-5`
- `openrouter/anthropic/claude-opus-4.5`

---

## Invocation Pattern for x_agent_code

### Basic Invocation

```bash
opencode run -m MODEL "PROMPT"
```

### With JSON Output (for parsing)

```bash
opencode run --format json -m MODEL "PROMPT"
```

### Key Options

| Flag | Purpose |
|------|---------|
| `-m MODEL` | Select model (required for our use) |
| `--format json` | Machine-parseable output |
| `-c` | Continue last session |
| `-s SESSION_ID` | Continue specific session |
| `-f FILE` | Attach file to message |

---

## Integration Implications

### What Works
1. ✅ CLI invocation from subprocess
2. ✅ Model selection per-request
3. ✅ Project context awareness (reads files)
4. ✅ JSON output for parsing
5. ✅ Multiple model providers

### What Doesn't Work
1. ❌ `opencode serve` for local API (proxies to cloud)
2. ⚠️ Need to parse nested JSON for structured responses

### Recommended Approach

```python
import subprocess
import json

def ask_opencode(prompt: str, model: str = "opencode/gpt-5-nano") -> str:
    result = subprocess.run(
        ["opencode", "run", "-m", model, prompt],
        capture_output=True,
        text=True,
        cwd="/path/to/project"  # Important for context
    )
    return result.stdout.strip()

def ask_opencode_json(prompt: str, model: str) -> dict:
    result = subprocess.run(
        ["opencode", "run", "--format", "json", "-m", model, prompt],
        capture_output=True,
        text=True
    )
    # Parse last text event
    for line in result.stdout.strip().split('\n'):
        event = json.loads(line)
        if event.get('type') == 'text':
            return json.loads(event['part']['text'])
    return {}
```

---

## Next Steps

1. Implement `ask_agent()` MCP tool using this pattern
2. Test with `--dangerously-skip-permissions` for Claude Code invocation
3. Add interaction mode framing to prompts
4. Build response parsing for structured outputs
