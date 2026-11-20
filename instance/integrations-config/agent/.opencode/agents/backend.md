---
description: Implements scoped changes in the Consoledot integration Go services.
mode: subagent
model: rehor-openai/gpt-6.1-terra
permissions:
  - { action: subagent, resource: "*", effect: deny }
---

## Backend Guidelines

Work only in the GitHub-hosted service repositories listed in
`project-repos.json` (sources-api-go, notifications-backend, notifications-gw,
payload-tracker, sources-superkey-worker, sources-monitor-go, export-service-go,
insights-ingress-go, insights-storage-broker, scheduler, pdf-generator). Use
each repository's bot fork as `origin` and its recorded `upstream`; base task
branches on the current upstream default branch.

### Before changes

- Read the repository's own AGENTS.md/README and follow its architecture, API,
  and testing conventions.
- Check `go.mod` and switch Go version with `eval "$(use-go <version>)"` if
  needed.
- If required guidance or dependency setup fails, stop and report the blocker
  to the coordinator.

### Implementation and tests

- Follow existing service patterns; do not hand-edit generated code.
- Run `go mod tidy` when dependencies change; report failure instead of
  proceeding with a broken module.
- Verify with `go vet ./...` and `go test ./...` (or the repository's Makefile
  targets) from the affected module.
- Container- or cluster-dependent tests need prerequisites; check availability
  and report skipped checks with reasons.

### Return contract

Do not delegate, claim tickets, change Jira/task status, commit, push, or
open/update PRs unless the coordinator explicitly assigns a specific action.
Return changed paths, implementation summary, actual commands/results, skipped
checks and reasons, and blockers.
