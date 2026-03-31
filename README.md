# core-cicd-templates

Centralized, reusable GitHub Actions workflows and starter templates for the **dev2b-tec** organization.

---

## Contents

| File | Type | Purpose |
|------|------|---------|
| [`.github/workflows/ci-node.yml`](.github/workflows/ci-node.yml) | Reusable workflow | Lint, test and build Node.js projects |
| [`.github/workflows/ci-docker.yml`](.github/workflows/ci-docker.yml) | Reusable workflow | Build (and optionally push) Docker images |
| [`.github/workflows/deploy-pages.yml`](.github/workflows/deploy-pages.yml) | Reusable workflow | Deploy a build artifact to GitHub Pages |
| [`.github/workflows/release.yml`](.github/workflows/release.yml) | Reusable workflow | Automated semantic versioning and release |
| [`.github/workflow-templates/node-ci.yml`](.github/workflow-templates/node-ci.yml) | Starter template | Quickstart for Node.js CI |
| [`.github/workflow-templates/docker-build.yml`](.github/workflow-templates/docker-build.yml) | Starter template | Quickstart for Docker builds |

---

## Usage

### Node.js CI

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: dev2b-tec/core-cicd-templates/.github/workflows/ci-node.yml@main
    with:
      node-version: '20'
      lint-command: 'npm run lint'
      test-command: 'npm test'
      build-command: 'npm run build'
      upload-build-artifact: true
      artifact-path: 'dist'
```

#### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node-version` | Node.js version | `'20'` |
| `working-directory` | Project root | `'.'` |
| `install-command` | Dependency install command | `npm ci` |
| `lint-command` | Lint command (empty = skip) | `npm run lint` |
| `test-command` | Test command (empty = skip) | `npm test` |
| `build-command` | Build command (empty = skip) | `npm run build` |
| `upload-build-artifact` | Upload build output as artifact | `false` |
| `artifact-path` | Build output path | `dist` |
| `artifact-name` | Artifact name | `build` |

---

### Docker Build & Push

```yaml
# .github/workflows/docker.yml
name: Docker
on:
  push:
    branches: [main]
  pull_request:

jobs:
  docker:
    uses: dev2b-tec/core-cicd-templates/.github/workflows/ci-docker.yml@main
    with:
      image-name: ghcr.io/${{ github.repository }}
      registry: ghcr.io
      push: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

#### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `image-name` | Full image name | *(required)* |
| `registry` | Registry hostname | `ghcr.io` |
| `dockerfile` | Path to Dockerfile | `Dockerfile` |
| `context` | Build context | `'.'` |
| `push` | Push image to registry | `false` |
| `platforms` | Target platforms | `linux/amd64` |

---

### Deploy to GitHub Pages

Requires a prior job that uploads a build artifact (e.g. via the Node.js CI workflow).

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    uses: dev2b-tec/core-cicd-templates/.github/workflows/ci-node.yml@main
    with:
      build-command: 'npm run build'
      upload-build-artifact: true
      artifact-path: 'dist'

  deploy:
    needs: build
    uses: dev2b-tec/core-cicd-templates/.github/workflows/deploy-pages.yml@main
    with:
      artifact-name: build
```

---

### Semantic Release

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    uses: dev2b-tec/core-cicd-templates/.github/workflows/release.yml@main
    secrets:
      gh-token: ${{ secrets.GITHUB_TOKEN }}
```

A `.releaserc.json` (or equivalent) is required in your consuming repository to configure semantic-release.

---

## Starter Templates

Starter templates in [`.github/workflow-templates/`](.github/workflow-templates/) appear in the **Actions → New workflow** picker for every repository in the organization, allowing teams to bootstrap quickly.
