# ai-action-excessive-tools

An AI agent workflow grants tools (`Bash`, `Edit`, `Write`, `mcp__github__push_files`, etc.) that allow direct mutation of the repository or runner environment, while the trigger is from untrusted input (issue comments, PR bodies). A prompt-injection payload in that input can steer the agent into using those tools to push malicious code, delete files, or exfiltrate secrets through `Bash`.

## Remediation

Restrict `allowed_tools` to read-only or proposal-only operations. Force every mutation to go through a human-approved PR.

```yaml
# Before — agent can push directly, run shell commands
allowed_tools: |
  Bash
  Edit
  Write
  mcp__github__push_files
  mcp__github__delete_file

# After — agent can read and propose, humans approve
allowed_tools: |
  Read
  Glob
  Grep
  mcp__github__create_pull_request
```

`Bash` is the most dangerous grant — once the agent can execute shell, it can read every secret in the runner environment and exfiltrate via any network call. Don't grant `Bash` to agents triggered by untrusted input.

## Defense in depth

Combine with:
- `ai-action-unrestricted-trigger` mitigation (gate on `author_association`).
- Workflow-level `permissions: contents: read` so even if the agent escapes its sandbox, it can't push.
- A separate human-only workflow for the privileged operations the agent proposes.

## When write tools are necessary

If the agent genuinely needs to commit (e.g. a scheduled refactoring bot), trigger it from `workflow_dispatch` only, with an explicit actor allowlist:

```yaml
on:
  workflow_dispatch:
jobs:
  refactor:
    if: contains(fromJSON('["alice","bob"]'), github.actor)
```

Never grant write tools to a workflow that fires on any user-supplied event.
