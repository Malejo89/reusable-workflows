# Reusable Workflows

Centralized GitHub Actions workflows for the `Malejo89` repositories.

This repository keeps common deployment logic in one place so individual sites only need a small caller workflow with their own secrets and deployment options.

## Available workflows

### FTP Deploy

Reusable FTP/FTPS deployment workflow:

```yaml
uses: Malejo89/reusable-workflows/.github/workflows/ftp-deploy.yml@main
```

It wraps [`SamKirkland/FTP-Deploy-Action`](https://github.com/SamKirkland/FTP-Deploy-Action) and supports:

- FTP and FTPS deploys
- optional custom port
- optional remote server directory
- dry-run mode
- per-repository exclude rules
- compatibility with different secret names in caller repositories

## Billing note

Reusable workflow minutes are billed to the caller workflow, not to this repository.

That means:

- If a private repo calls this workflow, the run consumes minutes from that private repo owner/account.
- Making this repository public does not make private caller runs free.
- Public repositories using standard GitHub-hosted runners are free, but private repositories use the account's included quota and then billable minutes.

Because many `Malejo89` repositories are private, rollout should stay manual and batched.

## Basic caller workflow

Use this in a repository that has these secrets:

- `FTP_HOST`
- `FTP_USER`
- `FTP_PASSWORD`

```yaml
name: FTP Deployment

on:
  workflow_dispatch:
    inputs:
      dry_run:
        description: Log planned changes without uploading files.
        required: true
        type: boolean
        default: false

jobs:
  deploy:
    uses: Malejo89/reusable-workflows/.github/workflows/ftp-deploy.yml@main
    with:
      dry_run: ${{ inputs.dry_run }}
      server_dir: ./
      exclude: |
        **/.git*
        **/.git*/**
        **/node_modules/**
        README.md
        .github/**
    secrets:
      ftp_server: ${{ secrets.FTP_HOST }}
      ftp_username: ${{ secrets.FTP_USER }}
      ftp_password: ${{ secrets.FTP_PASSWORD }}
```

## Caller with FTP_SERVER / FTP_USERNAME

Some repositories use `FTP_SERVER` and `FTP_USERNAME` instead of `FTP_HOST` and `FTP_USER`.

```yaml
jobs:
  deploy:
    uses: Malejo89/reusable-workflows/.github/workflows/ftp-deploy.yml@main
    with:
      dry_run: ${{ inputs.dry_run }}
      server_dir: ./
    secrets:
      ftp_server: ${{ secrets.FTP_SERVER }}
      ftp_username: ${{ secrets.FTP_USERNAME }}
      ftp_password: ${{ secrets.FTP_PASSWORD }}
```

## Caller with cPanel / FTPS

For repositories using cPanel-specific secret names:

```yaml
jobs:
  deploy:
    uses: Malejo89/reusable-workflows/.github/workflows/ftp-deploy.yml@main
    with:
      dry_run: ${{ inputs.dry_run }}
      protocol: ftps
      local_dir: ./
      exclude: |
        **/.git*
        **/.git*/**
        .env
        docs/**
        scripts/**
    secrets:
      ftp_server: ${{ secrets.CPANEL_FTP_SERVER }}
      ftp_username: ${{ secrets.CPANEL_FTP_USERNAME }}
      ftp_password: ${{ secrets.CPANEL_FTP_PASSWORD }}
      ftp_server_dir: ${{ secrets.CPANEL_FTP_SERVER_DIR }}
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `local_dir` | no | `./` | Local directory to upload. |
| `server_dir` | no | `./` | Remote server directory. |
| `protocol` | no | `ftp` | FTP protocol, usually `ftp` or `ftps`. |
| `port` | no | empty | Optional FTP port. |
| `dry_run` | no | `false` | Logs planned changes without uploading files. |
| `dangerous_clean_slate` | no | `false` | Deletes remote files that are not present locally. Use carefully. |
| `exclude` | no | common Git/node exclusions | Newline-separated exclude patterns. |

## Secrets

| Secret | Required | Description |
| --- | --- | --- |
| `ftp_server` | yes | FTP host/server. |
| `ftp_username` | yes | FTP username. |
| `ftp_password` | yes | FTP password. |
| `ftp_server_dir` | no | Remote server directory as a secret, useful when the path differs per repo. |
| `ftp_port` | no | FTP port as a secret. |

## Safe rollout

Recommended rollout pattern:

1. Convert a small group of repositories first.
2. Keep caller workflows on `workflow_dispatch`.
3. Use `dry_run=true` only when you explicitly want a preview run.
4. Keep the default `dry_run=false` for normal deployments.
5. Inspect the workflow log after each batch before migrating more repositories.
6. Re-enable `push` triggers only for repositories where automatic deploy is wanted.

This avoids accidentally triggering hundreds of deployments and burning GitHub Actions minutes.

## Tested repositories

The first migration batch was tested with manual `dry_run=true` runs:

- `maxi-portfolio`
- `dj-mekxi.com`
- `mcc-workboard`
- `azambaid`
- `VirtualCV`

`dj-mekxi.com` was also tested with `dry_run=false`, replacing `data/releases.json` successfully.

## Maintenance

Update this repository when deployment behavior should change globally. Caller repositories should stay thin and only map local secrets/options into the reusable workflow.

For high-risk changes, create a tagged release and update caller repositories to a pinned tag instead of `@main`.
