# team-up

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](.claude-plugin/marketplace.json)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-plugin-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

A Claude Code plugin that orchestrates persistent agent teams. Give it a task, and it assembles a team of specialist agents — spec writers, engineers, reviewers, researchers — coordinates their work through a structured pipeline, and delivers reviewed, tested code.

## Why

Running one agent at a time works for small tasks. But for anything that needs a spec, a plan, implementation, and review, you end up doing the coordination yourself. team-up handles that coordination: it reads the task, proposes a team, assembles it, delegates work, manages handoffs, and makes sure nothing falls through the cracks.

## How It Works

When you invoke `/team-up`, Claude becomes the team lead and runs a 7-phase workflow:

1. **Assess** — Read the task, determine what roles are needed
2. **Propose** — Present team composition for your approval
3. **Assemble** — Spawn specialist agents via `TeamCreate`
4. **Coordinate** — Delegate work, handle handoffs, unblock agents
5. **Workflows** — Run pipelines (spec → plan → implement → review)
6. **Cleanup** — Verify no tech debt, all docs updated, quality gates passed
7. **Status** — Write a status note tracking what was done

## Agent Roles

| Agent | Job |
|-------|-----|
| **Spec Writer** | Turns a problem into a clear, testable specification |
| **Spec Reviewer** | Validates specs for completeness, consistency, and clarity |
| **Plan Writer** | Converts specs into bite-sized implementation tasks |
| **Plan Reviewer** | Verifies plans match specs and are buildable |
| **Researcher** | Validates APIs, libraries, and patterns are current (on-call) |
| **Engineer** | Implements code with TDD on feature branches |
| **QA Reviewer** | Two-stage review: spec compliance, then code quality |

Not every task needs every role. The lead assembles only what's needed — a bug fix might just need a researcher + engineer, while a new feature gets the full pipeline.

## Communication Protocol

All agents report status to the lead using structured codes:

- **DONE** — Work complete, ready for next step
- **DONE_WITH_CONCERNS** — Complete, but flagging an issue
- **NEEDS_CONTEXT** — Blocked on missing information
- **BLOCKED** — Stuck, needs intervention

## Installation

### From GitHub

```
/plugins add nvandessel/team-up
```

### From local directory

```
/plugins add /path/to/team-up
```

## Usage

Say "team up" or invoke `/team-up` with a description of what you want to build:

```
/team-up Build a REST API for managing bookmarks with tags and full-text search
```

Or point it at an existing spec:

```
/team-up Implement the spec in ~/notes/inbox/specs/2026-03-22-bookmark-api.md
```

The lead will propose a team, wait for your approval, then run the pipeline.

## Customizing Agents

Agent definitions live in `agents/` as markdown files. Each defines:

- **Role** — What the agent does
- **Boundaries** — What's in and out of scope
- **Process** — Step-by-step workflow
- **Quality standards** — Checklist before reporting done
- **Communication** — How to report status

To add a custom agent, create a new `.md` file in `agents/` following the same format:

```markdown
---
model: opus
---

# Role Name

## Role
What this agent does.

## Boundaries
- IN: What's in scope
- OUT: What's out of scope

## Process
1. Step one
2. Step two

## Quality Standards
- [ ] Check one
- [ ] Check two

## Communication
- DONE — [summary]
- NEEDS_CONTEXT — [question]
- BLOCKED — [problem]
```

The lead will discover and use any agent definitions it finds.

## Design Principles

- **User stays in control** — The lead proposes, you approve. No merges without sign-off.
- **No tech debt** — Work isn't done until cleanup is done.
- **Feature branches only** — Never commits to main.
- **Artifacts stay out of the repo** — Specs and plans go to `~/notes/inbox/`, not the codebase.
- **Escalate, don't loop** — When stuck, the lead asks you rather than retrying forever.
- **Right-size the team** — Only spin up roles the task actually needs.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- The `TeamCreate` and `Agent` tools (available in Claude Code)

## Acknowledgments

The workflow and agent coordination patterns in this plugin are inspired by [superpowers](https://github.com/obra/superpowers) by Jesse Vincent, which introduced structured skill-based workflows for Claude Code — brainstorming, TDD, plan writing, code review, and more. team-up builds on those ideas by turning them into a multi-agent team with persistent specialist roles.

## License

MIT
