# Cloudflare Connector

Availability: Limited Beta

The Cloudflare Connector helps your organization inventory Cloudflare zones
and review selected zone, origin-encryption, and DNS security settings in
Tenarie.

The current preview is read-only in Cloudflare. Tenarie does not create,
change, or delete Cloudflare zones, DNS records, or security settings.

## Included Workflows

### Cloudflare Zone Inventory

This Workflow creates or reuses one Tenarie infrastructure Asset for each
zone available to the connected Cloudflare API token. Tenarie uses the
Cloudflare zone ID as the stable provider identity, so later runs reuse the
same Asset.

New Assets include the zone name, provider status, paused state, zone type,
and last-modified time when Cloudflare returns those fields. Account, plan,
and permission details are removed by the Connector worker before records
reach Tenarie.

### Cloudflare Zone Status Review

This Workflow creates or reuses a Tenarie Task when a zone is paused or
Cloudflare does not report its status as `active`. Review each result to
confirm whether the state is expected before changing the zone.

### Cloudflare DNSSEC Status Review

This Workflow checks the DNSSEC status for each available zone and creates or
reuses a Tenarie Task when the status is not `active`. DNSSEC setup can depend
on registrar and parent-zone configuration, so review the reported state
before enabling or repairing DNSSEC.

The Connector keeps only the zone identity, DNSSEC status, and last-modified
time. DS records, digests, public keys, and other DNSSEC response fields are
removed by the Connector worker before records reach Tenarie.

### Cloudflare Encryption Mode Review

This Workflow checks the current origin encryption mode for each available
zone. It creates or reuses a Tenarie Task unless Cloudflare reports either:

- `strict`, shown in Cloudflare as **Full (strict)**
- `origin_pull`, shown on eligible Enterprise zones as **Strict (SSL-Only
  Origin Pull)**

Full mode uses the API value `full`. It encrypts traffic to the origin but
does not validate the origin certificate, so this Workflow treats it as
requiring review. Off, Flexible, missing, and unknown values also require
review.

Before changing a setting, confirm that the origin accepts HTTPS and presents
a current certificate for the requested hostname. The Workflow is read-only
and does not change the Cloudflare encryption mode.

### Cloudflare Unproxied Public DNS Records Review

This advisory Workflow creates or reuses a Tenarie Task for proxiable `A`,
`AAAA`, or `CNAME` records where Cloudflare proxying is disabled. An unproxied
record can be intentional, so each result requires review rather than proving
a vulnerability by itself.

The Connector keeps the record ID, name, type, TTL, proxy state, and
last-modified time. DNS record content, origin addresses, comments, tags, and
metadata are removed by the Connector worker before records reach Tenarie.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write`, `assets:write`, and `tasks:write` permission to import,
  preview, and run all included Workflows.
- A deployed and connected Tenarie Connector worker.
- A scoped Cloudflare API token with Zone Read, DNS Read, and Zone Settings Read
  permissions for the zones you want Tenarie to review.
- An administrator who can make the token available securely to the Connector
  worker.

Use a scoped API token. The current package does not require a Cloudflare
Global API Key or provider create, edit, or delete permissions.

## Connect Cloudflare

1. In Cloudflare, create an API token with Zone Read, DNS Read, and Zone Settings
   Read permissions.
2. Limit the token to the accounts or zones that Tenarie should review.
3. Store the token in the approved secret store used by your Tenarie
   Connector worker. Do not paste it into a Connector package or Workflow
   field.
4. In Tenarie, open **Connectors**.
5. Select **Add Connector**, then **Import Connector**.
6. Upload the Cloudflare Connector package supplied by Tenarie.
7. Wait for the import to finish and open **Cloudflare Connector**.
8. Confirm that the Connector worker is online and the Cloudflare
   authentication reference is available.

## Preview and run the Workflows

Use Preview before the first live run. Preview reads and evaluates source
data but does not create Assets or Tasks.

When the preview results look correct:

1. Open the required Cloudflare Workflow.
2. Review its source request, filters, destination, and estimated writes.
3. Select **Run Workflow**.
4. Review the execution history and the resulting Assets or Tasks.

An authorized user can run a Workflow manually or create an interval schedule
from the Workflow's **Schedule** tab. Scheduled runs use the selected Run As
user's current permissions, which Tenarie checks again before every run.

## Limits

- The package reads up to 50 zones in each execution partition and supports up
  to 20 bounded partitions.
- DNS record review reads at most five pages of 100 records for each zone in a
  partition. The run fails rather than reporting a complete result when that
  limit is reached before an empty terminal page.
- Per-step output, cumulative source-record, destination-write, request,
  response-size, execution-time, and 24-hour continuation-expiry limits still
  apply.
- A Workflow execution can perform at most 500 cumulative destination writes.
- Cloudflare API rate limits and the zones available to the API token also
  apply.
- Web Application Firewall review is not included in this package version.
  It requires separate validation of Cloudflare plan availability,
  permissions, ruleset semantics, and absent-resource behavior.

Preview and execution records contain approved normalized fields only.
Tenarie stores bounded summaries, hashes, counters, and destination outcomes;
it does not intentionally persist raw Cloudflare request or response bodies.

## Troubleshooting

- If authentication fails, confirm that the API token is active and available
  to the Connector worker.
- If a zone is missing, confirm that the token is allowed to read it.
- If DNSSEC or DNS record requests fail, confirm that the token has DNS Read
  permission for the affected zone.
- If encryption-mode requests fail, confirm that the token has Zone Settings Read
  permission for the affected zone.
- If a run reports an incomplete collection, reduce the zones available to
  the token or split the review across separately scoped Connectors.
- If a scheduled run pauses, confirm that the Run As user still has access to
  the Connector and every required Tenarie action scope.

Rotate or revoke the Cloudflare API token immediately if it may have been
exposed.
