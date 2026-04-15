---
name: sisakulint
description: >
  Harden GitHub Actions workflows with sisakulint.
  Trigger: sisakulint, workflow hardening, CI/CD security, actions pinning, workflow lint
---

# sisakulint

Static security linter for GitHub Actions workflows with auto-fix support.

## Steps

1. Run `sisakulint`. If not found, install with `go install github.com/sisaku-security/sisakulint/cmd/sisakulint@latest`
2. Review findings, then apply auto-fix: `sisakulint -fix on`
3. Apply manual remediations for rules that auto-fix cannot handle (see below)
4. Validate: re-run `sisakulint` and confirm zero findings
5. If findings remain, fix and re-run until all rules pass

For remote repos: `sisakulint -remote owner/repo` to scan, then clone locally for fixes.

## Gotchas

- **GitHub API rate limit**: `sisakulint -fix on` resolves commit SHAs via the GitHub API. On rate limit, it silently skips pins. Always run `sisakulint -fix dry-run` first to capture SHAs, then apply manually if `-fix on` misses any.
- **artipacked is never auto-fixed**: The `-fix` flag does not add `persist-credentials: false`. Always check `actions/checkout` steps manually after auto-fix.
- **`go install` requires Go 1.23+**: Older Go versions fail silently or produce incompatible binaries.
- **Monorepo path filters**: When workflows use `paths:` filters, `sisakulint -remote` may miss files outside the default branch tree. Clone and run locally for full coverage.

## Manual remediation patterns

### artipacked — Credential persistence

```yaml
- uses: actions/checkout@<sha>
  with:
    persist-credentials: false
```

### code-injection-critical — Expression interpolation

Replace direct expression usage in `run:` with an intermediate env var:

```yaml
# Before (vulnerable — expression injected directly into shell)
- run: echo "Hello ${{ github.event.issue.title }}"

# After (safe — expression bound to env var, quoted in shell)
- run: echo "Hello $TITLE"
  env:
    TITLE: ${{ github.event.issue.title }}
```

### dependabot-github-actions — Missing dependabot config

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

### untrusted-checkout — PR code checkout

For `pull_request_target` workflows, never checkout `github.event.pull_request.head.sha` in a step that runs untrusted code. Use `pull_request` trigger instead, or isolate the checkout into a separate job with minimal permissions.

## Full rules reference

See [references/rules.md](references/rules.md) for the complete list of 52 rules across 7 categories.
