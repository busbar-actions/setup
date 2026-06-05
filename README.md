> [!WARNING]
> **`busbar-actions` is under heavy active development — expect breaking changes.**
> These repositories are public, but **not ready for use yet** — please don't depend on them.
> A pilot is starting soon: **[star and watch the busbar-actions organization](https://github.com/busbar-actions)** for the launch of Discussions and the pilot announcement.

# busbar-actions/setup

Download a prebuilt **busbar** action binary from the release host
(`busbar-actions/actions-dist`) and add it to `PATH`. This is the single shared
install step used by every busbar action — it replaces the per-action
tag-resolve / OS-arch / `gh release download` boilerplate so each action's
`action.yml` is a thin install-and-invoke wrapper.

## What it does

This is a pure **composite** action (no compiled binary of its own). Its one step:

1. Maps the runner to a target triple. GitHub-hosted runners are linux/amd64, so
   only `x86_64-unknown-linux-gnu` is recognized; any other `RUNNER_OS`/`RUNNER_ARCH`
   **fails fast** with a `::error::` annotation rather than 404-ing on download.
2. Resolves the release tag — `latest` calls `gh release view` on `binary-repo`,
   otherwise the literal `version` is used.
3. Downloads `<binary>-<triple>` from the matching release into `${RUNNER_TEMP}/busbar-bin`,
   renames it to `<binary>`, and `chmod +x`.
4. Appends the bin dir to `GITHUB_PATH` (so later steps can call the binary by
   name) and writes `bin-path` / `tag` to `GITHUB_OUTPUT`.

## Usage

```yaml
steps:
  - uses: busbar-actions/setup@main
    with:
      binary: sf-schema-dump        # binary name to install (required)
      version: latest               # or a release tag, e.g. v0.2.0
      binary-repo: busbar-actions/actions-dist
```

Most consumers never call `setup` directly — each busbar action invokes it
internally and forwards its own `version` / `binary-repo` inputs.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `binary` | yes | — | Binary name to install (e.g. `sf-pii-scan`, `sf-schema-dump`). Also the on-disk name added to `PATH`. |
| `version` | no | `latest` | Release tag to download (e.g. `v0.1.0`). `latest` resolves the most recent release via `gh release view`. |
| `binary-repo` | no | `busbar-actions/actions-dist` | GitHub repo that publishes the binary releases. |

## Outputs

| Output | Description |
|---|---|
| `bin-path` | Absolute path to the installed binary (`${RUNNER_TEMP}/busbar-bin/<binary>`). |
| `tag` | The resolved release tag the binary was downloaded from. |

## Permissions & auth

The install step shells out to the GitHub CLI (`gh release view` / `gh release
download`) using `GH_TOKEN: ${{ github.token }}`. The default `GITHUB_TOKEN` is
sufficient when `binary-repo` is **public**. If `actions-dist` (or whatever
`binary-repo` you point at) is **private or in another org**, the default token
won't have cross-repo read access — pass a PAT/App token by setting `GH_TOKEN`
on the calling job, or expose the releases publicly.

This action **does not** mint, consume, or hand off any Salesforce / OIDC
credential. It has no token lifecycle to manage and registers no cleanup hook.

## Observability

Being composite, there is no binary and no `github-actions-ux` `Reporter`. The
shell step is hardened with `set -euo pipefail` and emits a single `::error::`
workflow annotation on the unsupported-runner gate, plus one human-readable
"Installed …" line on success. Beyond the gate, download/extract failures
surface only as raw `gh` exit codes / step failure — there is no Job Summary
entry and no annotation for a missing asset or a bad tag.

## Target support

GitHub-hosted runners are linux/amd64, the only target published to
`actions-dist`, so the action recognizes `x86_64-unknown-linux-gnu` only and
fails loudly on any other runner OS/arch. See
[`busbar-actions/actions-dist`](https://github.com/busbar-actions/actions-dist)
for the published binary list and asset-naming scheme (`<binary>-<target-triple>`).
