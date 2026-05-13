# openclaw_agent_party

An OpenClaw skill that broadcasts messages to multiple agents simultaneously — the digital equivalent of a group chat.

## What it does

`agent-party` lets you fan out a single message to any number of running OpenClaw agents and aggregate their responses in one place. Useful for status sweeps, parallel task assignment, and group deliberation.

## Installation

### Prerequisites

- [OpenClaw](https://openclaw.dev) installed (`openclaw --version`)
- `jq` available on PATH

### Add the skill to OpenClaw

```bash
# Clone this repo
git clone https://github.com/mixmash11/openclaw_agent_party
cd openclaw_agent_party

# Install the skill (points OpenClaw at the skills/ directory)
openclaw skills add ./skills/agent-party
```

Verify the skill is recognised:

```bash
openclaw skills list | grep agent-party
```

### jq (if missing)

```bash
brew install jq        # macOS
sudo apt install jq    # Debian/Ubuntu
```

## Usage

Once installed, the skill is available inside any OpenClaw session:

```
/agent-party broadcast "What are you currently working on?"
```

Or call the script directly for scripting use cases:

```bash
node skills/agent-party/skill.js broadcast --message "Status update please"
node skills/agent-party/skill.js discover
```

## Project structure

```
openclaw_agent_party/
└── skills/
    └── agent-party/
        ├── SKILL.md    # Skill definition and routing instructions
        └── skill.js    # Entry point (core logic)
```

## Development

This project is built during a hackathon. Active issues:

- **BRE-41** — Core broadcast logic
- **BRE-42** — Agent discovery and session filter
- **BRE-40** — UX / conversation flow design
- **BRE-43** — Integration & end-to-end test
