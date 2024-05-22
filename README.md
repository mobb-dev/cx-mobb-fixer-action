# Mobb Fixer for Checkmarx One GitHub Integration

This action is used alongside the Checkmarx One's GitHub Integration (https://checkmarx.com/resource/documents/en/34965-68678-github-cloud.html) where Checkmarx publishes a scan report in the Pull Request's comment section. 

This action will monitor the presence of such a comment and trigger a job to download the SAST report. The SAST report is submitted to the Mobb vulnerability analysis engine, and a fix is presented to the Pull Request's comment section. 

If you are using this on a private repo, the Mobb user to which the API key belongs must have access to the repo and must approve github access for the user on the Mobb platform beforehand.

![image](https://github.com/mobb-dev/cx-mobb-fixer-action/assets/5158535/da9221ef-1dd2-4b6d-b6ba-aa466b51e887)

## Inputs

## `cx-api-token`

**Required** your Checkmarx API token. [Find out how to get it here](https://checkmarx.com/resource/documents/en/34965-68775-generating-a-refresh-token--api-key-.html). 

## `mobb-api-token`

**Required** The Mobb API token to use with the action. [Find out how to get it here](https://docs.mobb.ai/mobb-user-docs/administration/access-tokens). 

## `github-token`

**Required** The GitHub api token to use with the action. Usually available as `${{ secrets.GITHUB_TOKEN }}`.

## Example usage

Create a file under the path `.github/workflow/mobb.yml`. 

A sample content of the workflow file: 
```
# Mobb/Checkamrx Fixer on pull requests
# This workflow defines the needed steps to run Checkmarx on every pull request and pass the results to Mobb Fixer.
#
# Secrets in use (add your missing ones):
# CX_API_TOKEN - Your Checkmarx credentials (find how to get it here: https://checkmarx.com/resource/documents/en/34965-68775-generating-a-refresh-token--api-key-.html)
# MOBB_API_TOKEN - Tour mobb API Token (find out how to get it here: https://docs.mobb.ai/mobb-user-docs/administration/access-tokens)
# GITHUB_TOKEN - Automatically set by GitHub

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
        uses: mobb-dev/cx-mobb-fixer-action@v2
        with:
          cx-api-token: ${{ secrets.CX_API_TOKEN  }}
          mobb-api-token: ${{ secrets.MOBB_API_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
