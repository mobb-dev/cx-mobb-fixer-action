# Mobb GitHub action for Checkmarx One GitHub Integration

This action is used alongside the Checkmarx One's GitHub Integration (https://checkmarx.com/resource/documents/en/34965-68678-github-cloud.html) where Checkmarx publishes a scan report in the Pull Request's comment section. This action will monitor the presence of such comment and trigger a job to download the SAST report. The SAST report is submitted to Mobb vulnerability analysis engine and links the URL of the fix report to the PR. If you are using this on a private repo then the Mobb user the API key belongs to must have access to the repo and must approve github access for the user on the Mobb platform beforehand.

![image](https://github.com/mobb-dev/cx-mobb-fixer-action/assets/5158535/407a007e-b140-4643-a3e3-b4c0b2050bbb)


# Pre-Requisites

## Inputs

## `cx-tenant`

**Required** The full path of the SAST report file.

## `cx-api-token`

**Required** your Checkmarx API token

## `cx-base-uri`

**Required** your Checkmarx app url, e.g. "https://ast.checkmarx.net/"

## `cx-base-auth-uri`

**Required** your Checkmarx auth url, e.g. "https://iam.checkmarx.net/"

## `api-key`

**Required** The Mobb API key to use with the action.

## `github-token`

**Required** The GitHub api token to use with the action. Usually available as `${{ secrets.GITHUB_TOKEN }}`.

## Outputs

## `fix-report-url`

The Mobb fix report URL.

## Example usage

```
name: "Mobb/Checkmarx"

on:
  issue_comment:
    types: [created]

jobs:
  report-and-fix:
    name: Get Report and Fix
    if: ${{ github.event.issue.pull_request && contains(github.event.comment.body,'Checkmarx One – Scan Summary & Details') }} # This makes sure that the comment originates from a PR and not an issue comment
    runs-on: 'ubuntu-latest'
    timeout-minutes: 360
    permissions:
      pull-requests: write
      statuses: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Run Mobb GH Fixer monitor for CxOne Comments
        if: always()
        uses: mobb-dev/cx-mobb-fixer-action@main
        with:
          cx-tenant: ${{secrets.CX_TENANT }}
          cx-api-token: ${{ secrets.CX_API_TOKEN  }}
          cx-base-uri: ${{ secrets.CX_BASE_URI }}
          cx-base-auth-uri: ${{ secrets.CX_BASE_AUTH_URI }}
          api-key: ${{ secrets.MOBB_API_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
