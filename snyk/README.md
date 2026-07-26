# Snyk Connector

Availability: Limited Beta

The Snyk Connector gives Tenarie a read-only view of Snyk Targets, Projects,
and currently open security Issues. It creates tenant-global Assets for Targets
and Projects and tenant-global Vulnerabilities for open, non-ignored security
Issues.

The Connector does not start Snyk Tests, change Snyk Projects, ignore Issues,
or apply fixes in Snyk.

## Included Workflows

### Snyk Target Inventory

Creates one application Asset for each Snyk Target, including its display name
and whether Snyk identifies the source as private. Repository URLs, integration
identifiers, and organization relationships are removed before records reach
Tenarie.

### Snyk Project Inventory

Creates one application Asset for each Snyk Project. The Asset description
includes the Project type, origin, Snyk status, read-only state, creation date,
latest issue-count timestamp, and latest critical, high, medium, and low issue
counts returned by Snyk.

### Snyk Open Issues

Creates one Vulnerability for every open, non-ignored Snyk security Issue.
License Issues are excluded because they require a separate policy and
licensing workflow rather than a Vulnerability record.

The Vulnerability records the Snyk Issue type, effective severity, provider
status, description, and related scan-item identifier. Tenarie's canonical
Vulnerability status remains a Tenarie decision and is not automatically
changed when Snyk later reports a different status.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write`, `assets:write`, and `vulnerabilities:write` permission to
  configure and run all included Workflows.
- A deployed and connected Tenarie Connector worker.
- Snyk API access for the intended Organization. Snyk generally restricts API
  access to Enterprise customers.
- A Snyk service account token for production automation, or a personal API
  token for a short setup test.
- Snyk **View Organization (org.read)**, **View Projects (org.project.read)**,
  and **View Project history (org.project.snapshot.read)** permissions. Target
  inventory uses the same Project read permission.
- The Snyk Organization ID from **Organization Settings > General**.
- An administrator who can make the complete provider-formatted authorization
  value available securely to the Connector worker.

Use the authorization value in this format:

```text
token SNYK_API_TOKEN
```

Do not paste the token into a Connector package, Workflow field, or Tenarie
record.

## Select the correct Snyk region

The package defaults to the Snyk US-01 REST endpoint:

```text
https://api.snyk.io/rest
```

If your Snyk Organization is hosted elsewhere, update the Connector base URL
before previewing:

- US-02: `https://api.us.snyk.io/rest`
- EU-01: `https://api.eu.snyk.io/rest`
- AU-01: `https://api.au.snyk.io/rest`

The base URL must use HTTPS and must end at the regional `/rest` API root.

## Connect Snyk

1. Create or select a least-privilege Snyk service account for the Organization.
2. Grant the account the required read permissions.
3. Store its complete `token ...` authorization value in the approved secret
   store used by the Tenarie Connector worker.
4. In Tenarie, open **Connectors**.
5. Select **Add Connector**, then **Import Connector**.
6. Upload the Snyk Connector package supplied by Tenarie.
7. Open **Snyk Connector** and select the correct regional base URL.
8. Confirm the Connector worker can resolve the Snyk authentication reference.
9. Open each imported Workflow and replace
   `REPLACE_WITH_SNYK_ORGANIZATION_ID` with the exact Organization ID.
10. Preview each Workflow before activating it.

All three Workflows import as Draft so a package can never query an unintended
Organization before an authorized user configures the Organization ID.

## Preview and run the Workflows

Preview reads and normalizes Snyk data but does not create or update Assets or
Vulnerabilities.

For **Snyk Target Inventory**, confirm:

- the expected Targets are present
- private-source values match Snyk
- repository URLs and integration details are absent

For **Snyk Project Inventory**, confirm:

- the expected Projects are present
- Project types and origins are correct
- latest issue counts and their update timestamps are present
- the record count matches the intended Organization coverage

For **Snyk Open Issues**, confirm:

- only open, non-ignored Issues are returned by Snyk
- license Issues estimate no Vulnerability write
- the severity, Issue type, description, and scan-item identifier are present
- the estimated writes remain within Tenarie's execution ceiling

After validation, activate and run one Workflow at a time. Review the created
records before enabling an interval schedule.

## Record synchronization

These Workflows use provider-managed synchronization for approved descriptive
fields:

- Target and Project Assets: title, Asset type, and description
- Snyk Vulnerabilities: name, description, remediation action, and Asset alias

Tenarie does not let Snyk overwrite record owners, workspaces, notes,
relationships, business impact, target dates, or canonical Vulnerability
status.

If a provider record disappears, Tenarie does not delete or automatically close
the destination record. A complete run records the first absence, and a second
complete absence at least 24 hours later records that the source no longer
reports it. Incomplete or failed collection never advances that lifecycle.

## Limits

- Every request pins Snyk REST version `2024-10-15`.
- Each Workflow reads up to five cursor-linked pages of 100 records.
- Next-link pagination does not continue across separate jobs. An Organization
  with more than 500 records in one Workflow requires a separately approved
  partitioning strategy.
- One source step can return at most 500 normalized records.
- One execution can perform at most 500 cumulative destination writes,
  including synchronization and source-lifecycle transitions.
- Snyk API rate limits and product-plan restrictions also apply.
- Snyk Tests, Export jobs, Audit Logs, license-policy handling, provider-side
  remediation, and canonical status synchronization are not included in this
  Preview.

Tenarie retains bounded Preview samples for review. Real executions discard
normalized source records after routing and retain bounded counters, hashes,
summaries, destination provenance, and synchronization metadata rather than raw
Snyk response bodies.

## Troubleshooting

- HTTP 401 indicates that Snyk did not accept the authentication value. Confirm
  the stored value begins with `token ` and that the token is active.
- HTTP 403 indicates that the authenticated account is not authorized for the
  request. Confirm the Organization permissions and that the Snyk plan includes
  the requested API capability.
- HTTP 404 usually indicates an incorrect Organization ID, regional base URL,
  or unavailable resource.
- HTTP 429 indicates Snyk rate limiting. Allow the retry window to pass before
  running the Workflow again.
- If Project Inventory reports a response-shape error, confirm that Snyk
  returned `latest_issue_counts` metadata for the selected API version.
- If a run stops at the five-page limit, do not treat the result as complete.
- If a scheduled run pauses, confirm that the Run As user still has access to
  the Connector and every required Tenarie action scope.

Rotate or revoke the Snyk token immediately if it may have been exposed.
