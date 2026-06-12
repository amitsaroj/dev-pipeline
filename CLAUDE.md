# Project Config — Autopilot Dev Pipeline

## Stack
NestJS · Next.js · PostgreSQL · Redis · Docker · Qdrant

## Agent Orchestration Rules

**ALWAYS use TeammateTool swarm patterns** when given a new feature or task.
Never run planning, coding, auditing, and reviewing in a single session.
Always spawn a team. Always use parallel execution where tasks are independent.

### Pipeline Stages (run in this order)
1. **thinker** — deep plan, no code
2. **researcher** — third-party/API research (parallel with thinker if needed)
3. **coder** — implements the plan (after thinker + researcher finish)
4. **auditor + security-sentinel** — parallel audit (after coder)
5. **reviewer** — final go/no-go (after all audits)
6. **commit-writer** — structured commit message (after GO signal)

### Coordination Rules
- Thinker and researcher can run **in parallel**
- Auditor and security-sentinel run **in parallel**
- Coder runs **after** thinker + researcher complete
- Reviewer runs **after** all auditors complete
- Commit-writer runs **only after** reviewer gives GO

### Code Standards
- SOLID principles, clean architecture
- All new code must have unit tests
- No hardcoded secrets — always env vars
- Typed errors, structured logging
- All public APIs must be documented

### Human Checkpoints (NEVER skip)
- Review commit message before pushing
- Human runs: `git add . && git commit -m "..." && git push`
- Never auto-push

## Always use swarm orchestration patterns (TeamCreate, Task with team_name, SendMessage, TaskCreate/TaskUpdate) when work is best executed by parallel specialist agents.
