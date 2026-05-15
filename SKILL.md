---
name: agent-party
description: >
  Run a multi-round roundtable meeting with multiple OpenClaw agents. Each
  round broadcasts the topic to all agents simultaneously, collects their
  contributions (including Actionables), shares results back, then refines
  in the next round. Use when you want structured multi-agent collaboration
  where model diversity produces genuinely different perspectives and
  concrete, owned next steps.
metadata:
  {
    "openclaw":
      {
        "emoji": "🎉",
        "os": ["darwin", "linux"],
      },
  }
---

# Party Mode — Meeting Skill
## For the Hackathon Repo

---

## Overview

This is a **broadcast roundtable mode** for multi-agent collaboration. Every agent receives the same prompt simultaneously (fire-and-forget), their contributions are collected, shared back with the group, and the next round refines based on what everyone said.

The goal is to **benefit from model diversity** — different underlying models genuinely reason differently. The same question seen through Opus, Haiku, Qwen3-Coder-Next, and GPT-oss-120b produces four genuinely different lenses, not four polished versions of the same answer.

Each agent also defines **Actionables** — concrete things they personally own — so the meeting ends with a structured, grouped list of who does what.

---

## Agent Discovery and Session Filter

Before starting any roundtable, discover which agents are actually running and reachable. This step determines whether to use **real agent sessions** (preferred) or fall back to **solo spawn mode**.

### Step 1 — List Active Sessions

```
sessions_list(
  kinds: ["main"],          // main sessions only — skip threads, hooks, cron
  activeMinutes: 60,        // agents active in the last hour
  includeLastMessage: true  // helps identify what each agent is doing
)
```

Returns an array of session objects. Relevant fields:

| Field | Use |
|---|---|
| `sessionKey` | Passed to `sessions_send` and `sessions_history` |
| `agentId` | Used to exclude own session and for exact-match filtering |
| `label` | Human-readable agent name — used for display and name-based filtering |

### Step 2 — Filter by Name or Role (when `--agent` is specified)

If the user named specific agents (via `--agent <name>` or @-mention), narrow the list:

```
matching = sessions.filter(s =>
  s.label.toLowerCase().includes(target.toLowerCase())
  || s.agentId === target
)
```

For role-based filtering, pass the term directly into `sessions_list` instead:

```
sessions_list(kinds: ["main"], activeMinutes: 60, search: "engineer")
```

The `search` parameter does full-text matching against session metadata.

### Step 3 — Exclude Own Session

Your own session appears in the results. Remove it:

```
others = sessions.filter(s => s.sessionKey !== ownSessionKey)
```

**How to identify own session:** OpenClaw provides the current session key in the execution context. If it is not directly accessible, fall back to comparing `s.agentId` against your own agent identifier, or exclude the session whose label matches your own name.

### Step 4 — Handle Empty and Undersized Results

| `others` size | Action |
|---|---|
| 0 (no agents active) | Announce: *"No other agents are currently available. Running in solo mode."* → proceed with `--solo` |
| 1 agent | Announce: *"Only [Agent] is available. Continuing as a 1-on-1."* → proceed with just that agent |
| 2–4 agents | Proceed with roundtable |
| > 4 agents | Cap at 4 for coherence: `others = others.slice(0, 4)` |

If `--agent` was specified but no session matched: ask the user whether to include all available agents or abort.

If `--agent` was not specified but no agents are active: announce and fall back to `--solo` automatically.

### Step 5 — Build the Agent Roster

Collect `{ sessionKey, label }` for each selected agent. This roster is the input to the broadcast loop:

```
roster = [
  { sessionKey: "agent:main:main", label: "Amelia" },
  { sessionKey: "agent:work:main", label: "Winston" },
  ...
]
```

Pass `roster` into `sessions_send` and `sessions_history` in all subsequent steps. Never hard-code session keys — always derive them from the discovery step.

### Solo Mode Fallback

When `others` is empty after filtering, or when `--solo` is passed explicitly:

- Skip `sessions_list` and `sessions_send` entirely
- Spawn a subagent for each "agent" persona instead
- The roundtable script below is identical — only the delivery mechanism differs

This ensures the skill works even when no other agents are running or reachable.

---

## The Broadcast-Collect-Share Cycle

Repeat for each round `r` from 1 to `--rounds` (default: 2).

---

### Phase 1 — Broadcast

Send the round prompt to **all agents simultaneously** using fire-and-forget:

```
for agent in roster:
  sessions_send(
    sessionKey: agent.sessionKey,
    message: round_prompt(r, topic, prior_digest),
    timeoutSeconds: 0   // fire-and-forget — do not block
  )
```

`timeoutSeconds: 0` means the call returns immediately without waiting for a reply. This is the only way to achieve parallel delivery, since `sessions_send` is one-to-one and `SUBAGENT_TOOL_DENY_ALWAYS` blocks parallel subagents from calling it.

**Important:** `sessions_send` is only callable from the main agent session. Never attempt to fan out via spawned subagents.

#### Round 1 Prompt Template

```
You are {name}, a knight at the roundtable.

## The Topic
{topic as stated by the facilitator}

## Your Role
{name} — {one-line description of this agent's model/specialty}

## Guidelines
- Respond with your genuine perspective on the topic.
- End your response with a section called **Actionables** — 3 to 5 specific, concrete things that YOU will do or own related to this topic. Use checkbox format:
  - [ ] Action 1
  - [ ] Action 2
- If you have nothing substantive to contribute, say so briefly. You may still list Actionables if you have clear ownership.
- Do NOT use tools. Just respond with your perspective and Actionables.
```

#### Round 2..N Prompt Template

```
You are {name}, a knight at the roundtable — Round {r} of {total_rounds}.

## The Topic
{topic}

## What Your Peers Contributed in Round {r-1}
{prior_digest — one paragraph per agent, max 150 words each}

## Your Task This Round
Refine your position based on what you've heard. Build on, challenge, or confirm the contributions above.

End your response with an updated **Actionables** section — keep, drop, or add items based on this round's discussion.
```

---

### Phase 2 — Collect

After broadcasting, wait briefly then poll each agent for their reply using `sessions_history`:

```
replies = {}
for agent in roster:
  history = sessions_history(sessionKey: agent.sessionKey, limit: 5)
  // take the most recent assistant message posted after the broadcast
  replies[agent.label] = extract_latest_reply(history, after: broadcast_timestamp)
```

If an agent hasn't replied yet, wait up to 30 seconds total (poll every 5 s) before marking them as "no response this round."

---

### Phase 3 — Share Digest

Compile a **round digest** from all replies and display it to the user:

```
## Round {r} — Contributions

**{Agent A}:**
{Agent A's response, capped at 200 words if very long}

**Actionables:**
- [ ] {A1}
- [ ] {A2}

---

**{Agent B}:**
{Agent B's response}

**Actionables:**
- [ ] {B1}

---

_{Agent C} passed — nothing to add._
```

Store the digest as `prior_digest` for Round `r+1`.

---

### Phase 4 — Refine (Rounds 2..N)

After sharing the digest with the user, start Round `r+1`: broadcast the Round 2..N prompt (which includes `prior_digest`) to all agents again. Repeat Phases 1–3.

---

## Final Aggregation

After the last round, collect **all Actionables** from all agents across all rounds and present a grouped summary:

```
## Roundtable Summary — [topic]

### Actionables by Topic

**[Topic Group A]**
| Owner | Action |
|---|---|
| {Agent} | {Actionable} |
| {Agent} | {Actionable} |

**[Topic Group B]**
| Owner | Action |
|---|---|
| {Agent} | {Actionable} |

### Convergences
- {Points where 2+ agents agreed}

### Divergences
- {Points where agents disagreed — note which agent held which position}

### Open Questions
- {Questions the group raised but did not resolve}
```

**De-duplication:** If two agents listed the same Actionable, merge into one row and list both owners.

**Grouping:** Use the topic itself to determine logical groups (e.g., "Architecture", "Testing", "UX"). If the topic is narrow, one group is fine.

---

## Decision Rules

| Situation | What to do |
|---|---|
| Agent passes | Note it in the digest. Don't retry. |
| All agents pass | Ask the user if they want to reframe the topic |
| Agent doesn't reply in 30 s | Mark as "no response this round" and continue |
| Agent goes off-topic | Include their response but note the drift; refocus in next round's prompt |
| Agent overlaps with prior response | Let it stand — different phrasing has value |
| User wants a specific agent only | Include only that agent's `sessionKey` in the roster |
| Topic shifts mid-meeting | Reset `prior_digest`, start a new Round 1 |
| All rounds done but user wants more | Ask if they want another round or a summary |

---

## Arguments

- `--rounds <n>` — Number of broadcast-collect-share cycles. Default: `2`. Minimum: `1`.
- `--agents <name,...>` — Restrict the roster to named agents (comma-separated). Default: all active agents up to the 4-agent cap. Accepts partial name matches.
- `--solo` — Skip real sessions entirely. Roleplay all agents yourself as subagents. Use when no other agents are running or speed matters more than independence.
- `--model <model>` — Force all agents to use the same model. Overrides the natural model diversity of the group.
- `--agent <code>` — Include one specific agent by name or ID regardless of the normal filter (e.g., `--agent amelia`). Can be repeated.

---

## Exit

User says they're done (any natural phrasing). Output the Final Aggregation section. Return to normal mode.
