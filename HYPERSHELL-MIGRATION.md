# Consoledot Integrations HyperShell migration draft

This migrates the integrations agent to the OpenCode runtime with OpenAI models
and prepares its dispatcher profile. It does not enable the dispatcher or
replace the existing deployment. Do not enable this repository in the dispatcher
until the gates below are complete.

## What changes here

- Profile `instance/integrations-config` selects the OpenCode coordinator
  runtime (`runtime: opencode`, `BOT_EXECUTION_ENGINE: coordinator`) and the
  `rehor-openai` provider with `gpt-6.1-terra`.
- Native OpenCode configuration replaces the Claude layout: `AGENTS.md`
  replaces `CLAUDE.md`, subagents live in `.opencode/agents/`, and the provider
  is declared in `opencode-deployment.json` (no secrets; URL and token come
  from deployment environment).
- Preflights are declared in `instance.yaml` and executed by the dispatcher's
  TypeScript engine (`@rehor/preflights`) instead of the workflow default
  Python set: `gh-pr-status`, `gl-mr-status`, `jira-sprint`.
- Top-level `env` carries the non-secret profile settings for dispatcher wiring
  and the sandbox runner.
- The legacy `deploy/template.yaml` Deployment is left untouched; this
  repository moves to dispatcher-managed sandboxes and does not maintain the
  classic template further.

## Before rollout

1. Merge [Rehor PR #667](https://github.com/OpenShift-Fleet/rehor/pull/667)
   (TypeScript preflights) and [PR #668](https://github.com/OpenShift-Fleet/rehor/pull/668)
   (top-level `env`), then deploy a dispatcher image containing both. Older
   dispatchers do not read top-level `env` and cannot run the declared
   preflights.
2. Wire the OpenAI model route in app-interface: `REHOR_MODEL_PROXY_URL`
   (proxy port 8450) and the gateway token for `REHOR_MODEL_PROXY_TOKEN`.
   This PR contains no secret; the token placeholder must be replaced by the
   deployment. Add this repository to the dispatcher's
   `AGENT_DISPATCHER_CONFIG_REPOS` in the same GitOps change, not here.
3. Verify the `dev-bot` submodule pin builds a compatible runner image and
   promote it. Verify the configured `latest` tag resolves to that image before
   enabling dispatch; the tag can change. The dispatcher checks out the config
   repository and its pinned submodule separately from the baked runner image;
   their runtime, profile, and workflow contents must be compatible.
4. Confirm the instance ID, label, and Jira board settings against the current
   deployment. `BOT_LABEL: hcc-ai-integrations` and
   `include_backlog: false` are copied from the template defaults; reconcile
   them with the app-interface values for `BOT_INCLUDE_BACKLOG` and board
   settings before cutover. Changing the instance ID can hide existing tasks.
   The `jira-sprint` preflight's optional `email` pass is not configured
   because the bot's Jira identity comes from deployment secrets; validate
   assigned-ticket behavior before cutover.
5. Plan cutover so the legacy runner and dispatcher never process the same
   instance simultaneously. The legacy KEDA window (weekdays 8–17 in three
   timezones) is not reproduced by the dispatcher; resolve the intended
   operating window before enabling dispatch.
6. Run no-work and representative work cycles. Verify Jira feedback wake on an
   active task, one GitHub PR and one GitLab MR cycle, task identity, and
   cleanup.

## Rollback

The old pinned Python `InstanceConfig` ignores the new profile keys, so the
currently deployed image keeps running unchanged with this config merged.
Full rollback is reverting this config revision, or setting
`BOT_EXECUTION_ENGINE: python` via GitOps while the dispatcher drains. Remove
this repository from dispatcher configuration and verify the dispatcher has
stopped launching this profile before re-enabling the KEDA schedule on the
legacy Deployment.
