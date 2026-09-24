# OpenCode workflows

This repository uses [opencode-review-threads](https://github.com/tonythethompson/opencode-review-threads) through two reusable workflows pinned to `v1.4.1`.

## Workflows

| File | Trigger | Purpose |
|---|---|---|
| `.github/workflows/opencode-review.yml` | `pull_request` (opened, reopened, synchronize, ready_for_review) | Posts one structured GitHub review per run: a short summary body plus inline, individually resolvable review threads, authored by `opencode-agent[bot]` |
| `.github/workflows/opencode.yml` | PR conversation or review comments containing `/oc` / `/opencode`; `workflow_dispatch` with `prompt` | On-demand OpenCode agent. `/oc fix ...` prompts can edit code and push commits to the PR branch; `/oc autopilot` runs a merge-readiness pass (conflicts, unresolved threads, failing CI) |

## Gates

- **Review:** same-repository PRs only (fork PRs receive no secrets), non-draft, author association `OWNER`/`MEMBER`/`COLLABORATOR`/`CONTRIBUTOR`, and neither the PR author nor the triggering actor may be a bot. A `model` input is an explicit opt-in bypass of the association check.
- **Bot:** commenters must have `OWNER`/`MEMBER`/`COLLABORATOR`/`CONTRIBUTOR` association, and the triggering actor must not be a bot (bot-triggered runs cannot mint the OIDC/secrets the reusable call needs and fail at startup).

## Incremental reviews

After a submitted review, subsequent pushes diff only commits since that review, so unchanged findings are not re-raised. Comment `/oc pr-review` for a fresh full pass.

## Secrets

| Secret | Purpose |
|---|---|
| `OPENCODE_API_KEY` | Zen provider (`zen:` chain entries) |
| `CLOUDFLARE_ACCOUNT_ID` + `CLOUDFLARE_API_TOKEN` | Cloudflare Workers AI (`cf:` chain entries) |
| `CONTEXT7_API_KEY` | Optional. Enables the bundled context7 MCP server so fix/autopilot runs can consult current third-party library docs before editing |

Each job probes `models-review` / `models-fix` in order; the first reachable model wins. Missing providers are skipped cleanly, but at least one must work.

## Requirement

The OpenCode GitHub App must be installed on this repository for the `opencode-agent[bot]` token exchange. If it is not installed, set `use-github-token: true` on both caller jobs; reviews and replies then post as `github-actions[bot]` instead; in that mode the bot caller also needs `contents: write` for `/oc fix` pushes.
