# GitHub Connector

Availability: Limited Beta

The GitHub Connector helps your organization inventory repositories, detect
repositories that are expected to remain private but currently have broader
visibility, confirm that organization-wide two-factor authentication is
required, and identify organization members who do not have MFA enabled.

The current preview is read-only in GitHub. Tenarie does not change repository
visibility, topics, branch settings, Actions settings, or security alerts.

## Included Workflows

### GitHub Repository Inventory

This Workflow creates or reuses one Tenarie application Asset for each
repository available to the connected GitHub token. Tenarie uses GitHub's
numeric repository ID as the stable provider identity, so renaming a repository
does not create a second Asset.

Each Asset reports the repository's current `public`, `private`, or `internal`
visibility, owner, default branch, archive and disabled state, fork state,
topics, and bounded activity timestamps.

Permission objects, clone URLs, temporary clone tokens, owner profile details,
and repository content are removed by the Connector worker before records reach
Tenarie.

### GitHub Expected Private Repository Exposure Review

This Workflow creates a Tenarie Task when both conditions are true:

- the repository has the `tenarie-private` topic
- GitHub reports that its current visibility is not `private`

This gives the Workflow an explicit policy baseline and avoids treating every
intentionally public repository as a failure. It also detects `internal`
visibility when the repository policy requires strictly private access.

Add the `tenarie-private` topic to every repository that must remain private.
GitHub topic names are always public, including topics created on private
repositories. Do not encode customer names, project names, classifications, or
other sensitive information in the topic.

The topic is a lightweight policy marker for accidental visibility changes. It
is not a tamper-resistant security control: a user who can change repository
topics or visibility may be able to change both values. Protect those GitHub
administrative permissions separately.

### GitHub Organization Members Without MFA Review

This Workflow appends one immutable Control Test Result per execution and
creates a Tenarie Task for every organization member returned by GitHub's
`2fa_disabled` member filter. GitHub exposes this filter only to organization
owners.

GitHub's filtered response is the exception set, so no additional Workflow
filter is required:

- a complete empty response produces a passing Control Test Result
- one or more returned members produce a failing result and remediation Tasks
- incomplete collection produces an inconclusive result
- provider failure produces an error result, never a pass

The Workflow imports as a draft because every customer has a different GitHub
organization name. Before activating it:

1. Open the Workflow's request step.
2. Replace `REPLACE_WITH_GITHUB_ORGANIZATION` in the request path with the
   exact GitHub organization name.
3. Save the request step.
4. Open the Control Test Result destination and select the active tenant-global
   Control Test that should receive this history.
5. Keep **Exception Records** set to `get_1.records`.
6. Preview the Workflow with an organization-owner token.
7. Activate the Workflow only after the Preview returns the expected
   organization members.

The package uses an intentionally unavailable placeholder target so it cannot
select a customer Control Test during import. The Workflow remains Draft and
cannot activate until an authorized user binds the destination to an active
Control Test.

An empty, complete Preview means GitHub returned no direct organization
members from the disabled-2FA filter. Preview never writes a Control Test
Result. A real complete execution appends the pass or fail result.

This first Workflow checks direct organization members. Outside collaborators,
enterprise-managed user identity-provider MFA, insecure MFA method review, and
GitHub Enterprise Server are not included.

### GitHub Organization Two-Factor Authentication Enforcement Review

This Workflow checks the GitHub organization's
`two_factor_requirement_enabled` setting and creates a Tenarie Task when
GitHub does not report the setting as enabled. When enabled, GitHub requires
everyone with access to the organization's repositories to use two-factor
authentication.

The Workflow imports as a draft because every customer has a different GitHub
organization name. Replace `REPLACE_WITH_GITHUB_ORGANIZATION` in the request
path with the exact organization name before previewing it.

GitHub exposes the enforcement setting as part of the full organization
details available to an organization owner. If the setting is missing from the
response, the Workflow creates a Task instead of treating the organization as
compliant. Preview with an organization-owner token and confirm the normalized
record contains `two_factor_requirement_enabled` before activation.

This organization-level Workflow complements the member exception Workflow:
one checks whether enforcement is enabled, while the other identifies direct
members GitHub reports without MFA.

## Before you start

You need:

- A Tenarie Team, Business, or Enterprise account with Connectors available.
- `connectors:write`, `assets:write`, `tasks:write`, and `tests:write`
  permission to configure, preview, and run all included Workflows.
- A deployed and connected Tenarie Connector worker.
- A GitHub fine-grained personal access token with **Metadata: Read-only**
  access to every repository that Tenarie should inventory.
- For the MFA Workflow, **Members: Read-only** organization permission and a
  token belonging to an owner of the organization being reviewed.
- An administrator who can make the token available securely to the Connector
  worker.

A classic personal access token can be used as a broader fallback. It requires
`repo` for private and internal repository inventory, `read:org` for the member
MFA Workflow, and `admin:org` to read the full organization details used by the
two-factor enforcement Workflow. These classic scopes grant substantially more
access than this read-only Connector uses, so prefer a fine-grained token for
production.

Repository coverage is limited to repositories visible to the token. A token
restricted to selected repositories cannot prove organization-wide coverage.
Use an approved token scope that includes the full intended repository set and
review the Preview record count against GitHub before relying on the result.

## Connect GitHub

1. Create a fine-grained GitHub personal access token owned by the organization
   that Tenarie should review.
2. Grant **Metadata: Read-only** and select every repository that Tenarie
   should review.
3. If you will use the MFA Workflows, grant **Members: Read-only** and confirm
   the token's user is an organization owner. GitHub does not require an
   additional fine-grained permission for the organization details endpoint,
   but the owner identity is required to receive its full details.
4. Store the token in the approved secret store used by your Tenarie Connector
   worker. Do not paste it into a Connector package or Workflow field.
5. Add the `tenarie-private` topic to repositories that must remain private.
6. In Tenarie, open **Connectors**.
7. Select **Add Connector**, then **Import Connector**.
8. Upload the GitHub Connector package supplied by Tenarie.
9. Wait for the import to finish and open **GitHub Connector**.
10. Confirm that the Connector worker is online and the GitHub authentication
   reference is available.
11. Configure the organization name in both draft MFA Workflows.
12. Bind the member MFA Workflow's Control Test Result destination to the
    intended active Control Test before previewing or activating it.

## Preview and run the Workflows

Preview each configured Workflow before its first live run. Preview reads and
evaluates source data but does not create Assets or Tasks.

Confirm that:

- expected public and private repositories are present
- each repository reports the correct current visibility
- every expected-private repository has the `tenarie-private` topic
- the exposure review estimates a Task only for marked repositories whose
  visibility is not private
- the two-factor enforcement review returns the organization record and
  estimates a Task only when enforcement is false or not returned
- the member MFA review estimates one Control Test Result per execution and one
  Task for each organization member GitHub returns from the `2fa_disabled`
  filter

When the preview results look correct:

1. Run **GitHub Repository Inventory**.
2. Review the created or reused Assets and their visibility notes.
3. Run **GitHub Expected Private Repository Exposure Review**.
4. Review any Tasks and confirm the repository's intended GitHub visibility
   before changing it.
5. Run **GitHub Organization Members Without MFA Review**.
6. Confirm the Control Test Result passes for a complete empty response or
   fails when GitHub returns one or more members.
7. Review each returned account's access requirement and arrange approved MFA
   enrollment through your GitHub account-administration process.
8. Run **GitHub Organization Two-Factor Authentication Enforcement Review**.
9. If it creates a Task, confirm the token received full organization details
   and enable the organization-wide requirement in GitHub.

An authorized user can run a Workflow manually or create an interval schedule
from the Workflow's **Schedule** tab. Scheduled runs use the selected Run As
user's current permissions, which Tenarie checks again before every run.

## Limits

- The package reads up to 50 repositories in each execution partition and
  supports up to 20 bounded partitions.
- Repository Inventory routes one Asset action per repository and remains
  subject to the 500-write execution ceiling. Split very large repository
  estates across separately scoped Connectors until a larger-estate strategy
  is approved.
- GitHub returns only repositories available to the connected token.
- The MFA Workflow requires an organization-owner token and organization
  Members read permission. It checks direct organization members and does not
  check outside collaborators.
- The member MFA Workflow appends one Control Test Result per execution in
  addition to any per-member Tasks; both actions share the 500-write execution
  ceiling.
- The two-factor enforcement Workflow requires organization-owner visibility
  into full organization details. A missing enforcement field creates a
  finding rather than a passing result.
- Per-step output, cumulative source-record, destination-write, request,
  response-size, execution-time, and 24-hour continuation-expiry limits apply.
- A Workflow execution can perform at most 500 cumulative destination writes.
- GitHub API rate limits also apply.
- Default-branch protection, GitHub Actions policy, and security-alert reviews
  are not included in this first package slice.

Preview and execution records contain approved normalized fields only. Tenarie
stores bounded summaries, hashes, counters, and destination outcomes; it does
not intentionally persist raw GitHub request or response bodies.

## Troubleshooting

- If authentication fails, confirm that the token is active and available to
  the Connector worker.
- If a repository is missing, confirm that the token has access to it and that
  the authenticated user has explicit permission to read it.
- If all private repositories are missing, review the token's repository
  selection and Metadata permission.
- If a repository expected to be private produces no Task, confirm that the
  repository still has the exact lowercase `tenarie-private` topic.
- If the MFA Workflow fails, confirm that its request path contains the correct
  organization, the token user is an organization owner, and the token has
  organization Members read permission.
- If the member MFA Workflow cannot activate after import, open its Control
  Test Result destination and replace the intentionally unavailable package
  target with the intended active Control Test.
- If the MFA Workflow returns no accounts, confirm the run completed without
  authorization, pagination, or provider errors before treating the exception
  set as empty.
- If the enforcement Workflow creates a Task with a value of `not returned`,
  confirm that the token belongs to an organization owner and can read full
  organization details.
- If a run reports an incomplete collection, compare the record count with the
  intended repository set and inspect continuation execution history.
- If a scheduled run pauses, confirm that the Run As user still has access to
  the Connector and every required Tenarie action scope.

Rotate or revoke the GitHub token immediately if it may have been exposed.
