# ai-action-unrestricted-trigger

An AI agent workflow (e.g. Claude Code Action) is triggered by `issue_comment`, `issues`, or `pull_request_target` without any check on who initiated the event. Any GitHub user — including outside contributors and random drive-by accounts — can invoke the agent, consume the org's API budget, and steer it with prompt content.

## Remediation

Gate the workflow on `author_association` so only trusted accounts can trigger it:

```yaml
jobs:
  ai-review:
    if: >-
      contains(github.event.comment.body, '/ai-review') &&
      contains(fromJSON('["MEMBER","OWNER","COLLABORATOR"]'),
               github.event.comment.author_association)
    runs-on: ubuntu-latest
```

`github.event.comment.author_association` values: `OWNER`, `MEMBER`, `COLLABORATOR`, `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, `FIRST_TIMER`, `NONE`. The first three are organization-trust levels; `CONTRIBUTOR` includes anyone who has had a PR merged, which is usually too broad.

## For `pull_request_target` triggers

`pull_request_target` runs with org secrets. Gate on the PR author, not the actor (the actor on a fork PR is the fork owner):

```yaml
if: >-
  contains(fromJSON('["MEMBER","OWNER","COLLABORATOR"]'),
           github.event.pull_request.author_association)
```

## Why this matters

AI agent workflows typically have:
- Long-lived API keys for the LLM provider (billed)
- Write access to repository contents (review comments, suggestions, branch creation)
- Privileged tool invocations (file edits, command execution)

A workflow that lets anyone trigger it is effectively granting that combination to the internet.
