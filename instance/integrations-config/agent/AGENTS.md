# Consoledot Integrations Instance — Additional Instructions

## Model Tiers

Roles use three `rehor-openai` models. All are declared in
`opencode-deployment.json` and allowlisted by the model proxy:

- `integrations-coordinator`: `gpt-6.1-sol` for coordination and review.
- `backend`, `frontend`, `app-interface`, `tooling`: `gpt-6.1-terra` for
  substantial implementation.
- `cve`: `gpt-6-luna` for bounded remediation.
- `rds-upgrade`: `gpt-6.1-sol` for complex multi-stage reasoning.

OpenCode 2.1.2 resolves the primary session from the root model selected by
`instance.yaml`, so `gpt-6.1-sol` must stay in both `instance.yaml` and the
coordinator frontmatter. A subagent uses its frontmatter model when the
subagent tool creates the child session; an explicit `task` model overrides it.
Do not pass a model on `task` calls without a ticket-specific reason.
`gpt-6.1-terra` is not in the pinned OpenCode 2.1.2 baked catalog (the runner
disables catalog fetching), so it depends on the deployment declaration, proxy
allowlist, and fallback pricing; verify it on a real staging cycle before
enabling dispatch.

## Native Subagents

`opencode.json` selects `integrations-coordinator` as the default primary agent
and limits delegation depth to one. The coordinator may invoke only the native
subagents in `.opencode/agents/`: `backend`, `frontend`, `app-interface`,
`tooling`, `cve`, and `rds-upgrade`. These are actual native subagents, not
instruction snippets. When shared workflow guidance refers to
`personas/<name>/prompt.md`, map the role to the matching native subagent
instead of reading legacy persona paths; this profile has no persona directory:

- `frontend` → `frontend`; `backend`/`operator`/`rbac` → `backend`;
  `config` → `app-interface`; `tooling` → `tooling`; `cve` → `cve` (layered on
  the repository-matching base subagent); `rds-upgrade` → `rds-upgrade`
  (layered on `app-interface`).

- Invoke `backend` for scoped Go service and API implementation (sources,
  notifications backend/gateway, payload tracker, scheduler, ingress, storage
  broker, export service, PDF generator).
- Invoke `frontend` for scoped Node frontend work (notifications-frontend,
  user-preferences-frontend).
- Invoke `app-interface` for GitLab-hosted work: app-interface service
  definitions and IQE plugins.
- Invoke `tooling` for Dockerfile, script, and CI tooling changes; `cve` for
  CVE remediation; `rds-upgrade` for staged RDS blue-green upgrades.
- Coordinator owns Jira/task lifecycle, status, integration, review, commit/push,
  PR/MR lifecycle, and cycle completion. Workers implement assigned scope,
  return evidence, and cannot delegate.
- Subagents share the runner filesystem and inherited tools. Their role
  permissions do not provide separate OS sandboxes. Keep every brief inside
  approved repository/workspace boundaries and inspect changes before acting.

Use `/home/botuser/repos` for project checkouts, `/home/botuser/data` for
persistent task data, and `/tmp/opencode` for task scratch files. These are the
deployment-approved external roots; the runner workspace is `/home/botuser/app`.

## Native Runtime Verification Override

OpenCode has no LSP runtime or tools. This overrides workflow and persona
directives to use LSP, including LSP diagnostics before a commit: use the
current repository's existing lint, typecheck, compiler, and test targets for
diagnostics and verification, and report the commands actually run and their
results. Do not claim LSP diagnostics were performed.

## Supported Repositories

Select the repository from the ticket's canonical `repo:` label and
`project-repos.json`. Clone the bot fork as `origin` and the recorded upstream
as `upstream`; base task branches on the current upstream default branch and
open PRs/MRs against upstream only when the coordinator explicitly assigns
publication. GitLab-hosted entries (`app-interface`, `iqe-*` plugins) use the
`host: gitlab` identity and review flow.

Follow each repository's own AGENTS.md/README guidance, version declarations,
and check commands; do not assume commands from one repository apply to
another. Stop and report when required guidance or task authorization is
unavailable; optional verification setup being unavailable does not block
implementation.

## Version Management

This instance has **nvm** (Node) and **goenv** (Go) version managers installed.
Use them to match the version required by each repo.

### Node.js (nvm)

Before working on a repo with `package.json`, prefer its `.nvmrc` or
`engines.node` field and attempt version-manager installation/use once:

```bash
nvm use          # reads .nvmrc if present
nvm install 20   # install + switch to Node 20
nvm use default  # back to default (22)
```

If the required version cannot be installed or used, choose the closest
installed Node version and continue implementation. Do not change repository
runtime declarations or package/npm configuration. Report the actual version
and compatibility uncertainty; run available checks and accurately report
skips. Default: Node.js 22 LTS. Available globally via `/usr/local/bin/node`.

### Go (goenv)

Before working on a repo with `go.mod`, check the `go` directive and switch if
needed:

```bash
goenv versions                    # list installed versions
goenv install 1.23.0              # install a new version
goenv global 1.23.0               # set as default
goenv local 1.24.2                # set for current directory only
```

Pre-installed: Go 1.24.2 (default), 1.25.7. Available globally via
`/usr/local/bin/go`.

### When to switch

- `go.mod` says `go 1.23` → `goenv install 1.23.x && goenv local 1.23.x`
- `.nvmrc` says `20` → `nvm use 20` (installs automatically if missing)
- No version file → use the defaults
