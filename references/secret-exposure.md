# secret-exposure

A step's `env:` block grants more secrets than the step actually uses. Each extra secret in the environment is reachable by any subprocess and any third-party action the step shells out to. The rule fires when a step exposes secrets that don't appear in the `run:` body or the called action's documented inputs.

## Remediation

Remove every secret the step doesn't actually consume. Move runtime-only secrets (database URLs, payment keys) to a runtime secrets manager — they don't belong in CI.

```yaml
# Before — 6 secrets exposed to one deploy script
- run: ./scripts/deploy.sh
  env:
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    STRIPE_SECRET_KEY: ${{ secrets.STRIPE_SECRET_KEY }}
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

# After — only the secret the publish step needs
- run: ./scripts/deploy.sh
  env:
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

For AWS, prefer OIDC (`aws-actions/configure-aws-credentials` with `role-to-assume`) over long-lived access keys.

## Why "convenient bundling" is unsafe

A CI step that runs `npm install` may execute arbitrary `postinstall` scripts from any transitive dependency. Every secret in that step's environment is readable by those scripts. The blast radius of one compromised dependency scales with the number of secrets you bundled together.

Keep each step's `env:` minimal: only the secrets the code on the next line needs.
