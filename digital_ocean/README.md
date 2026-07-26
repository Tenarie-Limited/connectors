# DigitalOcean Connector

Availability: Limited Beta

The DigitalOcean Connector helps your organization inventory standard
DigitalOcean Droplets, Cloud Firewalls, managed database clusters, and
Monitoring alert policies and Load Balancers; preserve returned firewall
associations; and identify selected high-confidence or advisory firewall,
database, alert-policy, and Load Balancer configuration issues in Tenarie.

The current preview is read-only in DigitalOcean. Tenarie does not create,
change, or delete DigitalOcean resources.

## Included Workflows

### DigitalOcean Database Inventory

This Workflow reads DigitalOcean managed database clusters and creates or
reuses one Tenarie infrastructure Asset for each cluster whose DigitalOcean
status is `online`. Tenarie uses the DigitalOcean database cluster ID as the
stable provider identity so later runs reuse the same Asset.

New Asset records include the cluster name, engine, version, provider status,
region, size, node count, storage size, private-network identifier, and
creation time when DigitalOcean returns those fields. Connection objects,
passwords, certificates, and other non-approved database response fields are
removed by the Connector worker before preview or execution records reach
Tenarie.

### DigitalOcean Database Public Trusted Sources Check

This Workflow supports accounts with up to 50 managed database clusters in
one run. It fails without partial findings when the account exceeds that
limit. For supported accounts, it creates or reuses a Tenarie Task when an IP
trusted source permits all public IPv4 (`0.0.0.0/0`) or IPv6 (`::/0`)
addresses.

Droplet, Kubernetes, application, tag, and restricted IP sources are not
treated as findings. The Workflow is read-only and does not modify database
firewall rules.

### DigitalOcean Droplet Inventory

This Workflow reads the standard Droplets available to the connected
DigitalOcean account and creates or reuses one Tenarie infrastructure Asset
for each Droplet. Tenarie uses the DigitalOcean Droplet ID as the stable
provider identity so later runs reuse the same Asset.

New Asset records include the Droplet name, provider status, region, size, VPC
identifier, returned features, and tags. Reusing an existing Asset links the
Workflow execution to that Asset without overwriting user-managed Asset fields.

### DigitalOcean Firewall Inventory

This Workflow reads the Cloud Firewalls available to the connected
DigitalOcean account and creates or reuses one Tenarie Asset for each
firewall. Tenarie uses the DigitalOcean firewall ID as the stable provider
identity so later runs reuse the same Asset.

New Asset records include the firewall name, provider status, attached Droplet
IDs returned by DigitalOcean, and pending-change information. DigitalOcean
returns `droplet_ids` only when the token includes `droplet:read`.

### DigitalOcean Insecure Firewall Ports Check

This Workflow reviews inbound Cloud Firewall rules and identifies public
IPv4 or IPv6 access to:

- FTP on port 21
- Telnet on port 23
- all ports
- IPv4 source `0.0.0.0/0`
- IPv6 source `::/0`

For each matching rule, the Workflow creates or reuses a Tenarie Task
containing the firewall name, firewall ID, protocol, exposed ports, and source
addresses. A rule that does not meet these conditions does not create a Task.

The Workflow does not treat public HTTP or HTTPS access as a finding. Other
administrative or application ports require organization-specific policy and
are not evaluated in this preview.

### DigitalOcean Load Balancer Inventory

This Workflow reads DigitalOcean Load Balancers and creates or reuses one
Tenarie infrastructure Asset per Load Balancer. Tenarie uses the DigitalOcean
Load Balancer ID as the stable provider identity so later runs reuse the same
Asset.

New Assets include approved provider status, network, address, VPC, size,
target, redirect, TLS policy, timeout, and creation metadata.
Certificate identifiers, domain configuration, project identifiers, complete
forwarding-rule objects, detailed health-check objects, and firewall address
rules are removed by the Connector worker before Preview or execution records
reach Tenarie.

### DigitalOcean Errored Load Balancers Check

This Workflow creates or reuses a Tenarie Task when DigitalOcean reports a
Load Balancer in the `errored` state. The Task asks the user to investigate
provider events, forwarding configuration, targets, and health checks.

### DigitalOcean Load Balancers Without Targets Check

This Workflow creates or reuses a Tenarie Task when a Load Balancer has no
Droplet IDs, Droplet tag, or Regional Load Balancer targets. A Load Balancer
with at least one of those target types is not treated as a finding.

### DigitalOcean Load Balancer HTTP Redirect Review

This Workflow creates an advisory Tenarie Task when a Load Balancer listens on
HTTP port 80 while the DigitalOcean HTTP-to-HTTPS redirect is disabled. The
user must confirm whether plaintext HTTP is intentionally required before
changing the configuration.

This review does not prove certificate validity, backend encryption,
health-check correctness, target health, domain security, or whether a public
firewall rule is appropriate.
TLS passthrough, backend protocols, public listener ports, PROXY protocol, and
firewall policy remain architecture-dependent context rather than automatic
findings.

### DigitalOcean Alert Policy Inventory

This Workflow reads DigitalOcean Monitoring alert policies and creates or
reuses one Tenarie process Asset per policy. Tenarie uses the DigitalOcean
policy UUID as the stable provider identity so later runs reuse the same
Asset.

New Assets include the policy description, enabled state, metric type,
comparison, threshold, evaluation window, entity targets, and tag targets.
Email recipients, Slack channels, webhook URLs, and other notification details
are removed by the Connector worker before Preview or execution records reach
Tenarie.

### DigitalOcean Disabled Alert Policies Check

This Workflow creates or reuses a Tenarie Task for each Monitoring alert
policy that DigitalOcean reports as disabled. The Task asks the user to enable
the policy or formally retire it when it is no longer required.

### DigitalOcean Untargeted Alert Policies Check

This Workflow creates or reuses a Tenarie Task when an alert policy has
neither entity targets nor tag targets. A policy with at least one entity or
tag target is not treated as a finding.

These Workflows review alert-policy configuration. They do not ingest live
alert incidents, prove that every resource has monitoring, or assess
notification-channel coverage.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write`, `assets:write`, and `tasks:write` permission to import,
  preview, and run all included Workflows.
- A deployed and connected Tenarie Connector worker.
- A DigitalOcean API token with the custom `database:read`, `firewall:read`,
  `droplet:read`, `monitoring:read`, and `load_balancer:read` scopes.
- The supporting `regions:read`, `sizes:read`, `actions:read`, and `image:read`
  scopes that DigitalOcean requires with `monitoring:read`.
- The supporting `tag:read` and `vpc:read` scopes that DigitalOcean requires
  with `load_balancer:read`.
- An administrator who can make the token available securely to the Connector
  worker.

The current Workflows do not require provider create, update, or delete scopes.
Live Monitoring alert incidents and provider-side Load Balancer remediation
are not included in this package version.

## Connect DigitalOcean

1. In DigitalOcean, create a custom API token with `database:read`,
   `firewall:read`, `droplet:read`, `monitoring:read`,
   `load_balancer:read`, `regions:read`, `sizes:read`, `actions:read`,
   `image:read`, `tag:read`, and `vpc:read`.
2. Store the token in the approved secret store used by your Tenarie Connector
   worker. Do not paste the token into a Connector package or Workflow field.
3. In Tenarie, open **Connectors**.
4. Select **Add Connector**, then **Import Connector**.
5. Upload the DigitalOcean Connector package supplied by Tenarie.
6. Wait for the import to finish and open **DigitalOcean Connector**.
7. Confirm that the Connector worker is online and the DigitalOcean
   authentication reference is available.

DigitalOcean shows a newly created token only once. Store it securely and
rotate or revoke it immediately if it may have been exposed.

If your organization previously imported **DigitalOcean Firewall Connector**,
keep it until you have validated this package. Import this package as the new
**DigitalOcean Connector**, test all included Workflows, disable the old
schedule, and then enable the corresponding schedule on the new Connector.
Keeping the old Connector preserves its execution history.

## Preview and run the Workflows

Use Preview before the first live run. Preview reads and evaluates source data
but does not create Assets or Tasks.

When the preview results look correct:

1. Open the required DigitalOcean Workflow.
2. Review its source request, filters, destination, and estimated writes.
3. Select **Run Workflow**.
4. Review the execution history and the resulting Assets or Tasks.

An authorized user can run a Workflow manually or create an interval schedule
from the Workflow's **Schedule** tab. Scheduled runs use the selected Run As
user's current permissions, which Tenarie checks again before every run.

## Limits

- The Droplet Workflow requests standard, non-GPU Droplets. GPU Droplets are
  not included in this package version.
- The Droplet, Cloud Firewall, Monitoring alert-policy, and Load Balancer
  Workflows request up to 100 resources per page and read at most five pages.
- DigitalOcean managed database listing is one account-level request. The
  package does not send unsupported page-number or page-size parameters to
  that endpoint.
- Tenarie accepts at most 500 normalized source records from one request step.
- The database trusted-source Workflow fails closed when more than 50 clusters
  are returned because its bounded follow-up request executes once per
  cluster.
- DigitalOcean may return HTTP 429 when the connected token reaches a provider
  rate limit.

If an account exceeds a Workflow page or record limit, the execution may not
represent the complete resource or alert-policy estate. Review the execution
status before relying on it as complete assessment evidence.

The firewall inventory records only associations that DigitalOcean positively
returns. It does not prove that every Droplet has a firewall and does not
create an “unprotected Droplet” finding. The Load Balancer list response does
not prove that assigned targets are currently healthy. Current Asset reuse
also does not overwrite existing Asset fields when provider metadata changes.

## Troubleshooting

### Authentication failed

Confirm that the token is active, includes the documented read scopes, and is
available to the Connector worker. If the token was rotated, update the worker
secret before the next preview or run.

### No Assets or Tasks were created

For an inventory Workflow, confirm the connected DigitalOcean account has the
corresponding standard Droplets, Cloud Firewalls, managed database clusters,
Monitoring alert policies, or Load Balancers and inspect Preview for returned
records.

For a security-review Workflow, no Tasks is a valid result when no matching
public firewall rule, database trusted-source rule, disabled alert policy, or
untargeted alert policy, errored Load Balancer, targetless Load Balancer, or
HTTP redirect review is present. Use Preview to confirm how many records
reached each filter and destination.

### DigitalOcean returned a rate-limit error

Wait for the period indicated by DigitalOcean and retry the preview or run.
Avoid repeatedly starting a Workflow while a previous execution is active.

### The import was rejected

Confirm that you uploaded the unmodified DigitalOcean Connector package
supplied by Tenarie and that a Connector named **DigitalOcean Connector** does
not already exist in your organization.

### The result appears incomplete

Check the execution history for pagination, repeated-page, or record-limit
messages. A Droplet, Cloud Firewall, Monitoring, or Load Balancer run that
reaches the five-page boundary, or any run that reaches a record boundary,
requires review before it is used as complete assessment evidence.

## DigitalOcean references

- [Cloud Firewalls API](https://docs.digitalocean.com/reference/api/reference/firewalls/)
- [`firewall:read` scope](https://docs.digitalocean.com/reference/api/scopes/firewall/read/)
- [Managed Databases API](https://docs.digitalocean.com/products/databases/postgresql/reference/api/)
- [`database:read` scope](https://docs.digitalocean.com/reference/api/scopes/database/read/)
- [Secure managed PostgreSQL clusters](https://docs.digitalocean.com/products/databases/postgresql/how-to/secure/)
- [Droplets API](https://docs.digitalocean.com/products/droplets/reference/api/droplets/)
- [`droplet:read` scope](https://docs.digitalocean.com/reference/api/scopes/droplet/read/)
- [Monitoring API](https://docs.digitalocean.com/reference/api/reference/monitoring/)
- [`monitoring:read` scope](https://docs.digitalocean.com/reference/api/scopes/monitoring/read/)
- [Load Balancers API](https://docs.digitalocean.com/reference/api/reference/load-balancers/)
- [`load_balancer:read` scope](https://docs.digitalocean.com/reference/api/scopes/load_balancer/read/)
- [Load Balancer features](https://docs.digitalocean.com/products/networking/load-balancers/details/features/)
- [Create a personal access token](https://docs.digitalocean.com/reference/api/create-personal-access-token/)
- [DigitalOcean API rate limits](https://docs.digitalocean.com/reference/api/reference/public-apis/#rate-limit)
