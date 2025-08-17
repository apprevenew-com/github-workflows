# github-workflows

## deploy-gitops-gar

```yml
name: Deploy
on:
  release:
    types: [published]

jobs:
  deploy:
    uses: apprevenew-com/github-workflows/.github/workflows/deploy.yml@v1
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