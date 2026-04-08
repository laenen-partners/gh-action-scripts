# gh-action-scripts

Collection of reusable GitHub Actions workflows.

## Workflows

### CI (`ci.yaml`)

Runs the full CI pipeline: setup, codegen, format, lint, build, test, and vulnerability scan.

Expects the caller repo to have a `Taskfile.yml` and `mise.toml` in the root.

```yaml
jobs:
  ci:
    uses: laenen-partners/gh-action-scripts/.github/workflows/ci.yaml@main
    with:
      upload-artifacts: true # optional, default: false
```

| Input | Type | Default | Description |
|---|---|---|---|
| `upload-artifacts` | `boolean` | `false` | Upload JUnit test results as artifacts |
| `go-mod-private` | `string` | `""` | Value for `GOMODPRIVATE` / `GONOSUMCHECK` (e.g. `github.com/myorg/*`) |
| `go-mod-private-owner` | `string` | `""` | GitHub org owner for the app token |

| Secret | Required | Description |
|---|---|---|
| `GO_MOD_APP_ID` | When using private modules | GitHub App ID for private module access |
| `GO_MOD_APP_PRIVATE_KEY` | When using private modules | GitHub App private key |

### Release (`release.yaml`)

Determines the next semantic version using [go-semver-release](https://github.com/s0ders/go-semver-release), syncs cross-module versions, tags (including Go submodule tags), and creates a GitHub release.

Expects a `.semver.yaml` and `go.mod` in the repo root.

```yaml
jobs:
  release:
    permissions:
      contents: write
    uses: laenen-partners/gh-action-scripts/.github/workflows/release.yaml@main
```

| Input | Type | Default | Description |
|---|---|---|---|
| `go-mod-private` | `string` | `""` | Value for `GOMODPRIVATE` / `GONOSUMCHECK` |
| `go-mod-private-owner` | `string` | `""` | GitHub org owner for the app token |

| Secret | Required | Description |
|---|---|---|
| `GO_MOD_APP_ID` | When using private modules | GitHub App ID for private module access |
| `GO_MOD_APP_PRIVATE_KEY` | When using private modules | GitHub App private key |

### Upgrade Tools (`upgrade-tools.yaml`)

Runs `mise upgrade --bump` and opens a PR if `mise.toml` changed.

```yaml
jobs:
  upgrade:
    permissions:
      contents: write
      pull-requests: write
    uses: laenen-partners/gh-action-scripts/.github/workflows/upgrade-tools.yaml@main
```

### Deploy to Railway (`railway-deploy.yaml`)

Deploys a service to Railway.

```yaml
jobs:
  deploy:
    uses: laenen-partners/gh-action-scripts/.github/workflows/railway-deploy.yaml@main
    with:
      service: my-service
    secrets:
      RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

| Input | Type | Required | Description |
|---|---|---|---|
| `service` | `string` | Yes | Railway service name |

| Secret | Required | Description |
|---|---|---|
| `RAILWAY_TOKEN` | Yes | Railway API token |

## Private Go Modules

For repos that depend on private Go modules, pass the `go-mod-private` inputs and secrets to `ci.yaml` and/or `release.yaml`:

```yaml
jobs:
  ci:
    uses: laenen-partners/gh-action-scripts/.github/workflows/ci.yaml@main
    with:
      go-mod-private: "github.com/myorg/*"
      go-mod-private-owner: "myorg"
    secrets:
      GO_MOD_APP_ID: ${{ secrets.GO_MOD_APP_ID }}
      GO_MOD_APP_PRIVATE_KEY: ${{ secrets.GO_MOD_APP_PRIVATE_KEY }}
```

This creates a GitHub App token scoped to the given owner and configures git to authenticate with it, allowing `go mod download` to fetch private modules.
