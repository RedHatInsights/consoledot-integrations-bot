# consoledot-integrations-bot

Custom bot runner instance for the ConsoleDot Integrations team, built on [dev-bot](https://github.com/RedHatInsights/platform-frontend-ai-dev).

Focused on ConsoleDot integration services — including [sources-api-go](https://github.com/RedHatInsights/sources-api-go), [notifications-backend](https://github.com/RedHatInsights/notifications-backend), [payload-tracker-go](https://github.com/RedHatInsights/payload-tracker-go), [export-service-go](https://github.com/RedHatInsights/export-service-go), and related microservices.

## Architecture

Uses dev-bot as a git submodule. The submodule ships `Dockerfile.runner` which builds the full bot image and runs instance-specific customization hooks from this repo.

The instance runs the OpenCode coordinator runtime with the `rehor-openai` provider through the shared model proxy. The coordinator and multi-stage `rds-upgrade` work use `gpt-6.1-sol`; substantial `backend`, `frontend`, `app-interface`, and `tooling` implementation uses `gpt-6.1-terra`; bounded `cve` remediation uses `gpt-6-luna`. The profile is dispatcher-managed: the agent dispatcher checks out this repository, runs the declared TypeScript preflights, and launches a per-run sandbox from the configured image.

```
consoledot-integrations-bot/
├── dev-bot/        # Git submodule (don't modify)
├── deploy/
│   └── template.yaml   # OpenShift deployment template (rollback path)
├── setup.sh        # Custom build steps (dnf install, pip install, etc.)
├── instance/       # Extra files COPYed into the image
│   └── integrations-config/
│       ├── opencode-deployment.json  # Provider/model config (no secrets)
│       └── agent/
│           ├── instance.yaml     # Dispatcher profile: image, env, preflights
│           ├── AGENTS.md          # Native instance instructions
│           ├── opencode.json      # Default agent + delegation depth
│           ├── .opencode/agents/  # Native subagents (coordinator, backend, frontend, app-interface)
│           ├── project-repos.json # Repos the bot works on
│           └── mcp.json           # MCP server config
└── README.md
```

Preflights are declared in `instance.yaml` (`gh-pr-status`, `gl-mr-status`, `jira-sprint`) and executed by the dispatcher's TypeScript engine; this repository ships no preflight scripts.

See [HYPERSHELL-MIGRATION.md](HYPERSHELL-MIGRATION.md) for rollout gates, prerequisites, and rollback.

No Dockerfile in this repo — Konflux points at `dev-bot/Dockerfile.runner`.

## Build

```bash
git submodule update --init --recursive
docker build -f dev-bot/Dockerfile.runner -t consoledot-integrations-bot:local .
```

## Customization

- **setup.sh** — runs as root during build. Install packages, write config, etc.
- **instance/** — files COPYed to `/home/botuser/app/instance/` in the image.

## Updating dev-bot

```bash
cd dev-bot && git pull origin master && cd ..
git add dev-bot
git commit -m "chore: update dev-bot submodule"
```

Konflux also opens automated PRs when dev-bot merges new features.

## Konflux

```yaml
dockerfile: dev-bot/Dockerfile.runner
path-context: .
```
