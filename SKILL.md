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
3. Apply manual remediations for rules that auto-fix cannot handle — open the per-rule reference linked from the table below.
4. Validate: re-run `sisakulint` and confirm zero findings
5. If findings remain, fix and re-run until all rules pass

For remote repos: `sisakulint -remote owner/repo` to scan, then clone locally for fixes.

## Gotchas

- **GitHub API rate limit**: `sisakulint -fix on` resolves commit SHAs via the GitHub API. On rate limit, it silently skips pins. Always run `sisakulint -fix dry-run` first to capture SHAs, then apply manually if `-fix on` misses any.
- **artipacked is never auto-fixed**: The `-fix` flag does not add `persist-credentials: false`. Always check `actions/checkout` steps manually after auto-fix.
- **`go install` requires Go 1.23+**: Older Go versions fail silently or produce incompatible binaries.
- **Monorepo path filters**: When workflows use `paths:` filters, `sisakulint -remote` may miss files outside the default branch tree. Clone and run locally for full coverage.
- **`-remote` trust boundary**: `sisakulint -remote` only performs static analysis and outputs rule IDs, line numbers, and messages — it never imports or executes external YAML. Scanning private repos requires `GITHUB_TOKEN`; keep the token scope minimal.
- **Step-level timeout-minutes is intentional**: `sisakulint` flags `missing-timeout-minutes` on individual steps even when the job has `timeout-minutes` set. This is **not a false positive** — a single step can consume the entire job timeout, blocking subsequent steps. Step-level timeouts are a defense-in-depth measure. Apply them.
- **AI rules partial coverage**: `sisakulint` reliably detects `ai-action-prompt-injection` but may not fire `ai-action-unrestricted-trigger` or `ai-action-excessive-tools` in all cases. After auto-fix, manually review AI agent workflows for these two patterns.

## Rules

The `Reference` column links to per-rule remediation guides under `references/`.

### Code Injection & Expression Safety (9)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| code-injection-critical | Critical | Code injection in privileged triggers | [code-injection-critical.md](references/code-injection-critical.md) |
| code-injection-medium | Medium | Code injection in normal triggers | [code-injection-critical.md](references/code-injection-critical.md) |
| envvar-injection-critical | Critical | Unsafe env var usage in high-risk triggers | |
| envvar-injection-medium | Medium | Unsafe env var usage in standard triggers | |
| envpath-injection-critical | Critical | PATH manipulation in privileged contexts | |
| envpath-injection-medium | Medium | PATH manipulation in regular contexts | |
| argument-injection | High | Command-line argument injection | |
| output-clobbering | High | Output clobbering via $GITHUB_OUTPUT | |
| unsound-contains | Medium | Unsafe contains() usage in conditions | |

### Supply Chain & Dependency Security (7)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| commit-sha | High | Action refs not pinned to full commit SHA | |
| known-vulnerable-actions | Critical | Actions with known vulnerabilities | |
| archived-uses | Medium | Archived/deprecated actions | |
| impostor-commit | Critical | Impostor commit attack patterns | |
| ref-confusion | High | Branch reference manipulation | |
| unpinned-images | High | Container images without version pins | |
| action-list | Medium | Action allowlist/blocklist enforcement | |

### Credential & Secret Protection (7)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| credentials | Critical | Hardcoded credential patterns (Rego) | |
| secret-exposure | High | Excessive secrets exposure | [secret-exposure.md](references/secret-exposure.md) |
| unmasked-secret-exposure | High | Unmasked secrets in workflow logs | [secret-in-log.md](references/secret-in-log.md) |
| secret-exfiltration | High | Network-based secret extraction | [secret-exfiltration.md](references/secret-exfiltration.md) |
| secrets-in-artifacts | High | Sensitive data in uploaded artifacts | |
| secrets-inherit | Medium | Excessive secrets inheritance | |
| artipacked | Medium | Credential persistence vulnerability | [artipacked.md](references/artipacked.md) |

### Pipeline Poisoning & Artifact Integrity (8)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| untrusted-checkout | Critical | Checkout of untrusted PR code | [untrusted-checkout.md](references/untrusted-checkout.md) |
| untrusted-checkout-toctou-critical | Critical | TOCTOU race conditions in checkout (Critical) | [untrusted-checkout.md](references/untrusted-checkout.md) |
| untrusted-checkout-toctou-high | High | TOCTOU issues in checkout (High) | [untrusted-checkout.md](references/untrusted-checkout.md) |
| artifact-poisoning-critical | Critical | Malicious artifact injection in critical flows | [artifact-poisoning-critical.md](references/artifact-poisoning-critical.md) |
| artifact-poisoning-medium | Medium | Artifact tampering in standard flows | [artifact-poisoning-critical.md](references/artifact-poisoning-critical.md) |
| cache-poisoning | High | Cache corruption vulnerabilities | [cache-poisoning.md](references/cache-poisoning.md) |
| cache-poisoning-poisonable-step | High | Unsafe steps after vulnerable checkout | [cache-poisoning.md](references/cache-poisoning.md) |
| reusable-workflow-taint | High | Untrusted inputs in reusable workflows | |

### Triggers & Access Control (7)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| dangerous-triggers-critical | Critical | Privileged triggers without mitigations | |
| dangerous-triggers-medium | Medium | Privileged triggers with partial mitigations | |
| permissions | High | Permission scope and value validation | |
| bot-conditions | Medium | Bot actor conditions validation | [dependabot-github-actions.md](references/dependabot-github-actions.md) |
| improper-access-control | High | Label-based approval bypass | |
| self-hosted-runners | Medium | Self-hosted runner security | |
| request-forgery | High | SSRF vulnerabilities | |

### AI Agent Security (3)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| ai-action-unrestricted-trigger | Critical | AI actions allowing any user to trigger execution | [ai-action-unrestricted-trigger.md](references/ai-action-unrestricted-trigger.md) |
| ai-action-excessive-tools | High | Dangerous tool grants to AI agents in untrusted triggers | [ai-action-excessive-tools.md](references/ai-action-excessive-tools.md) |
| ai-action-prompt-injection | High | Untrusted input interpolated into AI agent prompts | |

### Workflow Quality & Best Practices (11)

| ID | Severity | Detection | Reference |
|----|----------|-----------|-----------|
| id | Medium | Job and environment variable ID collisions | |
| timeout-minutes | Medium | Missing timeout-minutes | |
| workflow-call | Medium | Reusable workflow call validation | |
| conditional | Medium | Conditional expression validation | |
| deprecated-commands | Low | Deprecated workflow commands | |
| environment-variable | Low | Environment variable name validation | |
| job-needs | Medium | Job dependency validation | |
| cache-bloat | Low | Cache size efficiency | |
| obfuscation | Medium | Obfuscated code in workflows | |
| expression | Medium | GitHub Actions expression syntax validation | |
| dependabot-github-actions | Medium | Dependabot config for Actions ecosystem | [dependabot-github-actions.md](references/dependabot-github-actions.md) |
