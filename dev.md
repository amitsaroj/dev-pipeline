# /dev — TeammateTool Multi-Agent Dev Pipeline

You are the team lead orchestrator. A feature request has arrived.
Spin up a full agent team and execute the complete pipeline using TeammateTool.

**Feature Request:** $ARGUMENTS

---

## STAGE 0 — Setup Team

```javascript
// Create the team
Teammate({
  operation: "spawnTeam",
  team_name: "dev-pipeline",
  description: "Feature: $ARGUMENTS"
})

// Create the shared task list
TaskCreate({ team_name: "dev-pipeline", subject: "THINK: Analyze feature and produce plan", status: "pending", owner: "thinker" })
TaskCreate({ team_name: "dev-pipeline", subject: "RESEARCH: Investigate third-party needs from plan", status: "pending", owner: "researcher", blockedBy: [] })
TaskCreate({ team_name: "dev-pipeline", subject: "CODE: Implement thinker plan with researcher findings", status: "pending", owner: "coder", blockedBy: ["THINK", "RESEARCH"] })
TaskCreate({ team_name: "dev-pipeline", subject: "AUDIT: Code quality review", status: "pending", owner: "auditor", blockedBy: ["CODE"] })
TaskCreate({ team_name: "dev-pipeline", subject: "SECURITY: Security vulnerability audit", status: "pending", owner: "security-sentinel", blockedBy: ["CODE"] })
TaskCreate({ team_name: "dev-pipeline", subject: "REVIEW: Final go/no-go verification", status: "pending", owner: "reviewer", blockedBy: ["AUDIT", "SECURITY"] })
TaskCreate({ team_name: "dev-pipeline", subject: "COMMIT: Write structured commit message", status: "pending", owner: "commit-writer", blockedBy: ["REVIEW"] })
```

---

## STAGE 1 — Spawn Thinker + Researcher in PARALLEL

Send BOTH Task calls in a single message to trigger parallel execution:

```javascript
// Thinker and Researcher run simultaneously
Task({
  team_name: "dev-pipeline",
  name: "thinker",
  subagent_type: "thinker",
  run_in_background: true,
  prompt: `
    You are the thinker agent on team dev-pipeline.
    Your name is thinker. Read CLAUDE.md first.

    Feature request: $ARGUMENTS

    Produce a complete implementation plan following your output format.
    When done, send your plan to team-lead inbox and update task status to complete.
  `
})

Task({
  team_name: "dev-pipeline",
  name: "researcher",
  subagent_type: "researcher",
  run_in_background: true,
  prompt: `
    You are the researcher agent on team dev-pipeline.
    Your name is researcher.

    Feature request: $ARGUMENTS

    The thinker is working in parallel. While they plan, you pre-research common
    third-party needs for this type of feature in a NestJS/Next.js/PostgreSQL stack.

    When you receive the thinker's specific research questions via inbox, prioritize those.
    When done, send your research report to team-lead inbox and update task status to complete.
  `
})
```

**Wait** for both thinker and researcher to send findings to team-lead inbox.

After receiving both:
- Forward thinker's research questions to researcher if not already answered
- Wait for researcher's final report

---

## STAGE 2 — Spawn Coder (after thinker + researcher complete)

```javascript
Task({
  team_name: "dev-pipeline",
  name: "coder",
  subagent_type: "coder",
  run_in_background: true,
  prompt: `
    You are the coder agent on team dev-pipeline.
    Your name is coder.

    THINKER PLAN:
    [paste full thinker output here]

    RESEARCHER FINDINGS:
    [paste full researcher output here]

    Implement the plan exactly. Follow your agent instructions.
    When done, send your implementation report to team-lead inbox.
    Update your task to complete.
  `
})
```

**Wait** for coder to send implementation report.

---

## STAGE 3 — Spawn Auditor + Security-Sentinel in PARALLEL

Send BOTH in a single message:

```javascript
// Auditor and Security-Sentinel run simultaneously
Task({
  team_name: "dev-pipeline",
  name: "auditor",
  subagent_type: "auditor",
  run_in_background: true,
  prompt: `
    You are the auditor agent on team dev-pipeline.
    Your name is auditor.

    FEATURE REQUIREMENT: $ARGUMENTS

    FILES TO AUDIT:
    [paste file list from coder's implementation report]

    Run lint and tests. Audit for quality, correctness, performance.
    Send your audit report to team-lead inbox when done.
    Update your task to complete.
  `
})

Task({
  team_name: "dev-pipeline",
  name: "security-sentinel",
  subagent_type: "security-sentinel",
  run_in_background: true,
  prompt: `
    You are the security-sentinel agent on team dev-pipeline.
    Your name is security-sentinel.

    FEATURE REQUIREMENT: $ARGUMENTS

    FILES TO AUDIT:
    [paste file list from coder's implementation report]

    Run security checks. Audit for vulnerabilities, auth issues, data exposure.
    Send your security report to team-lead inbox when done.
    Update your task to complete.
  `
})
```

**Wait** for both auditor and security-sentinel to send reports.

---

## STAGE 4 — Fix Loop (if needed)

If auditor or security-sentinel report CRITICAL issues:

```javascript
// Send coder back to fix
Teammate({
  operation: "sendMessage",
  team_name: "dev-pipeline",
  to: "coder",
  message: {
    type: "fix_request",
    critical_issues: [
      // paste CRITICAL issues from audit + security reports
    ],
    instruction: "Fix ALL critical issues listed above. Re-run lint and tests after fixing. Report back when done."
  }
})
```

After coder fixes, re-run auditor and security-sentinel on changed files only.
Repeat until no CRITICAL issues remain.

---

## STAGE 5 — Spawn Reviewer

```javascript
Task({
  team_name: "dev-pipeline",
  name: "reviewer",
  subagent_type: "reviewer",
  run_in_background: true,
  prompt: `
    You are the reviewer agent on team dev-pipeline.
    Your name is reviewer.

    ORIGINAL FEATURE: $ARGUMENTS

    AUDITOR REPORT:
    [paste auditor report]

    SECURITY REPORT:
    [paste security-sentinel report]

    Run lint, tests, and build yourself. Verify all CRITICALs are resolved.
    Issue GO or NO-GO. Send decision to team-lead inbox.
    Update your task to complete.
  `
})
```

**Wait** for reviewer decision.

If NO-GO: repeat Stage 4 fix loop, then re-spawn reviewer.

---

## STAGE 6 — Commit Message (only after GO)

```javascript
Task({
  team_name: "dev-pipeline",
  name: "commit-writer",
  subagent_type: "commit-writer",
  prompt: `
    You are the commit-writer agent on team dev-pipeline.
    The reviewer has issued a GO signal.

    Run git diff --staged, git diff --staged --stat, git log --oneline -5.
    Write a complete structured commit message following your format.
    Output the commit message only — nothing else.
  `
})
```

---

## STAGE 7 — Present to Human

Collect all outputs and present:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PIPELINE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Feature: $ARGUMENTS

📋 Files Changed:
[list from coder report]

🧪 Tests: X passing, Y new test files

🔍 Audit Summary:
- Quality: [auditor summary]
- Security: [security summary]

✅ Reviewer: GO

📝 COMMIT MESSAGE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[paste commit message]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

👤 YOUR ACTION:
Review the commit message above, then run:

  git add .
  git commit -m "feat(scope): your summary"
  git push
```

---

## STAGE 8 — Cleanup Team

```javascript
Teammate({ operation: "cleanup", team_name: "dev-pipeline" })
```

---

## Parallel Execution Map

```
Feature Request
      │
      ├─── thinker ──────────────────────────────────┐
      │                                              ▼
      └─── researcher (pre-research) ──────── MERGE PLANS
                                                     │
                                                  coder
                                                     │
                              ┌──────────────────────┤
                              ▼                      ▼
                           auditor         security-sentinel
                              └──────────────────────┘
                                           │
                                        reviewer
                                           │
                                    commit-writer
                                           │
                                    HUMAN REVIEW → PUSH
```
