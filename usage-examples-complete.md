# Usage Examples

## deploy-gitops-gar
This workflow is used to deploy a new release to a GitOps repository and push images to Google Artifact Registry. It triggers on GitHub release events and updates the target environment automatically.

```yml
name: Deploy
on:
  release:
    types: [published]

jobs:
  deploy:
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-release.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: ${{ vars.GAR_REGISTRY }}
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}
```

## release-on-merge-to-main
This workflow automatically creates a release when a pull request to the main branch is merged. It ensures that production-ready code is released consistently.

```yml
name: Release on merge to main
on:
  pull_request:
    types: [closed]

jobs:
  release:
    uses: apprevenew-com/github-workflows/.github/workflows/release-on-merge-to-main.yml@v1
    secrets:
      PAT: ${{ secrets.PAT }}
```

## require-version-label
This workflow enforces version labeling on pull requests. It blocks merging unless a version label (e.g., vX.Y.Z) is present, ensuring consistent version control and releases.

```yml
name: Require version label
on:
  pull_request:
    types: [opened, edited, labeled, unlabeled, synchronize]

jobs:
  guard:
    uses: apprevenew-com/github-workflows/.github/workflows/require-version-label.yml@v1
```

## build-and-push-to-stage-from-pr
This workflow builds and pushes temporary staging images for multiple services whenever a pull request is opened or labeled for staging deployment. It helps validate changes in a staging environment before production release.

```yml
name: Build and Push to Stage from PR

on:
  pull_request:
    types: [opened, labeled]

jobs:
  build-service-1:
    name: Build & Push service-1
    if: contains(github.event.pull_request.labels.*.name, 'deploy:stage')
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-pr.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-1
      image_prefix: temporary-staging
      pr_number: ${{ github.event.pull_request.number }}
      dockerfile: cmd/services/service-1/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-2:
    name: Build & Push service-2
    if: contains(github.event.pull_request.labels.*.name, 'deploy:stage')
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-pr.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-2
      image_prefix: temporary-staging
      pr_number: ${{ github.event.pull_request.number }}
      dockerfile: cmd/services/service-2/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-3:
    name: Build & Push service-3
    if: contains(github.event.pull_request.labels.*.name, 'deploy:stage')
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-pr.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-3
      image_prefix: temporary-staging
      pr_number: ${{ github.event.pull_request.number }}
      dockerfile: cmd/services/service-3/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-4:
    name: Build & Push service-4
    if: contains(github.event.pull_request.labels.*.name, 'deploy:stage')
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-pr.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-4
      image_prefix: temporary-staging
      pr_number: ${{ github.event.pull_request.number }}
      dockerfile: cmd/service-4/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}
```

## build-and-push-to-prod
This workflow builds and pushes stable production images for multiple services. It is manually triggered and always uses the latest release tag to ensure that production builds are traceable and reproducible.

```yml
name: Build and Push to PROD

on:
  workflow_dispatch:

concurrency:
  group: deploy-${{ github.repository }}-manual
  cancel-in-progress: false

jobs:
  get-latest-release:
    runs-on: ubuntu-latest
    outputs:
      version_tag: ${{ steps.get_release.outputs.version_tag }}
      sha_short: ${{ steps.git_sha.outputs.sha_short }}
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }

      - name: Get latest release tag
        id: get_release
        run: |
          set -euo pipefail
          PREFIX="v"
          LATEST_TAG=$(git tag --list "${PREFIX}[0-9]*" --sort=-v:refname                        | grep -E "^${PREFIX}[0-9]+\.[0-9]+\.[0-9]+$"                        | head -n1)
          if [ -z "$LATEST_TAG" ]; then
            echo "::error::No release tags found matching pattern ${PREFIX}x.y.z"
            exit 1
          fi
          echo "Found latest release: $LATEST_TAG"
          echo "version_tag=$LATEST_TAG" >> $GITHUB_OUTPUT

      - name: Get Git SHA
        id: git_sha
        run: |
          echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

  build-service-1:
    name: Build service-1
    needs: get-latest-release
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-dispatch.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-1
      image_prefix: stable
      git_tag: ${{ needs.get-latest-release.outputs.version_tag }}
      dockerfile: cmd/services/service-1/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-2:
    name: Build service-2
    needs: get-latest-release
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-dispatch.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-2
      image_prefix: stable
      git_tag: ${{ needs.get-latest-release.outputs.version_tag }}
      dockerfile: cmd/services/service-2/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-3:
    name: Build service-3
    needs: get-latest-release
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-dispatch.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-3
      image_prefix: stable
      git_tag: ${{ needs.get-latest-release.outputs.version_tag }}
      dockerfile: cmd/services/service-3/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

  build-service-4:
    name: Build service-4
    needs: get-latest-release
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar-dispatch.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-service
      image_name: service-4
      image_prefix: stable
      git_tag: ${{ needs.get-latest-release.outputs.version_tag }}
      dockerfile: cmd/service-4/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}
```

## ci
This workflow provides continuous integration checks for pull requests. It runs linting with `golangci-lint` and executes Go tests, ensuring code quality and correctness before merging.

```yml
name: CI

on:
  pull_request:
    branches:
      - '**'

jobs:
  lint:
    uses: apprevenew-com/github-workflows/.github/workflows/lint-go-app.yml@v1.5
    with:
      go-version: '1.25'
      golangci-lint-version: 'v2.4'

  test:
    uses: apprevenew-com/github-workflows/.github/workflows/test-go-app.yml@v1.5
    with:
      go-version: '1.25'
```
