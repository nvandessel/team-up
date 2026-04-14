# Peer-to-Peer Agent Handoffs

## Problem

Currently all communication routes through the lead agent. When plan-writer finishes, it reports DONE to the lead, the lead reads the full output, then forwards it to plan-reviewer. This:

- Bloats the lead's context with content it doesn't need to deeply understand
- Adds latency (two hops instead of one)
- Prevents natural iteration loops between peers (writer ↔ reviewer must round-trip through lead)

## Goal

Agents that naturally work in pairs (writer/reviewer, engineer/qa) should communicate directly. The lead stays informed of outcomes but doesn't need to be the message bus.

## Design Direction

1. **Peer pairs**: Define which agents are expected to iterate directly:
   - spec-writer ↔ spec-reviewer
   - plan-writer ↔ plan-reviewer
   - engineer ↔ qa-reviewer
   - Any agent ↔ researcher (on-demand)

2. **Protocol**: When peer A finishes a draft, it sends directly to peer B via SendMessage. Peer B reviews and either approves (reports to lead: "plan approved") or sends feedback directly back to peer A. They iterate until the reviewer is satisfied.

3. **Lead oversight**: Lead gets notified of the final outcome only — not every intermediate draft. Lead can still intervene if iterations exceed a threshold or if an agent reports BLOCKED.

4. **Changes needed**:
   - Update SKILL.md Phase 4 (Coordinate) to describe peer-to-peer patterns
   - Update each agent definition's Communication section with peer awareness
   - Add peer name injection into agent prompts at spawn time
   - Define iteration limits (e.g., max 3 rounds before escalating to lead)

## Context from Research

Superpowers avoids this problem by using fresh subagents per task (no persistent relationships). Team-up's strength is persistent specialists — peer-to-peer handoffs preserve that strength while reducing lead overhead.

## Success Criteria

- Writer/reviewer pairs iterate directly without lead forwarding each draft
- Lead's context only grows with final outcomes, not intermediate drafts
- Iteration limits prevent infinite loops
- Lead can still intervene when needed
