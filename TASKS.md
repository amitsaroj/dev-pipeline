# Pipeline Task Tracker

This file is the persistent memory of the dev-pipeline project.
Every future agent and contributor should read this first to understand what exists,
what is in progress, and what still needs to be built.

**Last updated:** 2026-06-13
**Total agents:** 17 (14 pipeline agents + model-router + briefing + watchdog)
**Pipeline version:** 2.1.0

---

## ✅ Completed

### Core Pipeline Agents
- [x] **thinker** — deep architecture planning, API contracts, risk analysis, no code
- [x] **researcher** — npm/GitHub package research, CVE checks, license compatibility, usage examples
- [x] **coder** — full implementation: TypeScript, unit tests, integration tests, env/docker updates
- [x] **build-validator** — `tsc --noEmit`, `npm run build`, circular deps, `docker-compose config` fast-fail gate
- [x] **docs-writer** — adds missing Swagger decorators, JSDoc, `.env.example` descriptions
- [x] **auditor** — code quality, SOLID, test coverage, N+1 queries, env/docker config completeness
- [x] **security-sentinel** — injection, auth/authz, data exposure, multi-tenant isolation, CORS/headers
- [x] **db-migrator** — PostgreSQL migration generation, reversibility check, FK index coverage
- [x] **dependency-auditor** — `npm audit`, license compliance (no GPL in SaaS), phantom deps, outdated packages
- [x] **reviewer** — final go/no-go: re-runs lint/test/build, verifies all CRITICALs resolved
- [x] **commit-writer** — conventional commit message from `git diff` (not `--staged`)
- [x] **changelog** — semver bump determination, Keep a Changelog format, `package.json` version bump
- [x] **pr-writer** — full GitHub PR description: summary, test plan, deploy notes, API examples
- [x] **preflight** — env health gate: git state, Node/npm, Docker, services, `.env`, baseline build/tests
- [x] **model-router** — detects available models (Claude → OpenAI → Cursor → Ollama), sets pipeline strategy
- [x] **briefing** — pre-pipeline requirements gathering: asks scope/constraints/acceptance criteria, saves FEATURE_BRIEF.md
- [x] **watchdog** — background health monitor; checks all agents every N minutes, pings stuck agents, escalates after 3 pings

### Orchestrator
- [x] **`/dev` command** (`commands/dev.md`) — full 8-stage orchestrator with human checkpoints, fix loop (max 3 retries), parallel execution map

### Infrastructure
- [x] **`CLAUDE.md`** — pipeline config with stack, agent stages, coordination rules, code standards
- [x] **`model-config.md`** — model priority chain, Ollama setup, anti-hallucination rules
- [x] **`scripts/check-models.sh`** — detects and reports available AI models
- [x] **`scripts/ollama-pipeline.sh`** — complete standalone pipeline for users without Claude access
- [x] **`scripts/watchdog.sh`** — standalone background health monitor with --status, --stop, --interval flags

### Critical Gaps Fixed (v1 → v2)
- [x] commit-writer was reading `git diff --staged` (empty before `git add`) → fixed to `git diff`
- [x] no human checkpoint before coder started → added plan approval gate
- [x] infinite fix loop → capped at 3 retries with human escalation
- [x] no feature branch creation → added `git checkout -b feat/<slug>` at Stage 0
- [x] no DB migration agent → added db-migrator
- [x] researcher had no `bash` tool → added bash to tools list
- [x] auditor/security-sentinel both ran tests redundantly → security-sentinel now grep-only
- [x] no Docker/env config validation → added to auditor + infra checks
- [x] no integration test requirement → added to coder + auditor
- [x] no PR description → added pr-writer
- [x] no CHANGELOG agent → added changelog
- [x] build errors only caught at reviewer stage → added build-validator (Stage 2.5)
- [x] no Swagger/JSDoc generation → added docs-writer
- [x] npm audit never ran (security-sentinel only grepped) → added dependency-auditor
- [x] no repo health check before starting → added preflight
- [x] duplicate nested folder structure → flattened to root `.claude/`
- [x] no model fallback for users without Claude access → added model-router + Ollama pipeline
- [x] no requirements gathering before coding → added briefing agent (saves FEATURE_BRIEF.md)
- [x] no visibility into stuck/failed agents during long runs → added watchdog (background, configurable interval)

---

## 🔄 In Progress

- [ ] **Ollama anti-hallucination tuning** — Prompts in `ollama-pipeline.sh` are first-pass; may need tuning per model family (llama vs deepseek vs mistral)
- [ ] **Model-specific agent variants** — Individual `.md` files could have Ollama-optimized prompt sections

---

## 📋 Planned (Next Sprint)

### New Agents
- [ ] **`performance-profiler`** — Runs k6/artillery load tests, checks EXPLAIN ANALYZE on new queries, benchmarks Redis ops. Runs after reviewer, optional stage.
- [ ] **`rollback-planner`** — Documents exact rollback steps: migration down command, feature flag toggle, data cleanup. Runs post-GO in parallel with commit-writer.
- [ ] **`api-contract-validator`** — Compares thinker's API contracts against actual implementation. Checks endpoint signatures, request/response shapes match OpenAPI spec.
- [ ] **`test-gap-finder`** — Adversarial tester: reads spec + implementation, finds edge cases not covered by existing tests. Runs parallel with auditor.
- [ ] **`cache-validator`** — Validates Redis cache invalidation logic when entities change, checks TTL settings, flags stale cache scenarios.
- [ ] **`qdrant-validator`** — Validates Qdrant collection setup, embedding strategy, vector deletion on entity update, index settings. Only runs if Qdrant is in the stack.

### Orchestrator Improvements
- [ ] **Cost estimator** — Before spawning agents, estimate token cost and confirm with user if above threshold
- [ ] **Stage skip flags** — Allow `/dev --skip-docs --skip-changelog "feature"` for faster runs on small changes
- [ ] **Resume support** — If pipeline crashes mid-run, resume from last completed stage without re-running earlier agents
- [ ] **Parallel model routing** — Run model-router in parallel with thinker/researcher instead of as a blocking gate

### Infrastructure
- [ ] **`scripts/resume-pipeline.sh`** — Resume a crashed Ollama pipeline from last saved checkpoint
- [ ] **`scripts/cost-estimate.sh`** — Estimate Claude API cost before running the pipeline
- [ ] **GitHub Actions workflow** — `.github/workflows/pipeline.yml` that runs on PR creation
- [ ] **Docker image for Ollama pipeline** — Pre-configured Docker image with Ollama + recommended models
- [ ] **VS Code extension stub** — Surface `/dev` command from VS Code command palette

---

## ⏳ Pending / Known Gaps

These gaps were identified in analysis but are lower priority or need more design work:

| Gap | Description | Severity | Notes |
|-----|-------------|----------|-------|
| **Backwards compatibility check** | No agent validates feature doesn't break existing client code paths | MEDIUM | Would need access to client code |
| **i18n coverage** | No check for hardcoded strings missing from i18n files | MEDIUM | Only relevant for multi-language apps |
| **Monitoring/alerting setup** | No check that new code paths have monitoring configured | MEDIUM | Needs Datadog/Grafana/Prometheus awareness |
| **Feature flag completeness** | No check for feature flags on risky changes | MEDIUM | Needs feature flag system awareness |
| **Backup/restore validation** | No check that migrations are safe for restore-from-backup scenarios | MEDIUM | Complex to implement generically |
| **Cost impact** | No check that new DB queries won't spike cloud costs | HIGH | Needs query plan analysis |
| **Rate limit tuning** | security-sentinel flags missing limits but not whether chosen limits are appropriate | LOW | Opinion-based, hard to automate |
| **PR review readiness check** | No cross-validation of PR description against actual diff | LOW | pr-writer is self-contained |

---

## 🚫 Decided Against

These were considered and explicitly rejected:

| Item | Reason |
|------|--------|
| Auto-push to remote | Safety: human must always review before push |
| Auto-merge PRs | Safety: requires human review |
| Modify existing tests | Scope creep: agents only add, never remove tests |
| Rewrite coder's architecture | Agents have separation of concerns; only coder writes code |
| GPT-3.5/small models for complex stages | Quality: thinker/coder/reviewer need high capability models |

---

## 📊 Pipeline Coverage Matrix

| Delivery Phase | Agent(s) | Coverage |
|---|---|---|
| Requirements gathering | briefing | ✅ Full |
| Env validation | preflight | ✅ Full |
| Model selection | model-router | ✅ Full |
| Agent health monitoring | watchdog | ✅ Full |
| Architecture planning | thinker | ✅ Full |
| Library research | researcher | ✅ Full |
| Implementation | coder | ✅ Full |
| Build validation | build-validator | ✅ Full |
| API documentation | docs-writer | ✅ Full |
| Code quality | auditor | ✅ Full |
| Security | security-sentinel | ✅ Full |
| Database safety | db-migrator | ✅ Full |
| Dependency security | dependency-auditor | ✅ Full |
| Final review | reviewer | ✅ Full |
| Commit message | commit-writer | ✅ Full |
| Release notes | changelog | ✅ Full |
| PR description | pr-writer | ✅ Full |
| Performance testing | — | ❌ Planned |
| Rollback planning | — | ❌ Planned |
| Cache invalidation check | — | ❌ Planned |
| Qdrant validation | — | ❌ Planned |
| API contract validation | — | ❌ Planned |
| Cost impact analysis | — | ⏳ Pending |
| Monitoring setup check | — | ⏳ Pending |

---

## 🔖 Agent Quick Reference

```
briefing         → Pre-pipeline (ask user: scope/constraints/acceptance criteria)
model-router     → Pre-gate   (detect Claude/OpenAI/Cursor/Ollama)
preflight        → Gate       (env + repo health check)
watchdog         → Background (entire pipeline — checks every N minutes)
thinker          → Stage 1  ∥ researcher
researcher       → Stage 1  ∥ thinker
coder            → Stage 2  (after human approves plan)
build-validator  → Stage 2.5 ∥ docs-writer
docs-writer      → Stage 2.5 ∥ build-validator
auditor          → Stage 3  ∥ security-sentinel ∥ db-migrator ∥ dependency-auditor
security-sentinel→ Stage 3  ∥
db-migrator      → Stage 3  ∥
dependency-auditor→Stage 3  ∥
reviewer         → Stage 4  (after all Stage 3 complete)
commit-writer    → Stage 5  ∥ changelog ∥ pr-writer
changelog        → Stage 5  ∥
pr-writer        → Stage 5  ∥
```
