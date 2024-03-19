# 💾 SFDX Data Import

This repository implements a simple GitHub composite action for uploading records to any kind of Salesforce org from within CI/CD automations based on CSV (Bulk API 2.0) or JSON (sObject Tree) format.

## Usage

After installing the SF CLI and authorising the relevant org in your GitHub workflow, data can be imported using this action as follows:

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

      - name: CSV Data Import
        uses: svierk/sfdx-data-import@main
        with:
          file-path: './data/accounts.csv'
          object-type: 'Account'
          external-id: 'Id'

      - name: JSON Data Import
        uses: svierk/sfdx-data-import@main
        with:
          file-path: './data/accounts.json'
```

The following actions were also used in the example workflow to create the prerequisites for the data import:

- [Get Node Version](https://github.com/svierk/get-node-version) | Pulls the Node.js version to be used from the _package.json_ of the project
- [SFDX CLI Setup](https://github.com/svierk/sfdx-cli-setup) | Installs the Salesforce CLI and related plugins
- [SFDX Login]() | Handles Salesforce login using a Salesforce DX authorization URL

Of course, the data import action can be used flexibly and the respective approach can vary.

## References

The two authorisation options supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here: 

- [data import tree](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_data_commands_unified.htm#cli_reference_data_import_tree_unified) (JSON format)
- [data upsert bulk](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_data_commands_unified.htm#cli_reference_data_upsert_bulk_unified) (CSV format)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-data-import/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-data-import/blob/main/LICENSE).