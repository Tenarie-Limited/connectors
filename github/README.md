# GitHub Connector

The GitHub Connector inventories repositories and reviews repository visibility
and organization multi-factor authentication (MFA). It reads GitHub data and
creates records in Tenarie without changing GitHub settings.

## Included workflows

| Workflow | Purpose |
| --- | --- |
| Repository Inventory | Creates or reuses an asset for each repository visible to the token. |
| Expected Private Repository Exposure Review | Creates review tasks for repositories marked `expected-private` whose visibility is not private. |
| Organization Members Without MFA Review | Records a control test result and creates tasks for direct organization members without MFA. |
| Organization Two-Factor Authentication Enforcement Review | Creates a task when the organization does not report MFA enforcement as enabled. |
| Organization MFA Enforcement Result | Reviews the organization's MFA requirement as a draft control test workflow. |
| Repository Confidentiality Result | Reviews repository visibility against confidentiality requirements as a draft control test workflow. |

## Requirements

- Access to Connectors in Tenarie and permission to manage connectors and create
  the assets, tasks, and test results required by your selected workflows.
- A GitHub token with read access to every repository you intend to review.
- For a fine-grained token, **Metadata: Read-only** repository permission.
- For organization MFA checks, **Members: Read-only** organization permission
  and a token belonging to an organization owner.
- The active control tests required by the result workflows: T-60, T-61, and T-62.

Coverage is limited to repositories available to the token. Confirm that its
repository selection covers your intended scope.

## Setup

1. Make the GitHub token available through your organization's approved
   connector credential configuration. Do not put credentials in package files.
2. In Tenarie, open **Connectors**, select **Add Connector**, then
   **Import Connector** and upload [github.bundle.json](github.bundle.json).
3. Open the imported connector and confirm its authentication configuration.
4. Use `https://api.github.com` as the base URL.
5. In each organization workflow, replace `REPLACE_WITH_GITHUB_ORGANIZATION`
   in the request path with the exact organization name.
6. Confirm the selected control test for each result workflow.
7. Preview each workflow before running it or enabling a schedule.

## Repository configuration

Add the `expected-private` topic to repositories that must remain private.
The exposure review uses this exact topic and treats both public and internal
visibility as exceptions to a private-only requirement.

Topics are public labels, including on private repositories. Do not put
sensitive information in them. Users who can change repository topics can also
remove this marker, so it is not an independent guarantee of confidentiality.

## MFA configuration and results

The member MFA review checks direct organization members. With complete
collection, no members without MFA produces a passing result; returned members
produce a failing result and review tasks. Incomplete collection is inconclusive,
and collection errors are not passing results.

The enforcement review requires access to full organization details. If the MFA
requirement is not returned, investigate token access before interpreting the
review task as a confirmed policy problem.

Keep the MFA enforcement result workflow in draft until missing policy information
can be distinguished from a compliant policy. Keep the confidentiality result
workflow in draft until the required-private repository set is confirmed and
missing repositories or unknown visibility can be handled reliably.

## Preview and troubleshooting

Preview does not create destination records. Check repository coverage, visibility,
organization identity, and the expected MFA exceptions before running workflows.

- Missing repositories: check token access and repository selection.
- Organization request errors: check the organization name and owner access.
- Missing enforcement information: confirm access to full organization details.
- Missing control tests: ask your administrator to make the required tests active.
- Incomplete collection: review the execution status and reduce the selected scope
  before relying on the result as complete evidence.

These workflows do not assess outside collaborators, authentication method
strength, branch protection, GitHub Actions policy, or GitHub Enterprise Server.
Workflow definitions are available in [workflows](workflows/).
