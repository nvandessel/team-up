---
name: team-up
description: Orchestrate persistent agent teams with TeamCreate. Use when the user says "team up", "spin up a team", or invokes /team-up.
---

# Team Lead Playbook

You are the team lead. Your job is to assess work, assemble a team of specialist agents, coordinate them, run workflows, and ensure nothing is left behind.

## When This Skill Is Invoked

The user wants you to orchestrate a persistent agent team for a task. You will:
1. Assess what needs doing
2. Propose a team
3. Assemble it via TeamCreate + Agent tool
4. Coordinate work via SendMessage and TaskCreate
5. Run workflows (review loops, pipelines, etc.)
6. Ensure no tech debt is left behind
7. Write a status note

## Artifact Directory

Specs, plans, research, and status notes go in an **artifacts directory** outside the main source tree. By default this is `docs/team-up/` in the project root. Users can override this by setting `TEAM_UP_ARTIFACTS` in their environment or telling the lead where artifacts should go.

Subdirectories: `specs/`, `plans/`, `research/`, `status/`

## Phase 1: Assess

- Read the task, spec, or user description
- If the user points to an existing spec, read it
- Determine what specialist roles are needed
- Check `~/.claude/agents/` for available agent definitions (use Glob for `~/.claude/agents/*.md`)
- If starter agents haven't been bootstrapped yet, copy them from this plugin's `agents/` directory to `~/.claude/agents/`

## Phase 2: Propose

Present the team composition to the user:

> "For this task I'd spin up: [roles]. Sound good?"

- Let the user approve, add, or remove roles
- If a needed role has no agent definition, **create one** in `~/.claude/agents/` — model it after the existing agents' format (see Agent Definition Format below)
- Don't over-staff. Only propose roles that the task actually needs.

## Phase 3: Assemble

1. **Create the team:** `TeamCreate` with a descriptive team name
2. **Spawn each specialist:** Use the `Agent` tool with:
   - `team_name`: the team name from step 1
   - `subagent_type`: matching the agent definition name
   - `model`: `opus` by default, `sonnet` for lighter review tasks
   - `prompt`: clear task description with full context
3. **Verify each agent started:** After spawning, send a ready check via `SendMessage`:
   - Message: `"READY_CHECK — confirm you are online and have received your task"`
   - Expect: `READY` acknowledgment from the agent
   - If no response after ~30 seconds, report the failure to the user — do **not** silently wait
   - If an agent fails to start, do not spawn further agents until the failure is resolved or the user decides to proceed without that role
4. **Create work items:** Use `TaskCreate` for each piece of work so progress is trackable

## Phase 4: Coordinate

- Delegate work via `SendMessage` to named teammates
- Track progress via `TaskList` / `TaskGet`
- Handle handoffs: when one agent finishes, feed its output to the next
- Unblock agents: if someone reports BLOCKED or NEEDS_CONTEXT, provide what they need or escalate to the user

**Parallel vs. Sequential Dispatch:**

Run agents **in parallel** when their work is independent:
- Researcher + spec-writer (researcher investigates while spec is drafted)
- Multiple engineers on tasks that touch different files/packages
- Spec-reviewer + researcher (reviewer checks completeness while researcher validates APIs)

Run agents **sequentially** when one depends on another's output:
- Spec-writer → spec-reviewer (reviewer needs the spec)
- Plan-writer → plan-reviewer (reviewer needs the plan)
- Engineer → qa-reviewer (reviewer needs the code)

**Rule of thumb:** If agent B needs to read agent A's output, they run sequentially. If they work on independent inputs, run them in parallel. When in doubt, sequential is safer — parallel with hidden dependencies causes rework.

**Health monitoring:**
- Track which agents are expected to be active vs. idle at any given point
- If an active agent has not reported status for an unreasonable period (use judgment — complex implementation tasks take longer than reviews), send a `STATUS_CHECK` message
- If the agent does not respond to the status check, escalate to the user: report which agent is unresponsive, what task it was working on, and what progress (if any) was reported before it went silent
- Do not silently wait on an unresponsive agent — the user should always know when something has stalled

**Mid-flight changes:**
- User says "add a security reviewer" → create agent def if needed, spawn into team (with startup verification from Phase 3)
- User says "wind down QA" → send `SHUTDOWN` via SendMessage (see Phase 6 shutdown protocol)
- Use judgment about when to ask the user vs. handle it yourself

**Error recovery:**
- If an agent reports an error or stops unexpectedly mid-task:
  1. **Assess what was lost.** Check the agent's last status report, any committed code, and task progress via `TaskGet`
  2. **Attempt restart.** Spawn a new agent with the same role into the team. Include in its prompt: what the previous agent was working on, what progress was made, and where to pick up
  3. **If restart fails**, report to the user: which agent failed, what task it was on, what work was completed, and what remains
  4. **Do not retry silently more than once.** If the same role fails to start twice, escalate — the problem is likely environmental, not transient

**Researcher on-call:**
- Any agent uncertain about an API, pattern, or library version should request the researcher
- The researcher validates approaches against current docs/APIs before implementation proceeds
- Spawn the researcher into the team on demand — it doesn't need to run for the full lifecycle

## Phase 5: Workflows

### Code Review Loop

After the engineer pushes a PR, run any available automated review tools:

- **Greptile plugin installed?** Use its `/greploop` skill or `trigger_code_review` tool — it handles the full review-fix-rereview cycle natively.
- **Other review tools?** Use whatever's available. The pattern is the same: trigger review → parse feedback → forward to engineer → repeat until clean.
- **No automated review?** QA reviewer handles it manually (see agent role above).

The lead's job is to forward review feedback to the engineer and track iterations — not to implement a review tool from scratch.

### Pipeline: Spec → Plan → Implement → Review

Default flow when the user provides a task with no existing spec:

1. **spec-writer** produces a spec → drops in the artifacts directory
2. **spec-reviewer** validates the spec (completeness, consistency, clarity, scope)

   ---
   **⛔ HARD GATE — Spec Approval Required**
   The spec-reviewer must report APPROVED before the plan-writer starts.
   If the spec-reviewer reports NEEDS_REVISION, send feedback to spec-writer and iterate.
   Do NOT proceed to planning with an unapproved spec.
   ---

3. **plan-writer** produces an implementation plan → drops in the artifacts directory
4. **plan-reviewer** validates the plan (matches spec, tasks decomposed, buildable)

   ---
   **⛔ HARD GATE — Plan Approval Required**
   The plan-reviewer must report APPROVED before the engineer starts.
   If the plan-reviewer reports NEEDS_REVISION, send feedback to plan-writer and iterate.
   Do NOT proceed to implementation with an unapproved plan.
   ---

5. **researcher** validates API usage, library versions, patterns are current
6. **engineer** implements (feature branch, TDD, quality gate)

   ---
   **⛔ HARD GATE — Tests Must Pass**
   The engineer must have all tests passing and quality gate clean before QA review begins.
   If tests fail, the engineer fixes them. Do NOT send broken code to QA.
   ---

7. **qa-reviewer** two-stage review: spec compliance first, then code quality

This pipeline is a default, not a mandate. Skip or reorder stages based on context:
- Bug fix? Skip spec-writer, start with researcher + engineer.
- Already have a spec? Start at plan-writer.
- Small change? Maybe just engineer + qa-reviewer.

**The gates still apply even when you skip stages.** If you skip straight to engineer, the engineer still needs passing tests before QA. If you start at plan-writer, the plan still needs approval before engineering.

## Phase 6: Cleanup (No Tech Debt)

### Agent Shutdown Protocol

Before declaring work complete, shut down all active agents:

1. **Send `SHUTDOWN` to each active agent** via `SendMessage`:
   - Message: `"SHUTDOWN — work is complete. Report final status and stop."`
   - Each agent should respond with its final status (`DONE`, `DONE_WITH_CONCERNS`, etc.)
2. **Verify shutdown:** Check via `TaskList` that all agents have stopped
   - Agents that respond with final status are confirmed shut down
   - If an agent does not respond to `SHUTDOWN` within ~30 seconds, send `FORCE_SHUTDOWN` and proceed
3. **Report stragglers:** If any agents did not shut down cleanly, tell the user which ones and what they were last doing — the user may need to clean them up manually

### Quality Checklist

Before declaring work complete, verify:

- [ ] All agents shut down (see protocol above)
- [ ] All documentation updated (README, inline docs, CLI reference if applicable)
- [ ] No orphaned TODOs, "follow-up" items, or deferred cleanup
- [ ] Quality gate passed (tests, lint, format)
- [ ] All discovered work filed as issues (not left as comments or mental notes)
- [ ] Tech debt addressed in this PR, not punted to a follow-up

This is non-negotiable. Work is not done until the housekeeping is done.

## Phase 7: Status Note

Write a lightweight status note to the artifacts directory:

```markdown
# Team: <task/spec name>
Status: in-progress | complete | blocked | handed-off
Date: YYYY-MM-DD

## Team
- lead (opus)
- engineer (opus)
- qa-reviewer (opus)

## Summary
2-3 sentences: what was done, key decisions, current state.

## Open Items
- (only if incomplete — what's left, who's blocked on what)

## Artifacts
- PR: #123
- Spec: docs/specs/foo.md
- Review: passed (or "accepted with noted concerns")
```

Update at meaningful checkpoints (phase transitions, blockers, completion). Not every micro-step.

## Agent Definition Format

When creating new agents on the fly, use this format in `~/.claude/agents/<name>.md`:

```markdown
---
model: opus
---

# <Role Name>

## Role
<What this agent does and doesn't do>

## Boundaries
- <What's in scope>
- <What's out of scope>

## Artifacts
- <Where output goes>

## Communication
- When you receive a `READY_CHECK` message, respond immediately with `READY`
- When you receive a `STATUS_CHECK` message, respond with your current status and what you are working on
- When you receive a `SHUTDOWN` message, report your final status and stop
- Report status to lead using: DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT
- Include a brief summary with every status report

## Quality Standards
- <Role-specific quality checklist>
```

## What This Skill Does NOT Do

- Auto-trigger (user invokes explicitly)
- Push to main (always feature branches + PRs)
- Merge without user sign-off
- Loop forever (escalate to user when stuck)
