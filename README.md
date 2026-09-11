# Pickthree — GitHub Action

**Good, fast, cheap — pick three.** Quiet AI code review that runs in your own CI with your own API keys: a walkthrough and at most 10 inline comments per pull request, or silence.

Pickthree reviews your pull requests inside *your own* GitHub Actions runner, with *your own* LLM
API keys, and never phones home: no telemetry, no licence server, nothing about your code reaches
us. Free for public repositories and for private repositories with up to 3 developers a month;
paid tiers are flat per organisation from \$49/month, with every feature on every tier. https://pickthree.dev

## Install

```yaml
# .github/workflows/pickthree.yml
name: Pickthree
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
  issue_comment:                       # @pickthree commands in the pull request conversation
    types: [created]
  pull_request_review_comment:         # replies on Pickthree's finding threads
    types: [created]

concurrency:
  group: pickthree-${{ github.event.pull_request.number || github.event.issue.number }}
  cancel-in-progress: false

permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: pickthreehq/pickthree-action@v1
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}   # or OPENAI_API_KEY / GEMINI_API_KEY / ...
```

That is the whole install: no checkout, no database, no server to run. The review state — which
commit was reviewed, the earlier findings, the learnings, the pause flag — lives in a hidden block
inside Pickthree's own walkthrough comment on the pull request.

## What a run does

| Event | Behaviour |
|---|---|
| `pull_request` opened / synchronize / reopened / ready_for_review | Review the head. A second push is **incremental**: only files whose diff changed since the last posted review go to the models, earlier findings are carried, and threads for findings you fixed are marked resolved. A rebase with no content change costs nothing. Drafts and other actions are ignored. |
| `issue_comment` on a pull request | `@pickthree review`, `pause`, `resume`, `resolve`, `configuration`, `explain` — from anyone with write access. |
| `pull_request_review_comment` | A reply on one of Pickthree's finding threads gets a grounded answer about that finding. |
| `workflow_dispatch` with `pr: <number>` | Forced full review of that pull request. |

Quiet by default: judge-confirmed critical and major findings go inline, at most 10 of them, and
everything else collapses into the walkthrough. A clean pull request gets a short walkthrough and
silence — no poem, no filler praise.

The Action never checks out or executes pull request code. It reads the diff through the API and
only *parses* the changed files, which is what makes `pull_request_target` safe for forks.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `github-token` | `${{ github.token }}` | Token for the API calls. A PAT or App token also resolves fixed findings' threads. |
| `config` | — | Path in the checkout to a configuration file applied as the top layer |
| `dry-run` | `false` | Print the review instead of posting it |
| `force` | `false` | Re-review an already-reviewed head, and the whole pull request |
| `pr` | — | Pull request number, for `workflow_dispatch` |
| `license` | — | License token for the paid tiers (pass it from a secret) |
| `image` | `ghcr.io/pickthreehq/pickthree` | Image repository — change it only for a registry mirror we have granted you |
| `version` | `0.1.0` | Image tag |
| `digest` | — | Pin by digest; wins over `version` |
| `registry-token` | `${{ github.token }}` | Credential for `docker login` — see *Access to the image* below |
| `registry-username` | `${{ github.actor }}` | Username for `docker login` |

Provider credentials and every other deployment variable come from the **job environment** (the
`env:` block above) and are forwarded into the container by name, so no value is ever printed.

## Outputs

`outcome`, `inline-count`, `minor-count`, `checks-failed`, `cost-usd`, `review-url`, `head-sha`.

## Configuration

A `.pickthree.yaml` at the repository root, validated against a published JSON schema, with an
organisation-wide layer, path-scoped rules, per-change-class model routing and a hard cap on inline
comments: https://pickthree.dev/docs/config

## Minimum permissions

`contents: read`, `pull-requests: write`, `issues: write`. That is all the workflow token needs.

## Access to the image

Pickthree is proprietary software and the container image this action runs is **private**: it is
never publicly pullable. Access is granted per customer as a **read-only registry token**, issued
together with your licence — pass it as `registry-token` from a repository secret:

```yaml
      - uses: pickthreehq/pickthree-action@v1
        with:
          registry-token: ${{ secrets.PICKTHREE_REGISTRY_TOKEN }}
          license: ${{ secrets.PICKTHREE_LICENSE }}
```

Without access the run stops before any code is read, with a message that says so. To get a token,
or to have your organisation's repositories granted access: **https://pickthree.dev/contact**

## What is in this repository

Only the wrapper: `action.yml`, this README, the licence and one selftest workflow. Pickthree
itself is **proprietary, closed-source** software, shipped as a signed container image with an
SBOM and build provenance. Not a line of the reviewer lives here.

Questions, or anything you need beyond the Action: **https://pickthree.dev/contact** ·
issues on the wrapper: https://github.com/pickthreehq/pickthree-action/issues
