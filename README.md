# Pickthree — GitHub Action

**Good, fast, cheap — pick three.** A quiet, self-hosted, bring-your-own-key AI code reviewer:
a walkthrough plus at most a handful of high-confidence inline comments — or silence.

```yaml
# .github/workflows/pickthree.yml
name: Pickthree
on: [pull_request]
permissions: { pull-requests: write, contents: read, issues: write }
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: pickthreehq/pickthree-action@v1
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}   # or OPENAI_API_KEY, GEMINI_API_KEY, …
```

- **Self-hosted by default** — the reviewer runs in *your* CI; code and conventions never leave your deployment.
- **BYO key on every tier** — any Anthropic / OpenAI / Google / OpenRouter model.
- **Quiet by default** — noise is the enemy; ≤10 inline comments or nothing.

> ⚙️ **Status:** the runnable `v1` wrapper ships with the first tagged release. Star the org to follow along.
