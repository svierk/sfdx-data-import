# 💾 SFDX Data Import

This repository implements a simple GitHub composite action for uploading records to any kind of Salesforce org from within CI/CD automations based on CSV (Bulk API 2.0) or JSON (sObject Tree) format.

## Usage

After installing the SF CLI and authorizing the relevant org in your GitHub workflow, data can be imported using this action as follows:

```
jobs:
  validation:
    name: Validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Select Node Version
        uses: svierk/get-node-version@main

      - name: Install Dependencies
        run: npm ci

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@main

      - name: Salesforce Org Login
        uses: svierk/sfdx-login@main
        with:
          sfdx-url: ${{ secrets.SFDX_AUTH_URL }}
          alias: awesome-org

      - name: CSV Data Import
        uses: svierk/sfdx-data-import@main
        with:
          file-path: './data/accounts.csv'
          object-type: 'Account'
          external-id: 'Id'
          target-org: awesome-org

      - name: JSON Data Import
        uses: svierk/sfdx-data-import@main
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

## References

The data import options supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here: 

- [data import tree](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_import_tree.html) (JSON format)
- [data upsert bulk](https://developer.salesforce.com/docs/platform/salesforce-cli-reference/guide/cli_reference_data_upsert_bulk.html) (CSV format)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-data-import/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-data-import/blob/main/LICENSE).