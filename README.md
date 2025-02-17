# GitHub Action: 42Crunch REST API Static Security Testing

The REST API Static Security Testing action locates REST API contracts that follow the OpenAPI Specification (OAS, formerly known as Swagger) and runs thorough security checks on them. Both OAS v2 and v3 are supported, in both JSON and YAML format.

You can use this action in the following scenarios:

- Add automatic static application security testing (SAST) task to your CI/CD workflows.
- Perform these checks on pull request reviews and/or code merges.
- Flag the located issues in GitHub's Security / Code Scanning Alerts.

The action is powered by 42Crunch [API Security Audit](https://docs.42crunch.com/latest/content/concepts/api_contract_security_audit.htm). Security Audit performs a static analysis of the API definition that includes more than 200 checks on best practices and potential vulnerabilities on how the API defines authentication, authorization, transport, and data coming in and going out. For more details on the checks, see [API Security Encyclopedia](https://apisecurity.io/encyclopedia/content/api-security-encyclopedia.htm).

## Discover APIs in your repositories

By default, this action will:

1. Look for any `.json` and `.yaml` files in the repository.
2. Pick the files that use OpenAPI schema.
3. Perform security audit on the OpenAPI definitions.

This way, you can locate any new or changed API contracts in the repository.

You can fine-tune how the action behaves by specifying specific parts of the repository or filename masks to be included or exclude in the discovery of APIs. You can even disable discovery completely and instead list only specific API files to be checked and map them to your existing APIs in 42Crunch API Security Platform. You configure all these settings in the configuration file `42c-conf.yaml` that you put in the root of your repository. For advanced examples, see [here](https://github.com/42Crunch/resources/tree/master/cicd/42c-conf-examples).

All discovered APIs are uploaded to an API collection in 42Crunch Platform. The action uses the environment variables `GITHUB_REPOSITORY` and `GITHUB_REF` to name the repository and the branch/tag/PR name from where the API collection originated from. During the subsequent action runs, the APIs in the collection are kept in sync with the changes in your repository.

## Use the action in CI/CD to block security issues

Add this action to your CI/CD workflows in GitHub and have it fail on API definitions that contain security issues.

Security Audit gives each API contract an audit score from 0 (very insecure) to 100 (no issues whatsoever). You can use he `min-score` parameter of the GitHub Action to set the threshold for the audit score where the action fails (the default is `75`, if no other value is specified). This helps to catch APIs definitions of bad quality and address the issues already during design time. Additionally, the plugin will enforce Security Quality gates defined at the platform level (default or tag-driven ones).

More advanced failure conditions can be set in the configuration file `42c-conf.yaml`, such as audit score by category (security or data validation), severity level of issues, or even specific issues, specified by their issue ID. For advanced examples, see [here](https://github.com/42Crunch/resources/tree/master/cicd/42c-conf-examples).

## Read detailed actionable reports

Everyy time the action runs, it includes a link to the detailed prioritized actionable report for each of your OpenAPI files:

<img src="images/link_to_detailed_report.jpg" width="1080" />

Follow the links to read the detailed report in 42Crunch Platform:

<img src="images/42Crunch_platform_Security_Audit.png" width="1080" />

## Get code scanning alerts in GitHub

You can also check the issues that the action found directly in GitHub, on the **Security** tab in **Code scanning alerts**.

To enable that, simply include `upload-to-code-scanning:true` to the parameters of the action in your GitHub workflow.

<img src="images/42Crunch_GitHub-REST_API_Code_Scanning_Alerts.png" width="1080" />

Click any of the alerts to see its exact location in your code and to get the details of the vulnerability and the recommended remediation steps.

<img src="images/42Crunch_GitHub-Alert_Details.png" width="1080" />

## Getting started

This action uses 42Crunch API Security Audit service. Before using the action, create a free 42Crunch account at https://platform.42crunch.com/register.

Then follow the steps described in [documentation](https://docs.42crunch.com/latest/content/tasks/integrate_github_actions.htm) to create an API token for the action to authenticate to 42Crunch Platform, and save it as a secret in GitHub.

## Action parameters

### `api-token`

**Required** The API token that the GitHub action uses to authenticate to 42Crunch Platform. Do not put your API token directly in the workflow file! Instead, create a Github secret in your repository settings and refer to it as shown in the example below.

### `min-score`

The minimum audit score that OpenAPI files must reach, otherwise the action fails. Default is `75`.

### `upload-to-code-scanning`

Upload the audit results to [Github Code Scanning](https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/about-code-scanning). Default is `false`

### `ignore-failures`

If set to `true`, forces to complete execution successfully even if the failures conditions (like min-score or SQG criteria) you have set are met. Default is `false`.

This parameter can be useful if you do not use the GitHub action for CI/CD or pull request scenarios, but simply want it to keep updating Code Scanning alerts on each code change.

### `ignore-network-errors`

If set to `true`, forces to complete execution successfully even if a network error has occurred (such as a failure to connect to 4unch Platform, etc). Default is `false`.

### `skip-local-checks`

If set to `true`, disables all local failure conditions (like minimum score) and fails execution only if the criteria defined in SQGs are not met. Default is `false`.

### `platform-url`

The URL where you access 42Crunch Platform URL. Default is `https://platform.42crunch.com`, and most users do not have to change this.

If you are an enterprise customer _not_ accessing 42Crunch Platform at https://platform.42crunch.com, enter the URL you use to access the platform.

### `root-directory`

The root directory that contains the 42c-conf.yaml configuration file and the APIs. If not specified, the current working directory for the plugin is used instead, which normally corresponds to the root of the checked out repository.

### `default-collection-name`

The default collection name used when creating collections for discovered apis.

### `log-level`

Level of details in the logs, one of: `FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`. Default is `INFO`.

### `share-everyone`

Automatically share new API collections with everyone in your organization in 42Crunch Platform. Accepted values are: `OFF`, `READ_ONLY`, `READ_WRITE`. Default is `OFF`.

### `json-report`

Writes Audit report in JSON format to a specified file, optional. Default: `undefined`, no report is written.

### `api-tags`

List of tags to be set on the newly created APIs. Tags are specified in the following format: "category1:name1 category2:name2", optional.

### `sarif-report`

Writes SARIF report to a specified file, optional. Default: `undefined`, no report is written.

### `audit-timeout`

Set the maximum timeout (in seconds) for the audit report. Fail if the result isn't ready within that interval. Default: 600

## Prerequisites

Create an API token in 42Crunch Platform and copy its value into a [repository secret](https://docs.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) named `API_TOKEN`.

## Example usage

```yaml
uses: 42Crunch/api-security-audit-action@v3
with:
  # Please create free account at https://platform.42crunch.com/register
  # Follow these steps to configure API_TOKEN https://docs.42crunch.com/latest/content/tasks/integrate_github_actions.htm
  api-token: ${{ secrets.API_TOKEN }}
```

A typical workflow which checks the contents of the repository and runs Security Audit on each of the OpenAPI files found in the project would look like this:

```yaml
on: [push]

jobs:
  api_audit_job:
    runs-on: ubuntu-latest
    name: Audit OpenAPI files
    steps:
      - uses: actions/checkout@v2
      - uses: 42Crunch/api-security-audit-action@v3
        with:
          api-token: ${{ secrets.API_TOKEN }}
          min-score: 85
```

## Support

The action is maintained by support@42crunch.com. If you run into an issue, or have a question not answered here, you can create a support ticket at [support.42crunch.com](https://support.42crunch.com/).

When reporting an issue, do include:

- The version of the GitHub action
- Relevant logs and error messages
- Steps to reproduce the issue
