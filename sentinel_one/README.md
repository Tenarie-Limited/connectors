# SentinelOne Connector

Availability: Preview

The SentinelOne Connector provides read-only Agent inventory and a focused
Control Test that confirms SentinelOne reports at least one installed and
active Agent.

This Preview is intentionally limited to the SentinelOne Agents API. It does
not retrieve threats, policies, activities, vulnerabilities, application
inventory, remote operations, or other SentinelOne data.

## Included Workflows

### SentinelOne Agent Inventory

Creates one infrastructure Asset for each SentinelOne Agent returned within
the configured scope. Tenarie uses the SentinelOne Agent ID as the stable
provider identity and synchronizes the approved Asset title, type, and
description fields.

The Asset description can include:

- computer name
- Agent version
- operating system and architecture
- SentinelOne Site and Group names
- registration and last-active dates
- active, update, network, uninstall, and decommissioned states
- active protection types

### SentinelOne Active Agent Presence

Adds one immutable result to a selected Tenarie Control Test.

The Workflow asks SentinelOne for the count of Agents matching:

- active
- not uninstalled
- not decommissioned

A complete response passes when SentinelOne reports one or more matching
Agents. A complete zero count fails. Authentication or provider failure is
recorded as an error.

This is a presence check only. It does not prove that every expected endpoint
has SentinelOne installed and does not measure organization-wide EDR coverage.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write` and `assets:write` permission for Agent Inventory.
- `connectors:write` and `tests:write` permission for the presence Control
  Test.
- A deployed and connected Tenarie Connector worker.
- The HTTPS management URL for your SentinelOne Console.
- A dedicated SentinelOne service user or Console user with an API Token.
- SentinelOne `Endpoints.view` permission.
- An active Tenarie Control Test for the presence Workflow.

Use the complete SentinelOne authorization value in this format:

```text
ApiToken SENTINELONE_API_TOKEN
```

Do not paste the token into a Connector package, Workflow field, Tenarie
record, or support message.

## Connect SentinelOne

1. Create or select a least-privilege SentinelOne service user.
2. Grant the user `Endpoints.view`.
3. Generate an API Token for that user.
4. Store the complete `ApiToken ...` authorization value in the approved
   secret store used by the Tenarie Connector worker.
5. In Tenarie, open **Connectors**.
6. Select **Add Connector**, then **Import Connector**.
7. Upload the SentinelOne Connector package supplied by Tenarie.
8. Open the imported Connector and replace
   `https://example.sentinelone.net` with the exact HTTPS management URL for
   your SentinelOne Console.
9. Confirm that the Connector worker can resolve the authentication
   reference.
10. Open **SentinelOne Active Agent Presence** and select the intended active
    Tenarie Control Test.
11. Preview both Workflows before activating them.

The Connector and both Workflows import as Draft. They cannot run on a
schedule until an authorized user verifies the management URL, authentication,
response shape, and Control Test binding.

## Preview and validate

For **SentinelOne Agent Inventory**, confirm:

- Agent IDs and computer names match SentinelOne
- Agent, operating-system, Site, Group, and state fields are correct
- the record count represents the intended SentinelOne scope
- license keys, usernames, directory identities, IP and MAC addresses, serial
  numbers, machine identifiers, proxy addresses, and other excluded fields do
  not appear

For **SentinelOne Active Agent Presence**, confirm:

- SentinelOne accepts the active, uninstalled, and decommissioned filters
- the count matches the same filtered view in SentinelOne
- a positive count plans one passing Control Test Result
- a zero count plans one failing Control Test Result

After Preview validation, run each Workflow manually and review the resulting
Asset or Control Test Result before enabling a schedule.

## Large inventory limits

Agent Inventory uses SentinelOne's numeric `skip` and `limit` parameters. The
first request omits `skip`, then later requests send positive offsets. Each
bounded execution reads at most five pages of 100 Agents and performs no more
than 500 destination writes. When SentinelOne reports another page, Tenarie
continues in a new bounded execution under the same collection cycle.

The supplied Workflow allows at most three executions, 10,000 cumulative
source records, 10,000 cumulative destination writes, and 24 hours, while
retaining the 500-write ceiling for every execution. SentinelOne documents
`skip` only through 1,000, so this package supports a complete collection of up
to 1,099 Agents. Larger Consoles require a separately validated stable Site or
Group partitioning strategy or a future protected-cursor capability.

Tenarie rejects repeated batches before another destination write, prevents
overlapping collection cycles for the Workflow, and reconciles missing Agents
only after a complete final page. Do not create permanent page-range Workflows
because Agent ordering can change between runs.

The count-only presence Workflow is not affected by this inventory limit
because it returns one aggregate count record and appends one result.

## Data handling

The Connector projects an approved field set before records reach Tenarie.
Agent inventory excludes:

- API tokens and license keys
- network-interface details, IP addresses, and MAC addresses
- logged-in usernames and directory identities
- email addresses and external identity-provider identifiers
- serial numbers and machine security identifiers
- proxy configuration and addresses
- tag assignment identities
- unapproved cloud-provider and account metadata

Real workflow executions discard normalized source records after routing.
Tenarie retains bounded counters, hashes, summaries, destination provenance,
and synchronization metadata rather than raw SentinelOne response bodies.
Preview samples remain bounded and short-lived for authorized review.

## Troubleshooting

- HTTP 400 means SentinelOne rejected the request configuration or filter.
  Recheck the Console API version and supported Agent filters.
- HTTP 401 means SentinelOne did not accept the API Token. Confirm the stored
  authorization value begins with `ApiToken ` and rotate the token if needed.
- HTTP 403 means the authenticated user lacks access to the selected scope or
  required permission.
- HTTP 429 means SentinelOne rate-limited the request. Allow the provider retry
  window to pass and review overlapping schedules.
- A response-shape error means the selected Console did not return the
  expected `data` or `pagination.totalItems` contract.
- If Agent Inventory reaches the three-execution or documented `skip` limit
  without a complete final page, treat it as incomplete and configure a
  smaller approved Site or Group scope.

Rotate or revoke the SentinelOne API Token immediately if it may have been
exposed.
