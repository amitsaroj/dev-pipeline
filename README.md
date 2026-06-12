# Autopilot Dev Pipeline

A plug-and-play multi-agent pipeline for Claude Code that turns a single feature request into production-ready, reviewed, and documented code — automatically.

Drop the `.claude/` folder into any NestJS · Next.js · PostgreSQL · Redis · Docker · Qdrant project, then run `/dev <feature>`.

---

## How It Works

You type one command:

```
/dev Add WhatsApp webhook that receives messages and triggers n8n workflow
```

14 specialist agents run in a structured pipeline — planning, coding, building, auditing, reviewing, and writing all your release artifacts — and hand you back a commit message, CHANGELOG entry, and PR description ready to push.

---

## Pipeline Flow

```
/dev <feature>
      │
   preflight           ← validates env, repo state, baseline build (BLOCKING GATE)
      │
      ├── thinker ─────────────────────────────────────────┐
      │                                                     ▼
      └── researcher (parallel) ─────────────────── MERGE PLANS
                                                           │
                                               ⛔ HUMAN CHECKPOINT
                                               (approve plan before coding)
                                                           │
                                                        coder
                                                           │
                                           ┌───────────────┤
                                           ▼               ▼
                                   build-validator    docs-writer
                                   (fast-fail gate — back to coder if TypeScript errors)
                                           │
                          ┌────────────────┼────────────────────┐
                          ▼                ▼            ▼        ▼
                       auditor   security-sentinel  db-migrator  dependency-auditor
                          └────────────────┴────────────┴────────┘
                                           │    fix loop (max 3 retries → human)
                                        reviewer
                                           │
                      ┌────────────────────┼──────────────────┐
                      ▼                    ▼                   ▼
               commit-writer          changelog            pr-writer
                      └────────────────────┴──────────────────┘
                                           │
                                   ⛔ HUMAN REVIEW → PUSH
```

---

## Agents (14 total)

| # | Agent | Role | Stage |
|---|-------|------|-------|
| 1 | **preflight** | Validates git state, Node/npm versions, Docker, services, `.env`, baseline build/tests. Blocks pipeline if broken. | Gate |
| 2 | **thinker** | Deep architecture plan — affected files, data models, API contracts, task breakdown, risks. Never writes code. | 1 ∥ |
| 3 | **researcher** | Investigates npm packages, APIs, best practices. Compares options, checks CVEs and licenses. | 1 ∥ |
| 4 | **coder** | Implements the plan — typed TypeScript, unit tests, integration tests, env vars, docker-compose updates. | 2 |
| 5 | **build-validator** | Runs `tsc --noEmit` + `npm run build` + `docker-compose config`. Fast-fail before expensive audits. | 2.5 ∥ |
| 6 | **docs-writer** | Adds missing Swagger decorators (`@ApiOperation`, `@ApiProperty`, etc.) and JSDoc to public service methods. | 2.5 ∥ |
| 7 | **auditor** | Code quality — correctness, error handling, test coverage, N+1 queries, SOLID violations, observability. | 3 ∥ |
| 8 | **security-sentinel** | Security — injection, auth/authz, data exposure, input validation, multi-tenant isolation, CORS. | 3 ∥ |
| 9 | **db-migrator** | Generates and validates PostgreSQL migrations — reversibility, unsafe ops, missing FK indexes. | 3 ∥ |
| 10 | **dependency-auditor** | Runs `npm audit`, checks outdated packages, license compliance (no GPL in SaaS), phantom deps. | 3 ∥ |
| 11 | **reviewer** | Final go/no-go — re-runs lint/test/build, verifies all CRITICALs resolved, checks feature completeness. | 4 |
| 12 | **commit-writer** | Structured git commit message (conventional commits format). Reads `git diff`, not `git diff --staged`. | 5 ∥ |
| 13 | **changelog** | Determines semver bump, writes Keep a Changelog entry, updates `package.json` version. | 5 ∥ |
| 14 | **pr-writer** | Full GitHub PR description — summary, test plan, deployment notes, breaking changes, API examples. | 5 ∥ |

**∥** = runs in parallel with other agents at the same stage.

---

## Installation

### Prerequisites

```bash
# Claude Code v1.0.33 or higher
claude --version

# Enable agent teams (experimental feature flag)
# Add to ~/.claude/settings.json (Linux/Mac) or %APPDATA%\claude\settings.json (Windows)
```

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Option A — New Project

```bash
# Copy .claude/ and CLAUDE.md into your project root
cp -r /path/to/dev-pipeline/.claude /your/project/
cp /path/to/dev-pipeline/CLAUDE.md /your/project/

# Verify
ls .claude/agents/
# preflight.md  thinker.md  researcher.md  coder.md
# build-validator.md  docs-writer.md  auditor.md  security-sentinel.md
# db-migrator.md  dependency-auditor.md  reviewer.md
# commit-writer.md  changelog.md  pr-writer.md
```

### Option B — Clone and Copy

```bash
git clone https://github.com/amitsaroj/dev-pipeline.git
cd your-project
cp -r ../dev-pipeline/.claude .
cp ../dev-pipeline/CLAUDE.md .
```

### Option C — Split-Pane Mode (recommended for watching agents)

Requires **tmux** to see all agents working simultaneously:

```bash
sudo apt install tmux          # Ubuntu/Debian
tmux new -s dev
claude
export CLAUDE_CODE_SPAWN_BACKEND=tmux
```

---

## Usage

```bash
cd /your/project
claude

# Feature examples
/dev Add WhatsApp webhook that receives messages and triggers n8n workflow
/dev Implement multi-tenant API key management with rate limiting per tenant
/dev Add Redis-based job queue for async email sending with retry logic
/dev Add vector search for products using Qdrant with semantic similarity
```

---

## Human Checkpoints

The pipeline has **3 moments** where it stops and waits for you:

| Checkpoint | When | What you decide |
|---|---|---|
| **Plan Approval** | After thinker + researcher | Approve the architecture before any code is written. Request changes if needed. |
| **Fix Loop Escalation** | If 3 fix attempts fail | Choose: give coder a specific approach, skip a specific issue, or abort. |
| **Final Review** | After pipeline completes | Review commit message, CHANGELOG, and PR description. Then push manually. |

You always push manually:
```bash
git add .
git commit -m "feat(scope): summary"
git push -u origin feat/<slug>
gh pr create --title "..." --body "..."
```

---

## Agent Token Budget

| Agent | ~Tokens | Model |
|---|---|---|
| preflight | 2k | Sonnet 4.6 |
| thinker | 8k | Opus 4.6 |
| researcher | 5k | Sonnet 4.6 |
| coder | 20k+ | Opus 4.6 |
| build-validator | 3k | Sonnet 4.6 |
| docs-writer | 4k | Sonnet 4.6 |
| auditor | 6k | Sonnet 4.6 |
| security-sentinel | 6k | Sonnet 4.6 |
| db-migrator | 4k | Sonnet 4.6 |
| dependency-auditor | 3k | Sonnet 4.6 |
| reviewer | 8k | Opus 4.6 |
| commit-writer | 2k | Sonnet 4.6 |
| changelog | 2k | Sonnet 4.6 |
| pr-writer | 3k | Sonnet 4.6 |
| **Total** | **~76k+** | |

To reduce cost: change `model: claude-opus-4-6` → `model: claude-sonnet-4-6` in `thinker.md`, `coder.md`, and `reviewer.md`.

---

## Project Structure

```
your-project/
├── CLAUDE.md                       ← pipeline config + orchestration rules
└── .claude/
    ├── commands/
    │   └── dev.md                  ← /dev slash command (orchestrator)
    └── agents/
        ├── preflight.md            ← env + repo health gate
        ├── thinker.md              ← architecture planning
        ├── researcher.md           ← library + API research
        ├── coder.md                ← implementation
        ├── build-validator.md      ← TypeScript build + Docker config
        ├── docs-writer.md          ← Swagger + JSDoc generation
        ├── auditor.md              ← code quality audit
        ├── security-sentinel.md    ← security vulnerability audit
        ├── db-migrator.md          ← database migration safety
        ├── dependency-auditor.md   ← npm audit + license check
        ├── reviewer.md             ← final go/no-go
        ├── commit-writer.md        ← git commit message
        ├── changelog.md            ← CHANGELOG.md + semver bump
        └── pr-writer.md            ← GitHub PR description
```

---

## Troubleshooting

**Agent teams not working:**
```bash
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS   # must output: 1
claude --version                              # must be 1.0.33+
```

**Preflight fails — services not running:**
```bash
docker compose up -d postgres redis qdrant    # start required services
```

**Teammate inbox not receiving messages:**
```bash
cat ~/.claude/teams/dev-pipeline/inboxes/team-lead.json
```

**Clean up a crashed pipeline:**
```bash
rm -rf ~/.claude/teams/dev-pipeline
rm -rf ~/.claude/tasks/dev-pipeline
```

---

## Stack

Optimised for: **NestJS · Next.js · PostgreSQL · Redis · Docker · Qdrant**

The agents understand TypeORM/Prisma migrations, NestJS modules, Next.js App Router, Swagger decorators, Jest unit tests, and supertest integration tests out of the box. Adapt the agent `.md` files for other stacks.

---

## License

MIT
