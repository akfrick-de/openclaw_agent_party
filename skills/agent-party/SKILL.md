---
name: agent-party
description: Broadcast a message to multiple OpenClaw agents simultaneously and collect their responses. Use when you want to run the same prompt across several agents in parallel, coordinate a group conversation, or fan out a task to a team of agents.
metadata:
  {
    "openclaw":
      {
        "emoji": "🎉",
        "os": ["darwin", "linux"],
        "requires": { "bins": ["jq"] },
        "install":
          [
            {
              "id": "brew-jq",
              "kind": "brew",
              "formula": "jq",
              "bins": ["jq"],
              "label": "Install jq (brew)",
            },
          ],
      },
  }
---

# Agent Party

Broadcast a message to multiple OpenClaw agents at once and aggregate their responses.

## Summary

**agent-party** is an OpenClaw skill that lets you fan out a single message to multiple agents in parallel and collect all their responses in one place. It is useful for group coordination, parallel task processing, and comparing outputs across different agent configurations. The skill exposes a simple CLI (`skill.js`) with two commands: `broadcast` (send a prompt to one or more agents) and `discover` (list reachable agents).

## When to Use

✅ **USE this skill when:**

- Sending the same prompt to several agents in parallel
- Coordinating a group discussion between multiple agents
- Fan-out tasks where multiple agents should independently process the same input
- Comparing responses across different agent configurations

## When NOT to Use

❌ **DON'T use this skill when:**

- Messaging a single agent — use the agent tool directly
- Needing sequential coordination between agents — chain them manually
- Running a pipeline where each agent depends on the previous output

## Usage

```bash
# Broadcast to all active agents
node skills/agent-party/skill.js broadcast --message "Summarize your current task"

# Broadcast to a filtered set of agents (by name or session)
node skills/agent-party/skill.js broadcast --message "Status update please" --filter "worker-*"

# List discoverable agents
node skills/agent-party/skill.js discover
```

## Implementation Status

This skill is under active development. See:

- `skill.js` — entry point (core broadcast logic, BRE-41)
- Session-filter and agent discovery (BRE-42)
- UX / conversation flow design (BRE-40)
