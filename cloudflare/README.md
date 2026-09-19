# Cloudflare Connector

The Cloudflare Connector inventories zones and reviews selected zone, DNS, and
origin-encryption settings. It reads Cloudflare data and creates records in
Tenarie without changing Cloudflare configuration.

## Included workflows

| Workflow | Purpose |
| --- | --- |
| Zone Inventory | Creates or reuses an asset for each zone available to the token. |
| Zone Status Review | Creates tasks for paused zones or zones not reported as active. |
| DNSSEC Status Review | Creates tasks for zones whose reported DNSSEC status is not active. |
| Encryption Mode Review | Creates tasks for origin encryption settings that require review. |
| Unproxied Public DNS Records Review | Creates advisory tasks for eligible DNS records with proxying disabled. |
| Origin Certificate Validation Result | Reviews reported certificate-validation settings as a draft control test workflow. |
| DNSSEC Reported State Result | Reviews reported DNSSEC status as a draft control test workflow. |

## Requirements

- Access to Connectors in Tenarie and permission to manage connectors and create
  the assets, tasks, and test results required by your selected workflows.
- A scoped Cloudflare API token with **Zone Read**, **DNS Read**, and
  **Zone Settings Read** permissions for the intended zones.
- Active control tests T-63 and T-64 for the result workflows.

The connector uses a scoped API token; a Global API Key is not required.

## Setup

1. Create a Cloudflare API token with the required read permissions and restrict
   it to the zones you intend to review.
2. Make it available through your organization's approved connector credential
   configuration. Do not put credentials in package files.
3. In Tenarie, open **Connectors**, select **Add Connector**, then
   **Import Connector** and upload [cloudflare.bundle.json](cloudflare.bundle.json).
4. Open the imported connector and confirm its authentication configuration.
5. Use `https://api.cloudflare.com` as the base URL.
6. Confirm the selected control tests and preview each intended workflow.

## Configuration and interpretation

The encryption review accepts **Full (strict)** (`strict`) and **Strict
(SSL-Only Origin Pull)** (`origin_pull`). Full mode (`full`) encrypts the
connection but does not validate the origin certificate. Other, missing, or
unknown values require review. Confirm that the origin supports HTTPS with an
appropriate certificate before changing settings in Cloudflare.

An unproxied DNS record may be intentional. Review each task against the service's
requirements rather than treating it as proof of a vulnerability.

Keep the result workflows in draft until the intended zones are confirmed and
missing zones or unknown settings can be handled reliably. Reported DNSSEC status
does not independently verify the DNS chain of trust; reported encryption settings
do not independently test the origin certificate or connection.

## Preview and troubleshooting

Preview does not create destination records. Confirm the zone list, reported
settings, and expected review tasks before running workflows or scheduling them.

- Missing zones: check the token's zone selection and Zone Read permission.
- DNS requests fail: check DNS Read permission for the affected zone.
- Encryption requests fail: check Zone Settings Read permission.
- Missing control tests: ask your administrator to make T-63 and T-64 active.
- Incomplete collection: reduce the selected scope and verify a complete run
  before treating the results as comprehensive.

DNS record review reads at most five pages of 100 records per zone. Provider rate
limits and workflow limits also apply. Web Application Firewall review is not
included. Workflow definitions are available in [workflows](workflows/).
