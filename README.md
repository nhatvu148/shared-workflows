# shared-workflows

Reusable GitHub Actions workflows, called by `uses:` from my other repos.

Public on purpose. A private repo can only share reusable workflows with repos
owned by the **same** user *or* the **same** org — there is no setting that spans
both. Since these are called from user-owned repos, an org, and public repos,
public is the only arrangement that reaches all three. Nothing here is sensitive:
they are job definitions with inputs, no secrets, no infrastructure detail.

## Workflows

| File | Provides |
|---|---|
| `rust-ci.yml` | `cargo fmt --check`, `clippy`, `test` or `check`, with `Swatinem/rust-cache` |
| `node-ci.yml` | `setup-node`, install, type-check, tests, optional Bun |
| `review.yml` | Kaniscope AI review on a PR — wraps `kaniscope-action` |
| `docker-publish.yml` | Build and push a container to GHCR, with version pruning |

`review.yml` and `docker-publish.yml` default `runner` to **ubuntu-latest**, not
self-hosted: Docker container actions and buildx only run on Linux.

## Use

```yaml
jobs:
  rust:
    uses: nhatvu148/shared-workflows/.github/workflows/rust-ci.yml@v1
    with:
      components: clippy, rustfmt
```

Pin a tag, never `@main` — this repo takes commits that should not be able to
break a consumer's CI without an explicit version bump.

## The `runner` input

Every workflow takes `runner`, a **JSON** string accepted in either shape:

```yaml
runner: '["self-hosted","macOS","ARM64"]'   # self-hosted
runner: '"ubuntu-latest"'                   # GitHub-hosted
```

It is read as `runs-on: ${{ fromJSON(inputs.runner) }}`. Callers normally omit it
and inherit the default, which is where the fleet's runner target is decided —
change the default here, move the tag, and every consumer follows.

**Consumers with no matching self-hosted runner must pass `runner` explicitly.**
GitHub does not fall back to hosted runners when no runner matches: the job
queues silently and eventually times out, with no failure and no notification.

## Related

Runner installation and monitoring live in a separate private repo. This one
holds only the workflow definitions.
