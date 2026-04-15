---
name: sisakulint
description: >
  Harden GitHub Actions workflows with sisakulint.
  Trigger: sisakulint, workflow hardening, CI/CD security, actions pinning, workflow lint
---

# sisakulint

Static security linter for GitHub Actions workflows with auto-fix support.

## Steps

1. Run `sisakulint`. If command not found, install with `go install github.com/sisaku-security/sisakulint/cmd/sisakulint@latest` and retry
2. Review findings, then apply auto-fix with `sisakulint -fix on`
3. If commit-sha resolution fails due to GitHub API rate limiting, use SHAs from `sisakulint -fix dry-run` output to fix remaining refs manually
4. Add `persist-credentials: false` to `actions/checkout` steps for the artipacked rule (may not be auto-fixed)
5. Re-run `sisakulint` and confirm all rules pass

For remote repositories, scan with `sisakulint -remote owner/repo`, then clone locally to apply fixes.

## Rules requiring manual remediation

| Rule | Detection | Remediation |
|------|-----------|-------------|
| commit-sha | Tag refs not pinned to full SHA | Copy SHAs from dry-run output (on API rate limit) |
| artipacked | Checkout credential persistence | Add `persist-credentials: false` |
| dependabot-github-actions | Missing dependabot config | Create `.github/dependabot.yaml` |
| code-injection-critical | Code injection in privileged triggers | Replace expression interpolation with intermediate env vars |
| untrusted-checkout | Checkout of untrusted PR code | Review `pull_request_target` ref usage |

## Full rules reference

See [references/rules.md](references/rules.md) for the complete list of all rules.
