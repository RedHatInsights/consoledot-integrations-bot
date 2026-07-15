# consoledot-integrations-bot

Custom bot runner instance for the ConsoleDot Integrations team, built on [dev-bot](https://github.com/RedHatInsights/platform-frontend-ai-dev).

Focused on ConsoleDot integration services — including [sources-api-go](https://github.com/RedHatInsights/sources-api-go), [notifications-backend](https://github.com/RedHatInsights/notifications-backend), [payload-tracker-go](https://github.com/RedHatInsights/payload-tracker-go), [export-service-go](https://github.com/RedHatInsights/export-service-go), and related microservices.

## Architecture

Uses dev-bot as a git submodule. The submodule ships `Dockerfile.runner` which builds the full bot image and runs instance-specific customization hooks from this repo.

```
consoledot-integrations-bot/
├── dev-bot/        # Git submodule (don't modify)
├── deploy/
│   └── template.yaml   # OpenShift deployment template
├── setup.sh        # Custom build steps (dnf install, pip install, etc.)
├── instance/       # Extra files COPYed into the image
│   └── integrations-config/
│       └── agent/
│           ├── project-repos.json   # Repos the bot works on
│           └── mcp.json             # MCP server config
└── README.md
```

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
