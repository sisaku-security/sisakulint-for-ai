# sisakulint skill for AI agents

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that scans and hardens GitHub Actions workflows using [sisakulint](https://github.com/sisaku-security/sisakulint).

## Install

```bash
npx skills add sisaku-security/sisakulint-for-ai
```

## Eval results

| Eval | Description | with_skill | without_skill |
|------|-------------|-----------|---------------|
| CI hardening | Expression injection, SHA pinning, credential persistence | 100% | 50% |
| Supply chain | Untrusted checkout, secret exposure, dependabot | 100% | 75% |
| Clean workflow | False positive avoidance on hardened workflow | 100% | 100% |
| AI agent | Prompt injection, unrestricted triggers, excessive tools | 100% | 100% |
| **Average** | | **100%** | **81%** |
