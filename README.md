# busbar-actions/setup

Downloads a prebuilt **busbar** action binary from the release host
(`busbar-actions/actions-dist`) and adds it to `PATH`. This is the single shared
install step used by every busbar action — it replaces the per-action
tag-resolve / OS-arch / `gh release download` boilerplate.

## Usage

```yaml
- uses: busbar-actions/setup@main
  with:
    binary: sf-schema-dump        # binary name to install
    version: latest               # or a release tag, e.g. v0.2.0
    binary-repo: busbar-actions/actions-dist
```

**Outputs:** `bin-path` (absolute path to the installed binary), `tag` (resolved
release tag).

GitHub-hosted runners are linux/amd64 — the only target published — so the action
fails fast on any other runner OS/arch.
