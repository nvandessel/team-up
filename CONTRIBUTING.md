# Contributing to team-up

Thanks for your interest in improving team-up. Here's how to get involved.

## Ways to contribute

- **Report bugs** — something broke or didn't work as expected? [Open an issue](https://github.com/nvandessel/team-up/issues/new?template=bug_report.md).
- **Suggest features** — new agent roles, workflow improvements, better coordination patterns? [Open a feature request](https://github.com/nvandessel/team-up/issues/new?template=feature_request.md).
- **Add or improve agents** — create a new specialist agent or refine an existing one.
- **Improve docs** — fix unclear language, add examples, or improve the README.

## Adding a new agent

1. Create a new `.md` file in `agents/`
2. Follow the existing format — see any agent in that directory for the pattern
3. Register it in `.claude-plugin/marketplace.json` under the `agents` array
4. Test it by running `/team-up` and verifying the lead discovers and uses your agent
5. Open a PR

## Agent format

Every agent definition needs:

- **YAML frontmatter** with `model: opus` (or `sonnet` for lighter tasks)
- **Role** — what the agent does and doesn't do
- **Boundaries** — explicit in-scope and out-of-scope
- **Process** — step-by-step workflow
- **Quality standards** — checklist before reporting done
- **Communication** — status codes: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED`

## Modifying the skill

The team lead playbook lives in `skills/team-up/SKILL.md`. If you're changing how the lead coordinates, update this file. Keep the 7-phase structure unless you have a good reason to change it.

## Pull requests

- Keep PRs focused — one change per PR
- Test with a real `/team-up` invocation before submitting
- Update the README if user-facing behavior changes

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
