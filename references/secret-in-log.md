# secret-in-log

A `${{ secrets.X }}`-derived environment variable reaches a logging sink (`echo`, `printf`, `tee`, redirect to `$GITHUB_OUTPUT` without masking, etc.). The variable's value can leak into the Action's log stream.

## What the rule actually checks

`pkg/core/secretinlog.go` (`hasAddMaskBefore`) looks for a `::add-mask::` workflow command **before** the line that emits the variable to the sink. The mask must be issued before the sink prints, or the rule fires.

## Remediation patterns

### Pattern A — mask before echo

Prepend `echo "::add-mask::$VAR"` immediately before any line that prints a secret-derived value:

```yaml
- run: |
    RECORDS=$(curl -s -H "Authorization: Bearer $API_TOKEN" "$API_URL" | jq -r '.result')
    echo "::add-mask::$RECORDS"        # mask first
    echo "Found records:"
    echo "$RECORDS" | jq -r '.[] | "\(.type) \(.name)"'   # now safe to echo
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
```

Order matters. The runner masks all *future* occurrences of the string. Anything printed before the `::add-mask::` line is unmasked in logs.

### Pattern B — avoid the echo entirely

For `echo "$X" | tool` patterns where the goal is to pipe a secret into a tool, prefer a here-string. The here-string is not emitted to the log:

```yaml
# Before — echoes the secret into the shell
- run: echo -n "$IOS_CERTIFICATE_BASE64" | base64 --decode -o "$CERTIFICATE_PATH"

# After — here-string keeps the secret off stdout
- run: base64 --decode -o "$CERTIFICATE_PATH" <<< "$IOS_CERTIFICATE_BASE64"
```

Process substitution (`< <(...)`) and `<<<` both work; the latter is shorter for single-value inputs.

### Pattern C — split into a separate step

If a multi-line script has many sinks, splitting the secret-touching part into its own step with `env:` makes the boundary auditable and lets you mask once at the top.

```yaml
- name: Resolve resources
  id: resolve
  run: |
    echo "::add-mask::$API_TOKEN"
    RECORDS=$(curl -s -H "Authorization: Bearer $API_TOKEN" "$API_URL")
    echo "::add-mask::$RECORDS"
    echo "records=$RECORDS" >> "$GITHUB_OUTPUT"
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
    API_URL: https://api.example.com/v1/resources
```

## Common false-positive shape

The rule flags any echo of a variable that was assigned from a curl with `Authorization: Bearer $SECRET`. Even if the curl output is purely public (e.g. a list of resource IDs), the rule conservatively treats the derived value as secret-tainted. Mask it anyway — it costs one line and removes the finding.
