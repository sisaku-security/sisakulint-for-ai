# artifact-poisoning-critical

`actions/download-artifact` extracts to the working directory by default. An attacker who can upload an artifact in any workflow (e.g. a PR build) can place files with names that collide with source paths — `package.json`, `.github/workflows/release.yml`, build scripts. The next job that runs `npm install`, executes a workflow, or sources a script is now running attacker-controlled content.

## Remediation

Always extract to a fresh, isolated directory under `${{ runner.temp }}` and consume from there explicitly.

```yaml
# Before — extracts to $GITHUB_WORKSPACE
- uses: actions/download-artifact@<sha>
  with:
    name: build-output

- run: ./scripts/deploy.sh   # may have been overwritten by the artifact

# After — extract to a scoped temp dir
- uses: actions/download-artifact@<sha>
  with:
    name: build-output
    path: ${{ runner.temp }}/build-output

- run: ./scripts/deploy.sh ${{ runner.temp }}/build-output
```

## Cross-workflow downloads (`dawidd6/action-download-artifact` etc.)

Third-party "download from previous workflow" actions are higher risk: they can pull artifacts from any past workflow run, including ones triggered from forks. Same fix — pin `path:` to a temp directory and never extract into the checkout.

```yaml
- uses: dawidd6/action-download-artifact@<sha>
  with:
    name: terraform-state
    path: ${{ runner.temp }}/state    # not '.'
    workflow_conclusion: success
    if_no_artifact_found: warn

- name: Move state into working dir
  run: |
    cp "${{ runner.temp }}/state/terraform.tfstate" .
```

The explicit `cp` makes the trust boundary visible: an artifact named `terraform.tfstate.backup.sh` or `../../.github/workflows/release.yml` can no longer escape the temp dir into your checkout.

## Don't rely on `name:` filtering alone

Artifact `name:` is set by whoever uploaded it. A malicious workflow can upload an artifact with any name. Combine name filtering with `workflow_conclusion: success` and, where possible, `commit:` SHA matching to limit which runs are eligible.
