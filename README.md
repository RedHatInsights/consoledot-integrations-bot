# consoledot-integrations-bot

Custom bot runner built on [dev-bot](https://github.com/RedHatInsights/platform-frontend-ai-dev).

## Build

\`\`\`bash
git submodule update --init --recursive
docker build -f dev-bot/Dockerfile.runner -t consoledot-integrations-bot:local .
\`\`\`

## Updating dev-bot

\`\`\`bash
cd dev-bot && git pull origin master && cd ..
git add dev-bot
git commit -m "chore: update dev-bot submodule"
\`\`\`