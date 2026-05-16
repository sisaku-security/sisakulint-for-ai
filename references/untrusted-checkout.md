# untrusted-checkout

A `pull_request_target` workflow (or similar privileged trigger) checks out the PR head SHA and then runs code from it — install scripts, lint config, build steps, etc. The PR head is attacker-controlled, but `pull_request_target` runs with write permissions and access to repository secrets. Checkout + run = remote code execution as the bot.

## Remediation

### Prefer `pull_request`

If the workflow doesn't actually need write access to the repository or secrets, switch the trigger:

```yaml
# Before
on: pull_request_target

# After
on: pull_request
```

`pull_request` runs without secrets and without write tokens. The checked-out PR code can't do damage.

### Or isolate the privileged step

When you do need privileged actions (commenting on the PR, labeling, etc.), split the workflow:

1. **Job A** (`pull_request_target`) — performs the privileged action using only metadata from `github.event.pull_request.*`. Never runs `npm install` or executes any file from the PR.
2. **Job B** (`pull_request`) — checks out and runs the PR code. No secrets, no write token.

Never combine "checkout the PR head" and "I have secrets / write permissions" in the same job.

### What never to do under `pull_request_target`

```yaml
# DON'T
- uses: actions/checkout@<sha>
  with:
    ref: ${{ github.event.pull_request.head.sha }}
- run: npm install        # executes attacker's postinstall scripts
- run: npm test           # executes attacker's test config
```

Even pulling the PR's `package.json` and running `npm ci` is enough — lockfile-controlled `preinstall` scripts will run.
