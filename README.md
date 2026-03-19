# shared-workflows

Reusable GitHub Actions workflows and composite actions for Node.js, Docker, and npm projects.

## Quick Start

```yaml
# .github/workflows/ci.yml — full CI in 12 lines
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  ci:
    uses: codefuturist/shared-workflows/.github/workflows/ci-node.yml@v1
    with:
      node-version: "24"
    secrets: inherit
```

## Reusable Workflows

### `ci-node.yml` — Node.js CI Pipeline

Full lint, typecheck, test, and build pipeline with optional integration tests and Docker smoke test.

```yaml
jobs:
  ci:
    uses: codefuturist/shared-workflows/.github/workflows/ci-node.yml@v1
    with:
      node-version: "24"           # default: "24"
      run-lint: true               # default: true — runs `pnpm check`
      run-typecheck: true          # default: true — runs `pnpm typecheck`
      run-test: true               # default: true — runs `pnpm test`
      run-build: true              # default: true — runs `pnpm build`
      run-integration: true        # default: false — runs `pnpm test:integration`
      integration-artifact-pattern: "reports/integration-*"  # upload test reports
      docker-smoke-test: true      # default: false — builds Dockerfile (no push)
      docker-image: "app:ci"       # default: "app:ci"
    secrets: inherit
```

### `docker-build-push.yml` — Multi-Platform Docker Build & Push

Builds and pushes Docker images to GHCR and Docker Hub with full tag control via `docker/metadata-action` rules. Supports dual-variant builds (e.g., Debian + Alpine).

```yaml
jobs:
  docker:
    uses: codefuturist/shared-workflows/.github/workflows/docker-build-push.yml@v1
    with:
      images: |
        ghcr.io/myorg/myapp
        dockerhub/myapp
      tags: |
        type=sha,prefix=sha-
        type=semver,pattern={{version}},value=v1.2.3
        type=raw,value=latest
      alpine-tags: |
        type=sha,prefix=sha-,suffix=-alpine
      platforms: "linux/amd64,linux/arm64"  # default
      dockerfile: "Dockerfile"              # default
      dockerfile-alpine: "Dockerfile.alpine" # default
      push: true                             # default
    secrets: inherit
```

### `release-npm.yml` — npm Publish with Provenance

Typechecks, builds, and publishes to npm with SLSA provenance attestation.

```yaml
jobs:
  publish:
    uses: codefuturist/shared-workflows/.github/workflows/release-npm.yml@v1
    with:
      node-version: "24"    # default: "24"
      npm-access: "public"  # default: "public"
      run-typecheck: true   # default: true
      run-build: true       # default: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Composite Actions

### `actions/setup-node-pnpm`

Checkout, enable pnpm via corepack, configure Node.js with caching, and install dependencies.

```yaml
- uses: codefuturist/shared-workflows/actions/setup-node-pnpm@v1
  with:
    node-version: "24"     # default: "24"
    registry-url: ""       # set for npm publishing (e.g. "https://registry.npmjs.org")
    fetch-depth: "1"       # default: "1" — use "0" for full git history
```

### `actions/notify-mattermost`

Send CI/CD notifications to Mattermost with consistent formatting (status emoji, workflow details table, optional alert escalation on failure).

```yaml
- uses: codefuturist/shared-workflows/actions/notify-mattermost@v1
  if: always()
  with:
    webhook-url: ${{ secrets.MATTERMOST_WEBHOOK_CI }}          # required — ci-cd channel
    webhook-url-alert: ${{ secrets.MATTERMOST_WEBHOOK_ALERT }} # optional — system-alerts on failure
    status: ${{ job.status }}                                   # required
    title: "My Workflow"                                        # optional — defaults to workflow name
    message: "Extra details here"                               # optional
    channel: ""                                                 # optional — override channel
```

### `actions/docker-setup`

Set up QEMU, Docker Buildx, and login to GHCR + Docker Hub (Docker Hub is optional).

```yaml
- uses: codefuturist/shared-workflows/actions/docker-setup@v1
  with:
    ghcr-username: ${{ github.actor }}
    ghcr-token: ${{ secrets.GITHUB_TOKEN }}
    dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}  # optional
    dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}        # optional
```

## Versioning

This repo uses **floating major tags** (`@v1`, `@v2`) following the same convention as official GitHub Actions. Pin to `@v1` for stability with automatic patch/minor updates. Use a full SHA for maximum reproducibility.

## Secrets

All workflows support `secrets: inherit` for same-org repos. For cross-org usage, pass secrets explicitly — each workflow documents its required secrets.

| Secret | Used By | Required |
|---|---|---|
| `NPM_TOKEN` | `release-npm.yml` | Yes |
| `GITHUB_TOKEN` | `docker-build-push.yml` (as `GHCR_TOKEN`) | Yes (auto-provided) |
| `DOCKERHUB_USERNAME` | `docker-build-push.yml`, `docker-setup` | No |
| `DOCKERHUB_TOKEN` | `docker-build-push.yml`, `docker-setup` | No |
| `MATTERMOST_WEBHOOK_CI` | `notify-mattermost` | Yes |
| `MATTERMOST_WEBHOOK_ALERT` | `notify-mattermost` | No |

## License

MIT
