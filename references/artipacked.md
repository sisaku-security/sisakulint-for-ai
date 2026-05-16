# artipacked

`actions/checkout` persists the `GITHUB_TOKEN` to `.git/config` by default. Any subsequent step in the same job can read it and use it to push, create comments, or perform other write operations under the bot's identity. The rule fires on every `actions/checkout` usage that does not explicitly opt out.

## Remediation

```yaml
- uses: actions/checkout@<sha>
  with:
    persist-credentials: false
```

Add it on every checkout, even in jobs that don't appear to need it — the protection is cheap and the boundary is hard to maintain otherwise. If a later step genuinely needs the token, pass it explicitly via `env: GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}` so the grant is visible at the call site.

## Why `-fix` does not handle this

`sisakulint -fix on` resolves commit SHAs and rewrites refs, but does not add `with:` keys to existing steps. The fix is always manual. Audit every `actions/checkout` line after running auto-fix.
