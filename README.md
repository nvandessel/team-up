# team-up

> Early days — feedback and bug reports welcome. [Open an issue](https://github.com/nvandessel/team-up/issues).

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-green.svg)](.claude-plugin/marketplace.json)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-plugin-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

> Orchestrate persistent agent teams in Claude Code. Give it a task, and it assembles
> specialist agents — spec writers, engineers, reviewers, researchers — coordinates their
> work through a structured pipeline, and delivers reviewed, tested code.

---

- [Why team-up](#why-team-up)
- [How it works](#how-it-works)
- [Agent roles](#agent-roles)
- [Install](#install) · [Usage](#usage)
- [Customizing agents](#customizing-agents)
- [Design principles](#design-principles)
- [Acknowledgments](#acknowledgments)

---

## Why team-up

- **Stop being the coordinator** — for anything that needs a spec, a plan, implementation, and review, you end up managing the handoffs yourself. team-up does that for you.
- **Right-size the team** — a bug fix gets a researcher + engineer. A new feature gets the full pipeline. The lead assembles only what the task needs.
- **No tech debt left behind** — work isn't done until cleanup is done. Quality gates, docs, and issue tracking are enforced before completion.
- **You stay in control** — the lead proposes, you approve. No merges without sign-off. When stuck, it asks you rather than looping forever.

## How it works

When you invoke `/team-up`, Claude becomes the team lead and runs a 7-phase workflow:

1. **Assess** — Read the task, determine what roles are needed
2. **Propose** — Present team composition for your approval
3. **Assemble** — Spawn specialist agents via `TeamCreate`
4. **Coordinate** — Delegate work, handle handoffs, unblock agents
5. **Workflows** — Run pipelines (spec → plan → implement → review)
6. **Cleanup** — Verify no tech debt, all docs updated, quality gates passed
7. **Status** — Write a status note tracking what was done

## Agent roles

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

### Communication protocol

All agents report status to the lead using structured codes:

- **DONE** — Work complete, ready for next step
- **DONE_WITH_CONCERNS** — Complete, but flagging an issue
- **NEEDS_CONTEXT** — Blocked on missing information
- **BLOCKED** — Stuck, needs intervention

## Install

1. Open Claude Code settings
2. Go to the **Marketplace** tab
3. Search for **team-up**
4. Click install

Or from a local directory:

```
/plugins add /path/to/team-up
```

### Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [Agent Teams enabled](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)

## Usage

Say "team up" or invoke `/team-up` with a description of what you want to build:

```
/team-up Build a REST API for managing bookmarks with tags and full-text search
```

Or point it at an existing spec:

```
/team-up Implement the spec in docs/team-up/specs/2026-03-22-bookmark-api.md
```

The lead will propose a team, wait for your approval, then run the pipeline.

### Artifacts

Specs, plans, research, and status notes go in `docs/team-up/` by default (with subdirectories `specs/`, `plans/`, `research/`, `status/`). Override by setting `TEAM_UP_ARTIFACTS` in your environment or just telling the lead where you want them.

## Customizing agents

Agent definitions live in `agents/` as markdown files. To add a custom agent, create a new `.md` file following the same format:

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

## Design principles

- **User stays in control** — The lead proposes, you approve. No merges without sign-off.
- **No tech debt** — Work isn't done until cleanup is done.
- **Feature branches only** — Never commits to main.
- **Escalate, don't loop** — When stuck, the lead asks you rather than retrying forever.
- **Right-size the team** — Only spin up roles the task actually needs.

## Acknowledgments

The workflow and agent coordination patterns in this plugin are inspired by [superpowers](https://github.com/obra/superpowers) by Jesse Vincent, which introduced structured skill-based workflows for Claude Code — brainstorming, TDD, plan writing, code review, and more. team-up builds on those ideas by turning them into a multi-agent team with persistent specialist roles.

## License

MIT
