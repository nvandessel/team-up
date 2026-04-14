# Agent Lifecycle Robustness

## Problem

Observed issues with agent lifecycle management:

1. **Incomplete cleanup**: Lead sometimes doesn't clean up all agents when work finishes, or cleanup commands fail silently
2. **Silent startup failures**: Agents fail to start (e.g., Claude API changes, tool availability changes) and the lead doesn't detect the failure — sits waiting for a response that never comes
3. **Version sensitivity**: When Claude updates, agent spawning may fail in new ways the lead doesn't anticipate

## Goal

Make the lead's lifecycle management defensive — verify agents actually started, detect failures, clean up reliably.

## Design Direction

### 1. Startup Verification

After spawning each agent, the lead should verify it's alive:
- Send a "ready check" message via SendMessage
- Expect an acknowledgment within a reasonable window
- If no response, report the failure to the user rather than silently waiting
- Consider a startup handshake protocol in agent definitions

### 2. Graceful Shutdown Protocol

When work completes or user requests wind-down:
- Send explicit shutdown messages to all active agents
- Verify each agent has stopped (check via TaskList or equivalent)
- Report any agents that didn't shut down cleanly
- Add a "force cleanup" fallback

### 3. Health Monitoring

During long-running work:
- Lead periodically checks that agents are responsive
- If an agent goes silent for too long, escalate to user
- Track which agents are expected to be active vs. idle

### 4. Error Recovery

When an agent fails mid-work:
- Detect the failure (timeout, error status, unexpected silence)
- Attempt to restart with the same context
- If restart fails, report to user with what was lost
- Consider checkpointing agent progress for recovery

## Changes Needed

- Update SKILL.md Phase 3 (Assemble) with startup verification
- Update SKILL.md Phase 6 (Cleanup) with graceful shutdown protocol
- Add health monitoring guidance to Phase 4 (Coordinate)
- Add error recovery patterns
- Update agent definitions with startup handshake expectations

## Success Criteria

- Lead detects agent startup failures instead of silently waiting
- All agents are cleaned up when work completes
- Long-running silent agents are detected and escalated
- Users get clear error messages when lifecycle issues occur
