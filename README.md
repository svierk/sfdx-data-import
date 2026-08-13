# 💾 SFDX Data Import

This repository implements a simple GitHub composite action for uploading records to any kind of Salesforce org from within CI/CD automations based on CSV (Bulk API 2.0) or JSON (sObject Tree) format.

## Usage

After installing the SF CLI and authorizing the relevant org in your GitHub workflow, data can be imported using this action as follows:

```yaml
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v7.0.1
        with:
          persist-credentials: false

      - name: Select Node Version
        uses: svierk/get-node-version@v1.5.1

      - name: Install Dependencies
        run: npm ci

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@v1.1.2

      - name: Salesforce Org Login
        uses: svierk/sfdx-login@v1.4.2
        with:
          sfdx-url: ${{ secrets.SFDX_AUTH_URL }}
          alias: awesome-org

      - name: CSV Data Import
        uses: svierk/sfdx-data-import@v1.3.1
        with:
          file-path: './data/accounts.csv'
          object-type: 'Account'
          external-id: 'Id'
          target-org: awesome-org

      - name: JSON Data Import
        uses: svierk/sfdx-data-import@v1.3.1
        with:
          file-path: './data/accounts.json,./data/contacts.json'
          api-version: '59.0'
```

The following actions were also used in the example workflow to create the prerequisites for the data import:

- [Get Node Version](https://github.com/svierk/get-node-version) | Pulls Node.js version to be used from the _package.json_ of the project
- [SFDX CLI Setup](https://github.com/svierk/sfdx-cli-setup) | Installs the Salesforce CLI and related plugins
- [SFDX Login](https://github.com/svierk/sfdx-login) | Handles Salesforce login using a Salesforce DX authorization URL

The file format is detected automatically from the file extension:

- `.csv` files are uploaded via Bulk API 2.0 and require the `object-type` input (the `external-id` defaults to `Id`).
- `.json` files are imported via the sObject Tree API and may be passed as a comma-separated, in-order list of files.

Any other extension, or a CSV upload without `object-type`, fails the step with a clear error instead of silently doing nothing.

Of course, the data import action can be used flexibly and the respective approach can vary.

## Inputs

| Name          | Required | Default | Description                                                                                          |
| ------------- | -------- | ------- | -------------------------------------------------------------------------------------------------- |
| `file-path`   | yes      |         | Path to the CSV or JSON file(s) to upload. CSV supports a single file; JSON supports a comma-separated, in-order list. |
| `object-type` | no       |         | API name of the Salesforce object. Required for CSV (Bulk API 2.0) uploads.                          |
| `external-id` | no       | `Id`    | Name of the external ID field (or `Id`). Used for CSV (Bulk API 2.0) uploads.                        |
| `target-org`  | no       |         | Username or alias of the target org. Not required if the default org is set.                         |
| `api-version` | no       |         | Override the api version used for api requests, e.g. `59.0`.                                         |
| `step-summary` | no      | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

## Security & versioning

Every `uses:` reference in the snippet above is **pinned to an exact release tag**, e.g. `svierk/sfdx-data-import@v1.3.1`. Do the same in your own pipelines:

- **Never reference a mutable ref** such as `@main` or `@v1`. It runs whatever code sits on that branch/tag at run time - with access to your org credentials - so a compromised or rewritten ref would run unnoticed.
- **Good - pin to an exact release tag** (`@v1.3.1`). Readable, concrete, and bumped through reviewed pull requests.
- **Strictest - pin to a full-length commit SHA** (`@a1b2c3d…`) with the version as a trailing comment. A SHA can never be re-pointed by the publisher; the cost is readability. Worth it for actions from publishers you don't control.
- **Enable [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot) for `github-actions`** in the repository that hosts the workflow, so those pins are bumped for you instead of silently ageing.

This applies to **all** actions your workflow references - this action as well as `actions/*` and any other third-party action.

Beyond pinning, a few rules are worth following in the workflow that calls this action:

- **Secrets travel as secrets** - hand the SFDX Auth URL straight into the `sfdx-url` input of [sfdx-login](https://github.com/svierk/sfdx-login) as shown above. Never interpolate a secret into a `run:` script via `${{ ... }}` - that allows command injection and can leak the value into the log. This action follows the same rule internally: every input is exposed to the shell as an **environment variable** and referenced as `"$FILE_PATH"`, `"$TARGET_ORG"` and so on.
- **Least-privilege `GITHUB_TOKEN`** - declare a `permissions:` block granting only what the job needs. A data import needs no write access at all, so `contents: read` is enough.
- **`persist-credentials: false` on checkout** - the token is not written to `.git/config`, so later steps (SF CLI, third-party actions) cannot reuse it.
- **Validate pull requests with `pull_request`, never `pull_request_target`** - the latter runs with the base repository's secrets, which would let a fork import its own data into your org.

## References

The data import options supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here: 

- [data import tree](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_import_tree.html) (JSON format)
- [data upsert bulk](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_upsert_bulk.html) (CSV format)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-data-import/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-data-import/blob/main/LICENSE).