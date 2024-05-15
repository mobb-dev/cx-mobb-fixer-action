# Mobb GitHub action

This action is used alongside with Checkmarx One's GitHub Integration (https://checkmarx.com/resource/documents/en/34965-68678-github-cloud.html) capability where Checkmarx publishes a scan report in the Pull Request's comment area. This action can be used to monitor the presence of the checkmarx comment and trigger a job to download the SAST report. The SAST report is submitted to Mobb  vulnerability analysis engine and links the URL of the fix report to the PR. If you are using this on a private repo then the Mobb user the API key belongs to must have access to the repo and must approve github access for the user on the Mobb platform beforehand.

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
# This example utilizes Mobb with Checkmarx via GitHub Actions

on: [pull_request]

jobs:
  Checkmarx-Mobb-example:
    runs-on: ubuntu-latest
    name: Fix Checkmarx findings with Mobb

    steps:
      - name: Checkout repo to get code
        uses: actions/checkout@v3

      - name: Setup Node on this machine
        uses: actions/setup-node@v3.6.0
        with:
          node-version: 18

      - name: Download and configure Checkmarx CLI
        run: |
          wget https://github.com/Checkmarx/ast-cli/releases/download/2.0.54/ast-cli_2.0.54_linux_x64.tar.gz -O checkmarx.tar.gz
          tar -xf checkmarx.tar.gz
          ./cx configure set --prop-name cx_apikey --prop-value ${{ secrets.CX_API_KEY }}
          ./cx configure set --prop-name cx_base_auth_uri --prop-value ${{ secrets.CX_BASE_AUTH_URI }}
          ./cx configure set --prop-name cx_base_uri --prop-value ${{ secrets.CX_BASE_URI }}
          ./cx configure set --prop-name cx_tenant --prop-value ${{ secrets.CX_TENANT }}
        shell: bash -l {0}

      - name: Run Checkmarx SAST scan
        run: ./cx scan create --project-name my-test-project -s ./ --report-format json --scan-types sast --branch nobranch  --threshold "sast-high=1"
        shell: bash -l {0}

      - name: Run Mobb on the findings and get fixes
        if: always()
        uses: mobb-dev/action@v1
        with:
          report-file: "cx_result.json"
          api-key: ${{ secrets.MOBB_API_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
