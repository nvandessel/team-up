# Superpowers-Inspired Improvements

## Problem

Superpowers (v5.0.7) has mature patterns that team-up could adopt to improve quality and developer experience. This workstream applies specific learnings — not wholesale copying.

## Improvements to Apply

### 1. Hard-Gate Process Enforcement

**What superpowers does**: Skills include explicit HARD-GATE blocks that prevent skipping phases (e.g., no code before brainstorming approval).

**Apply to team-up**: Add explicit gates in the lead playbook:
- Spec must be reviewer-approved before plan-writer starts
- Plan must be reviewer-approved before engineer starts
- Engineer must have passing tests before QA review
- These are currently implicit in the pipeline ordering but should be explicit and enforced

### 2. TDD Iron Law in Engineer Agent

**What superpowers does**: "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST. If you wrote code first, DELETE IT and start over."

**Apply to team-up**: Strengthen the engineer agent's TDD section with this same hard discipline. Currently the engineer agent says "TDD cycle" but doesn't enforce the iron law.

### 3. Parallel Agent Dispatch

**What superpowers does**: `dispatching-parallel-agents` skill explicitly handles independent tasks in parallel (e.g., 3 test failures across different files).

**Apply to team-up**: Add guidance in the lead playbook for when to run agents in parallel vs. sequentially. Currently the pipeline is linear even when stages could overlap (e.g., researcher can investigate while plan-writer works).

### 4. Bite-Sized Task Granularity

**What superpowers does**: Plans break work into "2-5 minute tasks" with complete file paths and code snippets.

**Apply to team-up**: Add this constraint to plan-writer's quality checklist. Tasks should be independently testable and completable in one focused session.

### 5. Visual Brainstorming Companion (Evaluate)

**What superpowers does**: Browser-based mockup server with `.events` file for non-blocking visual interaction during brainstorming.

**Apply to team-up**: Evaluate whether this makes sense for team-up's spec-writer phase. May be out of scope — team-up is about agent orchestration, not UI design. Note for consideration but don't force it.

## Changes Needed

- Update SKILL.md with hard-gate markers and parallel dispatch guidance
- Update engineer agent with TDD iron law
- Update plan-writer agent with bite-sized task constraint
- Evaluate visual companion integration (may defer)

## Success Criteria

- Lead playbook has explicit gates between phases
- Engineer agent enforces TDD iron law
- Lead knows when to parallelize agents
- Plan-writer produces bite-sized, independently testable tasks
