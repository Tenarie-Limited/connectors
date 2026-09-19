# DigitalOcean Connector

The DigitalOcean Connector inventories infrastructure and reviews selected
firewall, database access, monitoring, and load balancer settings. It reads
DigitalOcean data and creates records in Tenarie without changing provider resources.

## Included workflows

- **Inventory:** standard Droplets, Cloud Firewalls, managed databases, load
  balancers, and monitoring alert policies.
- **Firewall review:** tasks for selected public TCP access rules.
- **Database access review:** tasks and a draft vulnerability workflow for explicit
  unrestricted public access rules, plus a draft control test result workflow.
- **Load balancer review:** tasks for errored load balancers, missing targets,
  and HTTP listeners without HTTP-to-HTTPS redirection.
- **Monitoring review:** tasks for disabled alert policies and policies without
  entity or tag targets.

## Requirements

- Access to Connectors in Tenarie and permission to manage connectors and create
  the assets, tasks, vulnerabilities, and test results required by your workflows.
- A DigitalOcean API token with `database:read`, `firewall:read`, `droplet:read`,
  `monitoring:read`, and `load_balancer:read` permissions.
- Supporting read permissions for the selected resources: `regions:read`,
  `sizes:read`, `actions:read`, `image:read`, `tag:read`, and `vpc:read`.
- Active control test T-65 for the database access result workflow.

Provider create, edit, and delete permissions are not required.

## Setup

1. Create a DigitalOcean API token with the required read permissions.
2. Make it available through your organization's approved connector credential
   configuration. Do not put credentials in package files.
3. In Tenarie, open **Connectors**, select **Add Connector**, then
   **Import Connector** and upload [digital_ocean.bundle.json](digital_ocean.bundle.json).
4. Open the imported connector and confirm its authentication configuration.
5. Use `https://api.digitalocean.com` as the base URL.
6. Confirm the account and resource scope, then preview each intended workflow.
7. Confirm the selected control test before using the database result workflow.

## Configuration and interpretation

The firewall check selects public IPv4 (`0.0.0.0/0`) or IPv6 (`::/0`) TCP rules
with exact port values `21`, `23`, `0`, or `all`. Other ports, port ranges, and
UDP rules are outside this check. No findings does not establish that the firewall
is secure or that services are unreachable.

The database checks identify access rules containing exactly `0.0.0.0/0` or
`::/0`. They do not assess combined address ranges, equivalent representations,
default access, or actual network reachability. Review approved exceptions and
other safeguards before acting on a finding. Keep the result workflow in draft
until the intended databases are confirmed and missing or unknown access rules
can be handled reliably.

Load balancer HTTP redirect findings are advisory: some services intentionally
accept HTTP. Assigned targets do not establish target health. Monitoring checks
review policy configuration, not live incidents or notification delivery.

## Preview and troubleshooting

Preview does not create destination records. Confirm the expected resources,
access rules, and proposed findings before running a workflow or enabling a schedule.

- Authentication errors: check token validity and the required read permissions.
- Missing firewall associations: confirm the token includes `droplet:read`.
- Missing resources: confirm the connected account and preview its inventory.
- Missing control test: ask your administrator to make T-65 active.
- Rate limiting: wait for the period indicated by DigitalOcean before retrying.
- Incomplete results: check execution status and use a smaller supported scope.

Standard Droplet inventory excludes GPU Droplets. Droplet, firewall, monitoring,
and load balancer workflows read at most five pages of 100 resources. Database
access checks support up to 50 database clusters per run. Do not treat a collection
that exceeds its limits as a complete assessment.

Workflow definitions are available in [workflows](workflows/).
