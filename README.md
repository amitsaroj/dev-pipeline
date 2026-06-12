# Multi-Agent Dev Pipeline — Setup Guide

## Prerequisites

```bash
# Claude Code v1.0.33 or higher required
claude --version

# Enable agent teams (experimental feature flag)
# Add to your Claude Code settings.json:
```

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Settings file location:
- **Linux/Mac:** `~/.claude/settings.json`
- **Windows:** `%APPDATA%\claude\settings.json`

---

## Installation

### Option A — New Project
```bash
# Copy this entire dev-pipeline folder into your project root
cp -r dev-pipeline/. /your/project/

# Verify structure
ls -la .claude/agents/
# Should show: thinker.md, researcher.md, coder.md,
#              auditor.md, security-sentinel.md, db-migrator.md,
#              reviewer.md, commit-writer.md, changelog.md, pr-writer.md
```

### Option B — Existing Project
Copy individual files:
```bash
mkdir -p .claude/agents .claude/commands

# Copy agents
cp dev-pipeline/.claude/agents/*.md .claude/agents/

# Copy slash command
cp dev-pipeline/.claude/commands/dev.md .claude/commands/

# Merge CLAUDE.md (don't overwrite yours — append the relevant sections)
cat dev-pipeline/CLAUDE.md
```

---

## Enable Split-Pane Mode (optional but recommended)

Lets you watch all agents work simultaneously in separate terminal panes.
Requires **tmux**:

```bash
# Install tmux (Ubuntu/Debian)
sudo apt install tmux

# Start Claude Code inside a tmux session
tmux new -s dev
claude

# Force tmux backend
export CLAUDE_CODE_SPAWN_BACKEND=tmux
```

---

## Usage

```bash
# Open your project in Claude Code
cd /your/project
claude

# Run the pipeline on a new feature
/dev Add WhatsApp webhook that receives messages and triggers n8n workflow

/dev Implement multi-tenant API key management with rate limiting per tenant

/dev Add Redis-based job queue for async email sending with retry logic
```

---

## Pipeline Flow

```
/dev <your feature>
        │
        ▼
   [PARALLEL]
   thinker + researcher
        │
        ▼
   ⛔ HUMAN CHECKPOINT
   (approve plan before coding)
        │
        ▼
      coder
        │
        ▼
   [PARALLEL × 3]
   auditor + security-sentinel + db-migrator
        │
        ▼  (fix loop — max 3 retries, escalate on failure)
      reviewer
        │
        ▼
   [PARALLEL × 3]
   commit-writer + changelog + pr-writer
        │
        ▼
   👤 YOU REVIEW & PUSH
```

---

## Agent Token Budget (rough estimate per run)

| Agent | Tokens | Model |
|---|---|---|
| thinker | ~8k | Opus 4.6 |
| researcher | ~5k | Sonnet 4.6 |
| coder | ~20k+ | Opus 4.6 |
| auditor | ~6k | Sonnet 4.6 |
| security-sentinel | ~6k | Sonnet 4.6 |
| db-migrator | ~4k | Sonnet 4.6 |
| reviewer | ~8k | Opus 4.6 |
| commit-writer | ~2k | Sonnet 4.6 |
| changelog | ~2k | Sonnet 4.6 |
| pr-writer | ~3k | Sonnet 4.6 |
| **Total** | **~64k+** | |

Use Sonnet 4.6 for thinker/reviewer if cost is a concern:
Change `model: claude-opus-4-6` to `model: claude-sonnet-4-6` in the agent `.md` files.

---

## Troubleshooting

**Agent teams not working:**
```bash
# Verify flag is set
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
# Should output: 1

# Check Claude Code version
claude --version
# Must be 1.0.33+
```

**Teammate not receiving messages:**
```bash
# Check inbox files
cat ~/.claude/teams/dev-pipeline/inboxes/team-lead.json
```

**Team cleanup if something crashes:**
```bash
rm -rf ~/.claude/teams/dev-pipeline
rm -rf ~/.claude/tasks/dev-pipeline
```

---

## Files Structure

```
your-project/
├── CLAUDE.md                          ← project config + orchestration rules
└── .claude/
    ├── agents/
    │   ├── thinker.md                 ← planning agent
    │   ├── researcher.md              ← third-party research agent
    │   ├── coder.md                   ← implementation agent
    │   ├── auditor.md                 ← code quality + env/docker audit agent
    │   ├── security-sentinel.md       ← security vulnerability audit agent
    │   ├── db-migrator.md             ← database migration generation + safety agent
    │   ├── reviewer.md                ← final go/no-go agent
    │   ├── commit-writer.md           ← git commit message agent
    │   ├── changelog.md               ← CHANGELOG.md + semver bump agent
    │   └── pr-writer.md               ← GitHub PR description agent
    └── commands/
        └── dev.md                     ← /dev slash command (orchestrator)
```
# dev-pipeline
