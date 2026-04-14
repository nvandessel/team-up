# Debate & Consensus Protocol Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a structured debate workflow to team-up so N agents can debate a topic, challenge each other's reasoning, and converge on a recommendation.

**Architecture:** New `debater` agent definition + new "Debate / Consensus" workflow section in SKILL.md Phase 5. The lead orchestrates debate phases (Position → Challenge → Revise → Converge) using SendMessage. Participants use peer-to-peer SendMessage for the Challenge phase. No code — all markdown.

**Spec:** `docs/team-up/specs/2026-04-14-debate-consensus.md`

---

### Task 1: Create the debater agent definition

**Files:**
- Create: `agents/debater.md`

This agent participates in structured debates. It's spawned by the lead (or by the user) for a specific debate topic. Multiple instances run concurrently — one per participant.

- [ ] **Step 1: Create `agents/debater.md`**

```markdown
---
model: opus
---

# Debater

## Role
Participate in structured debates by arguing a position, challenging other positions, and revising your stance based on new arguments. You are an advocate — take your assigned position seriously and argue it rigorously, but update your view when presented with stronger reasoning.

## Boundaries
- IN: Arguing positions, critiquing other positions, revising your stance, signaling agreement or dissent
- OUT: Writing code, making final decisions (the lead or user decides), changing specs or plans

## Debate Phases

You will be told which phase you're in. Follow the phase instructions exactly.

### Position Phase
State your position on the topic. Structure your response as:
- **Position** — Your recommended approach (1-2 sentences)
- **Rationale** — Why this is the best approach (3-5 bullet points)
- **Risks** — What could go wrong with your approach (1-3 bullet points)
- **Assumptions** — What must be true for your position to hold

### Challenge Phase
You will receive other participants' positions via SendMessage. For each position you challenge:
- **Critique** — What's wrong or weak about their reasoning (be specific, not vague)
- **Counter-evidence** — Facts, trade-offs, or scenarios they missed
- **Question** — One question that would expose a flaw in their position if they can't answer it

Send challenges directly to each participant via SendMessage. Be rigorous but constructive — attack the argument, not the arguer.

### Revise Phase
After receiving challenges, update your position:
- **Original position** — Restate briefly
- **Challenges received** — Summarize the strongest arguments against your position
- **Revised position** — Your updated stance (may be unchanged, modified, or fully changed)
- **Concessions** — What you now agree the other side got right
- **Remaining disagreements** — What you still dispute and why

### Converge Phase
Signal your final stance:
- **Final position** — Your recommendation after all debate rounds
- **Confidence** — High / Medium / Low
- **Consensus signal** — AGREE (you'd accept another position), DISAGREE (you maintain yours is better), or DEFER (you'd accept the lead's/user's call)
- **If DISAGREE** — The single strongest reason you still hold your position

## When Assigned a Position
The lead may assign you a specific position to argue (devil's advocate, specific technology, etc.) or let you form your own. If assigned, argue that position genuinely — find the real strengths, don't straw-man it.

## Communication
Report to lead with status:
- **POSITION_STATED** — Position phase complete. [1-sentence summary of stance]
- **CHALLENGES_SENT** — Challenge phase complete. Challenged [N] positions.
- **REVISED** — Revision complete. Position [unchanged / modified / changed]. [1-sentence summary]
- **CONVERGED** — Final stance: [AGREE/DISAGREE/DEFER]. Confidence: [level]. [1-sentence summary]
```

- [ ] **Step 2: Verify the file follows the agent definition format**

Check against the format documented in SKILL.md: frontmatter with model, Role section, Boundaries section, Communication section with status codes. Confirm all are present.

- [ ] **Step 3: Commit**

```bash
git add agents/debater.md
git commit -m "feat: add debater agent definition for debate/consensus protocol"
```

---

### Task 2: Add the Debate / Consensus workflow to SKILL.md

**Files:**
- Modify: `skills/team-up/SKILL.md` (Phase 5: Workflows section, after the Pipeline workflow ending at line 98)

The workflow tells the lead how to run a debate. It covers: when to use it, how to spawn participants, how to run each phase, how to detect convergence, and how to produce the structured output.

- [ ] **Step 1: Add the Debate / Consensus workflow**

Insert the following after the Pipeline workflow section (after line 98, before `## Phase 6: Cleanup`) in `skills/team-up/SKILL.md`:

```markdown
### Debate / Consensus

Use when the team needs to make a decision: architecture choices, library selection, design trade-offs, or when the user asks agents to debate a topic.

**When to invoke:**
- You (the lead) hit a decision point during planning or implementation
- The user asks: "debate whether we should...", "brainstorm the best approach for..."
- A spec or plan has multiple viable approaches and the trade-offs aren't obvious

**Participants:**
- Minimum 2, recommended 3 (odd number for tie-breaking)
- Use existing team members when their expertise is relevant (e.g., researcher + engineer for library selection)
- Spawn dedicated `debater` agents for pure argumentation tasks
- Optionally assign positions: "argue for X" — useful for devil's advocate or ensuring all options get a fair hearing

**Running the debate:**

1. **Setup** — Frame the topic clearly. Tell each participant:
   - The question being debated
   - Any constraints (timeline, budget, compatibility requirements)
   - Their assigned position (if any), or "form your own position"
   - The phase they're starting: Position

2. **Position phase** — Send topic to all participants via SendMessage. Wait for all to report `POSITION_STATED`. Collect their positions.

3. **Challenge phase** — Share all positions with all participants. Instruct them to send challenges directly to each other participant via peer-to-peer SendMessage. Wait for all to report `CHALLENGES_SENT`.

4. **Revise phase** — Instruct participants to revise their positions based on challenges received. Wait for all to report `REVISED`. Collect revised positions.

5. **Converge phase** — Instruct participants to send their final stance. Wait for all to report `CONVERGED`. Collect convergence signals.

**Determining outcome:**

- **Consensus** (all AGREE or all share the same final position): Adopt the consensus position. Summarize and proceed.
- **Majority** (most AGREE, some DISAGREE): Adopt the majority position. Note dissenting view in the summary.
- **Split** (no clear majority, or critical DISAGREE with high confidence): Present the top options to the user with the debate summary. Do not decide for them.

**Debate output:**

After the debate concludes, produce a structured summary (include in status note or present to user):

```
## Debate: [Topic]
Date: YYYY-MM-DD
Participants: [list]

### Decision
[The chosen approach, or "Escalated to user — no consensus"]

### Rationale
[Why this approach won — key arguments that carried the debate]

### Dissenting Views
[Positions that disagreed and their strongest argument]

### Confidence
[High / Medium / Low — based on convergence signals]

### What Would Change This Decision
[Conditions under which this decision should be revisited]
```

**Guidelines:**
- Don't let debates run forever. Position → Challenge → Revise → Converge is one round. If no convergence after one round, escalate to the user with the summary.
- Debates are advisory. The user (or you, as lead, for operational decisions) makes the final call.
- For user-initiated debates, present the output directly to the user. For lead-initiated debates, use the outcome to inform your next action.
```

- [ ] **Step 2: Verify the edit integrates cleanly**

Read SKILL.md to confirm:
- The new workflow sits in Phase 5 alongside "Code Review Loop" and "Pipeline"
- No duplicate headings or broken markdown structure
- Phase 6 (Cleanup) still follows Phase 5

- [ ] **Step 3: Commit**

```bash
git add skills/team-up/SKILL.md
git commit -m "feat: add debate/consensus workflow to team lead playbook"
```

---

### Task 3: Final review

- [ ] **Step 1: Cross-check against spec**

Read `docs/team-up/specs/2026-04-14-debate-consensus.md` and verify every requirement is covered:
- Protocol phases (Position, Challenge, Revise, Converge) — covered in both agent def and workflow
- Output format (Decision, Rationale, Dissenting views, Confidence, What would change) — covered in workflow
- Use cases (architecture decisions, library selection, design trade-offs, user-initiated) — covered in "When to invoke"
- Lead-initiated and user-initiated — covered in Guidelines
- Graceful no-consensus handling — covered in "Determining outcome" (Split → escalate)

- [ ] **Step 2: Verify no spec gaps remain**

If any gaps found, fix them in the relevant file before proceeding.
