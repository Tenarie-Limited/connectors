# DigitalOcean Cloud Firewalls Connector

Availability: Preview

The DigitalOcean Cloud Firewalls Connector helps your organization review
firewall rules in Tenarie. It reads Cloud Firewall configuration from
DigitalOcean and can create Tenarie records when a Workflow finds a condition
that needs attention.

The current preview is read-only in DigitalOcean. Tenarie does not create,
change, or delete DigitalOcean firewalls or firewall rules.

## What the current Workflow checks

The included **DigitalOcean Insecure Firewall Ports Check** Workflow reviews
inbound Cloud Firewall rules and identifies public access to:

- FTP on port 21
- Telnet on port 23
- IPv4 source `0.0.0.0/0`
- IPv6 source `::/0`

For each matching rule, the Workflow creates a Tenarie Task containing the
firewall name, firewall ID, protocol, exposed ports, and source addresses. A
rule that does not meet these conditions does not create a Task.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write` and `tasks:write` permission to import and run the included
  active Workflow.
- A deployed and connected Tenarie Connector worker.
- A DigitalOcean API token with the custom `firewall:read` scope.
- An administrator who can make the token available securely to the Connector
  worker.

The current Workflow needs only `firewall:read`. DigitalOcean lists related
read scopes for tags, Droplets, load balancers, and Kubernetes, but those are
not required for this firewall-rule check.

## Connect DigitalOcean

1. In DigitalOcean, create a custom API token with `firewall:read` access.
2. Store the token in the approved secret store used by your Tenarie Connector
   worker. Do not paste the token into a Connector package or Workflow field.
3. In Tenarie, open **Connectors**.
4. Select **Add Connector**, then **Import Connector**.
5. Upload the DigitalOcean Connector package supplied by Tenarie.
6. Wait for the import to finish and open **DigitalOcean Firewall Connector**.
7. Confirm that the Connector worker is online and the DigitalOcean
   authentication reference is available.

DigitalOcean shows a newly created token only once. Store it securely and
rotate or revoke it immediately if it may have been exposed.

## Preview and run the Workflow

Use Preview before the first live run. Preview reads and evaluates source data
but does not create Tasks.

When the preview results look correct:

1. Open **DigitalOcean Insecure Firewall Ports Check**.
2. Review the source request, filters, destination, and estimated writes.
3. Select **Run Workflow**.
4. Review the execution history and created Tasks.

During Preview availability, an authorized user can run the Workflow manually
or create an interval schedule from the Workflow's **Schedule** tab. Scheduled
runs use the selected Run As user's current permissions, which Tenarie checks
again before every run. Preview itself remains source-only and never creates
Tasks.

## Limits

- The Workflow requests up to 100 firewalls per page.
- A run reads at most five pages.
- Tenarie accepts at most 500 normalized source records from one request step.
- DigitalOcean currently documents limits of 5,000 API requests per hour and
  250 requests per minute for a token. DigitalOcean may return HTTP 429 when a
  limit is reached.

If an account exceeds the Workflow page or record limit, the execution may not
represent the complete firewall estate. Review the execution status and contact
your Tenarie administrator before relying on the result as a complete control
assessment.

## Troubleshooting

### Authentication failed

Confirm that the token is active, includes `firewall:read`, and is available to
the Connector worker. If the token was rotated, the worker secret must be
updated before the next preview or run.

### No Tasks were created

This can be expected. The Workflow creates Tasks only for public IPv4 or IPv6
rules exposing port 21 or 23. Run Preview to confirm how many records reached
each filter and destination.

### DigitalOcean returned a rate-limit error

Wait for the period indicated by DigitalOcean and retry the preview or run.
Avoid repeatedly starting the Workflow while a previous execution is active.

### The import was rejected

Confirm that you uploaded the unmodified DigitalOcean Connector package
supplied by Tenarie and that a Connector with the same name does not already
exist in your organization.

### The result appears incomplete

Check the execution history for pagination or record-limit messages. A run that
reaches the five-page or 500-record boundary requires review before it is used
as evidence of a complete firewall assessment.

## DigitalOcean references

- [Cloud Firewalls API](https://docs.digitalocean.com/reference/api/reference/firewalls/)
- [`firewall:read` scope](https://docs.digitalocean.com/reference/api/scopes/firewall/read/)
- [Create a personal access token](https://docs.digitalocean.com/reference/api/create-personal-access-token/)
- [DigitalOcean API rate limits](https://docs.digitalocean.com/reference/api/reference/public-apis/#rate-limit)
