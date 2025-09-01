# Usage Examples

## deploy-gitops-gar

```yml
name: Deploy
on:
  release:
    types: [published]

jobs:
  deploy:
    uses: apprevenew-com/github-workflows/.github/workflows/deploy-gitops-gar.yml@v1
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: ${{ vars.GAR_REGISTRY }}
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}
```

## release-on-merge-to-main

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
    with:
      base-branch: "main"
```

## require-version-label

```yml
name: Require version label
on:
  pull_request:
    types: [opened, edited, labeled, unlabeled, synchronize]

jobs:
  guard:
    uses: apprevenew-com/github-workflows/.github/workflows/require-version-label.yml@v1
```

## build-and-push-pr-to-stage

```yml
name: Build and Push PR to Stage

on:
  pull_request:
    types: [opened, labeled]

jobs:
  prepare:
    if: contains(github.event.pull_request.labels.*.name, 'deploy:stage')
    runs-on: ubuntu-latest
    outputs:
      pr_tag: ${{ steps.set.outputs.pr_tag }}
      sha_short: ${{ steps.set.outputs.sha_short }}
      project_id: ${{ steps.set.outputs.project_id }}
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - id: set
        run: |
          sha_short=$(git rev-parse --short HEAD)
          echo "sha_short=$sha_short" >> $GITHUB_OUTPUT
          echo "pr_tag=temporary-pr-staging-$sha_short" >> $GITHUB_OUTPUT
          echo "project_id=${{ vars.GCP_PROJECT_ID }}" >> $GITHUB_OUTPUT

  call-ci-dispatch:
    needs: prepare
    uses: ./.github/workflows/deploy.yml
    with:
      gcp_project_id: ${{ vars.GCP_PROJECT_ID }}
      gar_registry: us-docker.pkg.dev
      gar_repository: gcf-artifacts-ct
      image_name: case-tracker-api
      dockerfile: cmd/services/api/Dockerfile
      context: .
      gitops_repository: ${{ vars.GITOPS_REPOSITORY }}
      gitops_repository_event_type: image-update
      gitops_repository_include_app_field: true
      allow_prerelease: true
      pr_number: ${{ github.event.pull_request.number }}
    secrets:
      GAR_CREDENTIALS: ${{ secrets.GAR_CREDENTIALS }}
      PAT: ${{ secrets.PAT }}

```
## lint
```yml

name: Lint
on:
  workflow_call:
    inputs:
      go-version:
        required: true
        type: string
      golangci-lint-version:
        required: true
        type: string
jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ inputs.go-version }}
      - name: golangci-lint
        uses: golangci/golangci-lint-action@v8
        with:
          version: ${{ inputs.golangci-lint-version }}

```
## test
```yml

name: Test
on:
  workflow_call:
    inputs:
      go-version:
        required: true
        type: string
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ inputs.go-version }}
      - name: Run tests
        run: go test -v -race ./...
```
