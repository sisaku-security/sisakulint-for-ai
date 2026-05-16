# cache-poisoning

GitHub Actions cache can be poisoned by an attacker who gains write access to it: subsequent jobs that restore the same cache key then execute the attacker's payload. `sisakulint` surfaces two related concerns under this rule.

## When it fires

1. **Cache eviction risk** — A `cache: <pm>` shortcut on `setup-node` / `setup-python` / etc., or a `actions/cache` step that runs on `pull_request` from forks or on workflows with `pull_request_target`, where the cache key is shared with `push`/`main` jobs.
2. **Cache hierarchy exploitation** — A workflow restores a cache that may have been written by a less-trusted workflow (e.g. preview/build job on PRs writes to a key that the deploy/release job reads).

## Remediation

### Drop the cache when the trade-off isn't worth it

The simplest fix on monorepos with parallel install jobs is to remove `cache: <pm>` entirely:

```yaml
# Before — cache: pnpm shared across PR + main jobs
- uses: actions/setup-node@<sha>
  with:
    node-version: 22
    cache: pnpm

# After — install runs from scratch; no cache surface
- uses: actions/setup-node@<sha>
  with:
    node-version: 22
```

Cost: a 30–90 s install time increase per job. Benefit: rule clears, no key collision between trust boundaries.

### Scope the cache key by trust level

If install time matters, separate the cache key by event type so PR caches never feed `main` jobs:

```yaml
- uses: actions/cache@<sha>
  with:
    path: ~/.pnpm-store
    key: pnpm-${{ github.event_name }}-${{ hashFiles('pnpm-lock.yaml') }}
    restore-keys: |
      pnpm-${{ github.event_name }}-
```

`push` and `pull_request` now use disjoint keys. A poisoned PR cache cannot be restored by a `main` build.

### Move cache to a separate, low-privilege workflow

For builds that must use the cache, put the cache-producing step in a workflow that has `permissions: contents: read` only, no secrets, and is never invoked from `pull_request_target`. The consuming workflow then reads but does not write the key.

## Why removing `cache:` is acceptable on small repos

Monorepos with multiple `pnpm install` jobs see real time cost from removing cache; single-package repos rarely notice. If CI runtime is fine without the cache, take that path — the alternatives (key scoping, workflow split) add complexity that outweighs the savings.
