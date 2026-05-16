# dependabot-github-actions

The repository has no Dependabot configuration for the `github-actions` ecosystem. Without it, pinned action SHAs never get updated, and the workflows drift away from upstream security fixes.

## Remediation

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

For monorepos with workflows in subdirectories, add a separate `updates:` entry per directory — Dependabot does not recurse.

## Pairing with auto-merge

To avoid weekly bump PRs piling up, add an auto-merge workflow restricted to Dependabot:

```yaml
# .github/workflows/dependabot-auto-merge.yml
on: pull_request
permissions:
  contents: write
  pull-requests: write
jobs:
  auto-merge:
    if: github.event.pull_request.user.login == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Use `github.event.pull_request.user.login`, not `github.actor` — the `bot-conditions` rule flags `github.actor == 'dependabot[bot]'` because `github.actor` can be spoofed in certain re-run scenarios.
