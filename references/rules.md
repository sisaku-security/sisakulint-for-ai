# sisakulint Rules Reference

## Code Injection & Expression Safety (9)

| ID | Severity | Detection |
|----|----------|-----------|
| code-injection-critical | Critical | Code injection in privileged triggers |
| code-injection-medium | Medium | Code injection in normal triggers |
| envvar-injection-critical | Critical | Unsafe env var usage in high-risk triggers |
| envvar-injection-medium | Medium | Unsafe env var usage in standard triggers |
| envpath-injection-critical | Critical | PATH manipulation in privileged contexts |
| envpath-injection-medium | Medium | PATH manipulation in regular contexts |
| argument-injection | High | Command-line argument injection |
| output-clobbering | High | Output clobbering via $GITHUB_OUTPUT |
| unsound-contains | Medium | Unsafe contains() usage in conditions |

## Supply Chain & Dependency Security (7)

| ID | Severity | Detection |
|----|----------|-----------|
| commit-sha | High | Action refs not pinned to full commit SHA |
| known-vulnerable-actions | Critical | Actions with known vulnerabilities |
| archived-uses | Medium | Archived/deprecated actions |
| impostor-commit | Critical | Impostor commit attack patterns |
| ref-confusion | High | Branch reference manipulation |
| unpinned-images | High | Container images without version pins |
| action-list | Medium | Action allowlist/blocklist enforcement |

## Credential & Secret Protection (7)

| ID | Severity | Detection |
|----|----------|-----------|
| credentials | Critical | Hardcoded credential patterns (Rego) |
| secret-exposure | High | Excessive secrets exposure |
| unmasked-secret-exposure | High | Unmasked secrets in workflow logs |
| secret-exfiltration | High | Network-based secret extraction |
| secrets-in-artifacts | High | Sensitive data in uploaded artifacts |
| secrets-inherit | Medium | Excessive secrets inheritance |
| artipacked | Medium | Credential persistence vulnerability |

## Pipeline Poisoning & Artifact Integrity (8)

| ID | Severity | Detection |
|----|----------|-----------|
| untrusted-checkout | Critical | Checkout of untrusted PR code |
| untrusted-checkout-toctou-critical | Critical | TOCTOU race conditions in checkout (Critical) |
| untrusted-checkout-toctou-high | High | TOCTOU issues in checkout (High) |
| artifact-poisoning-critical | Critical | Malicious artifact injection in critical flows |
| artifact-poisoning-medium | Medium | Artifact tampering in standard flows |
| cache-poisoning | High | Cache corruption vulnerabilities |
| cache-poisoning-poisonable-step | High | Unsafe steps after vulnerable checkout |
| reusable-workflow-taint | High | Untrusted inputs in reusable workflows |

## Triggers & Access Control (7)

| ID | Severity | Detection |
|----|----------|-----------|
| dangerous-triggers-critical | Critical | Privileged triggers without mitigations |
| dangerous-triggers-medium | Medium | Privileged triggers with partial mitigations |
| permissions | High | Permission scope and value validation |
| bot-conditions | Medium | Bot actor conditions validation |
| improper-access-control | High | Label-based approval bypass |
| self-hosted-runners | Medium | Self-hosted runner security |
| request-forgery | High | SSRF vulnerabilities |

## AI Agent Security (3)

| ID | Severity | Detection |
|----|----------|-----------|
| ai-action-unrestricted-trigger | Critical | AI actions allowing any user to trigger execution |
| ai-action-excessive-tools | High | Dangerous tool grants to AI agents in untrusted triggers |
| ai-action-prompt-injection | High | Untrusted input interpolated into AI agent prompts |

## Workflow Quality & Best Practices (11)

| ID | Severity | Detection |
|----|----------|-----------|
| id | Medium | Job and environment variable ID collisions |
| timeout-minutes | Medium | Missing timeout-minutes |
| workflow-call | Medium | Reusable workflow call validation |
| conditional | Medium | Conditional expression validation |
| deprecated-commands | Low | Deprecated workflow commands |
| environment-variable | Low | Environment variable name validation |
| job-needs | Medium | Job dependency validation |
| cache-bloat | Low | Cache size efficiency |
| obfuscation | Medium | Obfuscated code in workflows |
| expression | Medium | GitHub Actions expression syntax validation |
| dependabot-github-actions | Medium | Dependabot config for Actions ecosystem |
