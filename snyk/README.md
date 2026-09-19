# Snyk Connector

The Snyk Connector inventories targets and projects and reviews application security
findings. It reads Snyk data and creates records in Tenarie without starting scans,
ignoring findings, or applying fixes in Snyk.

## Included workflows

| Workflow | Purpose |
| --- | --- |
| Target Inventory | Creates assets for targets, including reported source privacy. |
| Project Inventory | Creates assets with project details and reported issue counts. |
| Open Issues | Creates vulnerabilities for open, non-ignored security issues. |
| High and Critical Issue Review | Creates tasks for open, non-ignored high or critical package and code issues. |
| Application Finding Threshold Result | Reviews high and critical findings as a draft control test workflow. |

## Requirements

- Access to Connectors in Tenarie and permission to manage connectors and create
  the assets, tasks, vulnerabilities, and test results required by your workflows.
- Snyk API access for the intended organization.
- A service account token with **View Organization** (`org.read`), **View Projects**
  (`org.project.read`), and **View Project history** (`org.project.snapshot.read`).
- The Snyk organization ID from **Organization Settings > General**.
- Active control test T-68 for the result workflow.

## Setup

1. Create or select a Snyk service account with the required read access.
2. Store the complete authorization value through your organization's approved
   connector credential configuration:

   ```text
   token SNYK_API_TOKEN
   ```

3. In Tenarie, open **Connectors**, select **Add Connector**, then
   **Import Connector** and upload [snyk.bundle.json](snyk.bundle.json).
4. Select the correct regional base URL from the table below.
5. Confirm the authentication configuration.
6. In each workflow, replace `REPLACE_WITH_SNYK_ORGANIZATION_ID` with the exact
   organization ID.
7. Confirm the selected control test and preview each intended workflow.
   Workflows are supplied as drafts.

Do not put API tokens in package files, workflow descriptions, or support messages.

## Regional configuration

| Region | Base URL |
| --- | --- |
| US-01 | `https://api.snyk.io/rest` |
| US-02 | `https://api.us.snyk.io/rest` |
| EU-01 | `https://api.eu.snyk.io/rest` |
| AU-01 | `https://api.au.snyk.io/rest` |

Use the HTTPS `/rest` endpoint for the region hosting your organization.

## Finding configuration and interpretation

Open Issues excludes ignored findings and license issues. The high and critical
review includes open, non-ignored package and code security issues. Review findings
against your organization's remediation priorities.

The Application Finding Threshold Result includes ignored findings. Ignoring a
finding does not make it an approved exemption. Keep this workflow in draft until
the application scope, severity classifications, and exemption policy are confirmed
and missing applications or unknown severity values can be handled reliably.

## Preview and troubleshooting

Preview does not create destination records. Confirm the organization, expected
targets and projects, issue severity, and proposed findings before running workflows
or enabling schedules.

- Authentication errors: confirm the token is active and the authorization value
  begins with `token `.
- Access errors: check organization permissions and API availability for the account.
- Missing resources: check the organization ID, regional URL, and token access.
- Missing control test: ask your administrator to make T-68 active.
- Rate limiting: wait for the provider's retry window before running again.
- Incomplete results: check execution status and reduce the selected scope.

Workflows read at most five pages of 100 records. Do not treat a capped collection
as complete coverage. These workflows do not handle license policy, start Snyk
tests, or remediate findings in Snyk.

Workflow definitions are available in [workflows](workflows/).
