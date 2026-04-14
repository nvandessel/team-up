# Debate & Consensus Protocol

## Problem

When a team needs to make a decision (architecture, library choice, design trade-offs), there's no structured way for agents to explore options together. The lead either makes the call alone or asks agents sequentially, losing the benefit of adversarial reasoning.

## Goal

A protocol where N agents can debate a topic, argue positions, challenge each other's reasoning, and converge on a recommendation. The output is a structured decision with rationale, dissenting views, and confidence level.

## Design Direction

1. **Debate mode**: A workflow the lead can invoke when a decision point arises. Could also be triggered by the user ("brainstorm the best approach for X").

2. **Participants**: Any subset of the team, or purpose-spawned debaters. Minimum 2, recommended 3 for tie-breaking.

3. **Protocol phases**:
   - **Position**: Each participant states their preferred approach with rationale
   - **Challenge**: Each participant critiques the other positions (direct peer-to-peer via SendMessage)
   - **Revise**: Participants update their positions based on challenges received
   - **Converge**: Participants vote or signal agreement. If consensus, done. If split, present options to lead/user with the debate summary.

4. **Output format**: Structured markdown with:
   - Decision (or top 2-3 options if no consensus)
   - Rationale
   - Dissenting views
   - Confidence level
   - What would change the decision

5. **Use cases**:
   - Architecture decisions during planning
   - Library/tool selection (researcher + engineer + qa)
   - Design trade-offs during spec writing
   - User-initiated brainstorming ("agents, debate whether we should use X or Y")

6. **Changes needed**:
   - New workflow section in SKILL.md (Phase 5: Workflows)
   - Possibly a new skill (`team-up:debate`) or a mode within the existing skill
   - Debate-specific agent prompt templates
   - Convergence detection logic in lead

## Context from Research

Superpowers has no equivalent — it's single-agent. This would be a unique differentiator for team-up. The peer-to-peer handoff work (#1) is a prerequisite since debate requires direct agent-to-agent communication.

## Success Criteria

- Agents can debate a topic and converge on a recommendation
- Debate output is structured and actionable
- Works for both lead-initiated and user-initiated debates
- Graceful handling of no-consensus (escalate options to user)
