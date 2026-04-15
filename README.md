# sisakulint skill for AI agents

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that scans and hardens GitHub Actions workflows using [sisakulint](https://github.com/sisaku-security/sisakulint).

## Install

```bash
# Claude Code
claude install-skill https://github.com/sisaku-security/sisakulint-for-ai

# Or copy SKILL.md to your .claude/skills/ directory
```

Requires Go 1.23+ (sisakulint is installed automatically on first run).

## Usage

The skill activates when you ask about GitHub Actions workflow security:

```
> Scan my workflows for security issues
> Harden .github/workflows/ci.yml
> Review this workflow for supply chain risks
> このワークフローのセキュリティを確認して
```

The agent will:

1. Run `sisakulint` to detect issues
2. Apply auto-fix (`sisakulint -fix on`) for SHA pinning and timeouts
3. Apply manual remediations for rules auto-fix cannot handle
4. Re-run until zero findings

## Eval results

Evaluated with 4 test cases (20 assertions) comparing with-skill vs without-skill:

| Eval | Description | with_skill | without_skill |
|------|-------------|-----------|---------------|
| CI hardening | Expression injection, SHA pinning, credential persistence | 100% | 50% |
| Supply chain | Untrusted checkout, secret exposure, dependabot | 100% | 75% |
| Clean workflow | False positive avoidance on hardened workflow | 100% | 100% |
| AI agent | Prompt injection, unrestricted triggers, excessive tools | 100% | 100% |
| **Average** | | **100%** | **81%** |

The skill adds the most value for standard hardening tasks where `persist-credentials: false`, step-level `timeout-minutes`, and dependabot configuration are easily overlooked without tooling.

Full eval data is in [`sisakulint-workspace/`](sisakulint-workspace/).

## Structure

```
SKILL.md                    # Skill definition (what agents read)
references/rules.md         # 52 rules across 7 categories
evals/
  evals.json                # 5 test cases with assertions
  files/                    # Workflow fixtures for testing
sisakulint-workspace/       # Eval results (iteration 1-3)
```

## Contributing

Run evals before submitting changes:

1. Edit `SKILL.md`
2. Run test cases from `evals/evals.json` with and without the skill
3. Grade assertions and update `sisakulint-workspace/iteration-N/`
4. Confirm pass rate does not regress

See [agentskills.io/skill-creation/evaluating-skills](https://agentskills.io/skill-creation/evaluating-skills) for the eval methodology.
