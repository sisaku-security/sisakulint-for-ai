---
name: sisakulint
description: >
  Scan and harden GitHub Actions workflows — pin actions to full commit SHAs,
  detect expression injection vulnerabilities, enforce credential hygiene,
  and set up Dependabot for actions updates. Use this skill when the user has
  GitHub Actions workflow files and wants to improve their CI/CD security posture,
  even if they don't explicitly mention "linting" or "hardening." Also use when
  reviewing .github/workflows/*.yml for supply-chain risks or preparing workflows
  for security compliance.
---

# sisakulint

Static security linter for GitHub Actions workflows with auto-fix support.

## Steps

1. Run `sisakulint`. If not found, install with `go install github.com/sisaku-security/sisakulint/cmd/sisakulint@v0.2.9`
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
- **`-remote` の信頼境界**: `sisakulint -remote` は外部リポジトリのワークフローを静的解析し、ルールID・行番号・メッセージのみ出力する。YAMLの取り込みや実行は行わない。ただしプライベートリポジトリのスキャンには `GITHUB_TOKEN` が必要なため、トークンのスコープは最小権限にすること。
- **Step-level timeout-minutes is intentional**: `sisakulint` flags `missing-timeout-minutes` on individual steps even when the job has `timeout-minutes` set. This is **not a false positive** — a single step can consume the entire job timeout, blocking subsequent steps. Step-level timeouts are a defense-in-depth measure. Apply them.
- **AI rules partial coverage**: `sisakulint` reliably detects `ai-action-prompt-injection` but may not fire `ai-action-unrestricted-trigger` or `ai-action-excessive-tools` in all cases. After auto-fix, manually review AI agent workflows for these two patterns (see manual remediation below).

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

### secret-exposure — Excessive secrets in environment

Remove secrets that are not required by the step. Secrets like `DATABASE_URL` or payment keys should never appear in CI deploy steps — they belong in a runtime secrets manager.

```yaml
# Before (6 secrets exposed to deploy script)
- run: ./scripts/deploy.sh
  env:
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    STRIPE_SECRET_KEY: ${{ secrets.STRIPE_SECRET_KEY }}

# After (only deploy-relevant secrets)
- run: ./scripts/deploy.sh
  env:
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### untrusted-checkout — PR code checkout

For `pull_request_target` workflows, never checkout `github.event.pull_request.head.sha` in a step that runs untrusted code. Use `pull_request` trigger instead, or isolate the checkout into a separate job with minimal permissions.

### ai-action-unrestricted-trigger — Missing actor allowlist

AI agent workflows triggered by `issue_comment` or similar events must gate on `github.event.comment.author_association` to prevent any user from triggering expensive or privileged AI execution.

```yaml
# Add actor allowlist before AI action runs
if: >-
  contains(github.event.comment.body, '/ai-review') &&
  contains(fromJSON('["MEMBER","OWNER","COLLABORATOR"]'),
           github.event.comment.author_association)
```

### ai-action-excessive-tools — Dangerous tool grants

AI agents triggered by untrusted input should not have write access to the codebase. Restrict `allowed_tools` to read-only or review-only operations.

```yaml
# Before (dangerous — full write access)
allowed_tools: |
  Bash
  Edit
  Write
  mcp__github__push_files
  mcp__github__delete_file

# After (safe — review only, creates PR for human approval)
allowed_tools: |
  Read
  Glob
  Grep
  mcp__github__create_pull_request
```

## Full rules reference

See [references/rules.md](references/rules.md) for the complete list of 52 rules across 7 categories.
