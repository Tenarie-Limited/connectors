# SentinelOne Connector

The SentinelOne Connector inventories agents and reviews their reported active
and version states. It reads the Agents API and creates records in Tenarie
without changing endpoints or performing remote operations.

## Included workflows

| Workflow | Purpose |
| --- | --- |
| Agent Inventory | Creates assets with agent, operating system, site, group, and reported state information. |
| Active Agent Presence | Records whether at least one active, installed, non-decommissioned agent is reported. |
| Inactive Agent Review | Creates tasks for eligible agents reported as inactive. |
| Outdated Agent Review | Creates tasks for eligible agents reported as out of date. |
| Eligible Agent Active Result | Reviews reported active state as a draft control test workflow. |
| Eligible Agent Currency Result | Reviews reported version state as a draft control test workflow. |

## Requirements

- Access to Connectors in Tenarie and permission to manage connectors and create
  the assets, tasks, and test results required by your workflows.
- The HTTPS management URL of your SentinelOne Console.
- An API token for a dedicated service user or Console user with `Endpoints.view`
  permission for the intended scope.
- An active control test selected for Active Agent Presence, and active tests
  T-66 and T-67 for the eligible-agent result workflows.

## Setup

1. Create or select a SentinelOne user with the required read access and generate
   its API token.
2. Store the complete authorization value through your organization's approved
   connector credential configuration:

   ```text
   ApiToken SENTINELONE_API_TOKEN
   ```

3. In Tenarie, open **Connectors**, select **Add Connector**, then
   **Import Connector** and upload [sentinel_one.bundle.json](sentinel_one.bundle.json).
4. Replace `https://example.sentinelone.net` with your Console's exact HTTPS
   management URL and confirm the authentication configuration.
5. Confirm the intended Console, Site, or Group scope for each workflow.
6. Select the intended active control test for **Active Agent Presence** and
   confirm the selected tests for the other result workflows.
7. Preview each workflow before activation. The connector and workflows are
   supplied as drafts.

Do not put API tokens in package files, workflow descriptions, or support messages.

## Configuration and interpretation

Active Agent Presence passes when a complete response reports one or more matching
agents and fails for a complete zero count. Authentication and provider failures
are errors. This is a presence check, not proof that every required endpoint is
protected.

The inactive and outdated reviews include agents only when they are not
uninstalled, decommissioned, or pending uninstall. Confirm that the selected
population matches the endpoints you intend to monitor.

Keep the eligible-agent result workflows in draft until the agent list is complete
and agent identities are reliable. For currency results, also confirm what counts
as an acceptable version and that the provider's reported status matches that
policy. An empty agent list does not establish that required agents are active
or current.

## Preview and troubleshooting

Preview does not create destination records. Compare agent identities, counts,
reported states, and presence results with the same Console view before running
workflows or enabling schedules.

- Authentication errors: confirm the token is active and the authorization value
  begins with `ApiToken `.
- Access errors: check `Endpoints.view` and the selected Site or Group access.
- Request errors: check the Console URL and supported agent filters.
- Missing control tests: confirm the selected tests are active in Tenarie.
- Incomplete inventory: reduce the scope to a supported Site or Group.

Agent Inventory supports complete collections of up to 1,099 agents. Exception
reviews read at most five pages of 100 agents; a full final page requires review
for incomplete coverage. Do not use changing page ranges as permanent endpoint
scopes. Threats, policies, vulnerabilities, and remote endpoint operations are
not included.

Workflow definitions are available in [workflows](workflows/).
