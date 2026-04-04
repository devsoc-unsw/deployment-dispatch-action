# deployment-dispatch-action

Composite GitHub Action that dispatches image update requests to the deployment repo and waits for the dispatched workflow to complete.

## Prerequisites

The caller repository needs access to these org-level Actions credentials:

- variable: `DEPLOYMENT_DISPATCHER_APP_ID`
- secret: `DEPLOYMENT_DISPATCHER_APP_PRIVATE_KEY`

The GitHub App must be installed on the target org and repository with these repository permissions on the deployment repo:

- `Actions: write`
- `Checks: read`

## Runner support

This action is only supported on GitHub-hosted `ubuntu-22.04` and `ubuntu-24.04` runners.

`ubuntu-slim` is not supported. Other runners are not supported either.

The action shell steps depend on the Ubuntu runner toolchain contract, including `bash`, `jq`, `gh workflow run` support, and GNU `timeout`.

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `deployment-dispatcher-app-id` | yes | — | GitHub App ID |
| `deployment-dispatcher-app-private-key` | yes | — | GitHub App private key |
| `updates` | yes | — | Newline-delimited `image=tag` pairs |
| `owner` | no | `devsoc-unsw` | Owner of the deployment repository |
| `repository` | no | `deployment` | Deployment repository name |
| `workflow` | no | `dispatch-image-update.yml` | Workflow file to dispatch |
| `watch-timeout-seconds` | no | `180` | Maximum time to wait for the dispatched workflow run to finish |

### Update format

```text
ghcr.io/devsoc-unsw/freerooms-backend=abc123
ghcr.io/devsoc-unsw/freerooms-frontend=abc123
```

One update per line. Image should be the full GHCR image name, tag should be the exact tag you just pushed.

Blank lines are ignored. Any non-empty line that is not a valid `image=tag` pair fails the action.

## Outputs

See [`action.yml`](action.yml) for outputs — they are usually not required by callers.

## Examples

### Single image

```yaml
- name: Dispatch deployment
  uses: devsoc-unsw/deployment-dispatch-action@v1
  with:
    deployment-dispatcher-app-id: ${{ vars.DEPLOYMENT_DISPATCHER_APP_ID }}
    deployment-dispatcher-app-private-key: ${{ secrets.DEPLOYMENT_DISPATCHER_APP_PRIVATE_KEY }}
    updates: |
      ghcr.io/devsoc-unsw/freerooms-backend=${{ github.sha }}
```

### Multiple images

```yaml
- name: Dispatch deployment
  uses: devsoc-unsw/deployment-dispatch-action@v1
  with:
    deployment-dispatcher-app-id: ${{ vars.DEPLOYMENT_DISPATCHER_APP_ID }}
    deployment-dispatcher-app-private-key: ${{ secrets.DEPLOYMENT_DISPATCHER_APP_PRIVATE_KEY }}
    updates: |
      ghcr.io/devsoc-unsw/freerooms-backend=${{ github.sha }}
      ghcr.io/devsoc-unsw/freerooms-frontend=${{ github.sha }}
```

### Custom target

```yaml
- name: Dispatch deployment
  uses: devsoc-unsw/deployment-dispatch-action@v1
  with:
    deployment-dispatcher-app-id: ${{ vars.DEPLOYMENT_DISPATCHER_APP_ID }}
    deployment-dispatcher-app-private-key: ${{ secrets.DEPLOYMENT_DISPATCHER_APP_PRIVATE_KEY }}
    owner: devsoc-unsw
    repository: deployment
    workflow: dispatch-image-update.yml
    updates: |
      ghcr.io/devsoc-unsw/notangles=${{ github.sha }}
```
