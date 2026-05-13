# Party Mode — Meeting Skill
## For the Hackathon Repo

---

## Overview

This is a **roundtable meeting mode** for multi-agent conversations. Unlike BMAD party mode (parallel subagent responses), this variant is **sequential** — agents are consulted one at a time, each brings their model-specific perspective, and each independently decides whether they have something to add.

The goal is to **benefit from model diversity** — different underlying models genuinely reason differently. The same question seen through Opus, Haiku, Qwen3-Coder-Next, and GPT-oss-120b produces four genuinely different lenses, not four polished versions of the same answer.

---

## The Core Loop

### 1. Receive the Meeting Topic

The user poses a question, problem, or topic to the group. You frame it for the room:

> *"I've brought together [list agents] to discuss: **[topic]**."*

### 2. Pick an Agent Order

Choose the order based on relevance and the kind of thinking you want to prime:

- **Lead with an agent whose expertise anchors the topic** — they set the frame
- **Follow with agents from different model families** — they react to the anchor with a different lens
- **Vary the order round to round** — don't always start with the same agent

Default group size: **3-4 agents** per topic. More than 4 and the meeting loses coherence. Fewer than 3 and you lose the model-diversity advantage.

### 3. Invite Each Agent — One at a Time

For each agent in order, spawn them with:

```
You are {name} ({title}), participating in a roundtable meeting.

## Your Persona
{icon} {name} — {description}

## The Meeting Topic
{topic as stated by the facilitator}

## Discussion So Far
{summary of what previous agents said this round — keep under 300 words}
{if no prior responses, omit this section}

## Your Model
You are running on {model}. Remember: your model has specific strengths and blind spots. Lean into what your model is genuinely good at, and flag what it struggles with.

## Guidelines
- You may respond OR pass. If you have nothing substantive to add to what's already been said, say so briefly and pass your turn.
- Start your response with: {icon} **{name}:**
- Respond authentically as {name}. Don't repeat or rephrase what others said — build on it, challenge it, or add a dimension they missed.
- If you agree with a prior response, say why in your own words rather than just endorsing.
- Disagree openly when your perspective tells you to. Don't hedge or soften.
- If the question is outside your expertise, say so — don't improvise.
- Scale your response to the substance. A brief point = a brief answer. Complex topic = give it the depth it needs.
- You may ask the facilitator to clarify the topic before responding.
- Do NOT use tools. Just respond with your perspective.
```

### 4. Agents May React to Each Other

After each agent responds, **ask the remaining agents if they want to react**:

> *"Winston, do you want to respond to what Sally just said? Pass is fine."*

Agents can:
- **Build** on a prior response (add a dimension, provide evidence)
- **Challenge** a prior response (disagree, flag a blind spot)
- **Pass** (nothing to add — this is valid and encouraged)

This is where model diversity pays off — one model's blind spot is another model's specialty.

### 5. Present the Full Meeting

After all agents have spoken (or passed), present the full exchange to the user:

- Each agent's response in full — unabridged, in their own voice
- A note when an agent passed ("_Winston passed — nothing to add_")
- Agent-to-agent reactions included as part of the relevant agent's turn

Format:
```
## Meeting: [topic]

[Agent A response]

[Agent B response or "Agent B passed"]

[Agent A reacts to B]

[Agent C response]
```

### 6. Wrap-Up

When the user signals the meeting is done, close with a brief summary of where agents converged, where they diverged, and any open questions the group didn't resolve.

---

## Decision Rules

| Situation | What to do |
|---|---|
| Agent passes | Note it, move to next. Don't retry. |
| All agents pass | Ask the user if they want to reframe the question |
| Agent goes off-topic | Gently redirect in the next spawn prompt |
| Agent overlaps with prior response | Let it stand — different phrasing has value |
| User wants a specific agent only | Spawn just that agent with the full discussion context |
| Topic shifts mid-meeting | Start a new meeting round with relevant agents |

---

## Arguments

- `--solo` — Skip subagents. Roleplay all agents yourself. Use when subagents are unavailable or speed matters more than independence.
- `--model <model>` — Force all agents to use the same model. Overrides the natural model diversity of the group.
- `--agent <code>` — Include a specific agent regardless of topic relevance (e.g., `--agent amelia`).

---

## Exit

User says they're done (any natural phrasing). Give a 2-3 sentence summary of key convergences and divergences. Return to normal mode.

---

## SKILL.md Frontmatter

```yaml
---
name: party-mode-meeting
description: 'Run a sequential roundtable meeting with multiple agents. Each agent is consulted one at a time and may respond or pass. Use when the user wants a structured multi-agent discussion where model diversity is a feature — agents on different underlying models genuinely contribute different perspectives. Compare: BMAD party mode (parallel), which this is adapted from.'
---
```
