# LayerV Home Assistant Gateway

[LayerV](https://layerv.ai) creates protected, revocable links to narrowly scoped Home Assistant
pages. A guest sees only the entities and actions you approve—not your normal
Home Assistant dashboard, account, or administrative controls.

The App runs the gateway and LayerV qURL Connector together. Home Assistant
provides local API access automatically, so you do not need to create a
long-lived Home Assistant token or expose an inbound router port.

## How it works

1. Create a reusable access page, such as **Cat Sitter**.
2. Add Home Assistant entities and approve the permitted actions.
3. Add a named guest and choose an expiration time.
4. Send that guest the activation qURL and one-time-displayed access link.
5. Revoke that guest—or every guest on the page—whenever needed.

Each guest receives an independent LayerV qURL. Revoking one guest does not
interrupt anyone else.

## Guest activity

Open a page's **Guests** section and select **View activity** beside a guest to
review actions made with that individual access grant. The history shows the
time, entity, approved action, safe action parameters, and whether Home
Assistant accepted the action.

Preview activity is deliberately excluded because it belongs to the Home
Assistant administrator, not the guest. The Gateway does not store guest
tokens, activation qURLs, access links, request headers, or arbitrary request
data in activity history.

After a guest is revoked, their history moves to **Recently ended guests**.
It remains available for 30 days for troubleshooting and accountability, then
is automatically deleted. Use **Delete guest record** to remove the revoked
guest entry and all of its history immediately. Active guest records cannot be
deleted without first revoking that guest. Deleting the entire access page
deletes all of its guest history immediately.

Expired guests are removed automatically from the active guest list the next
time the Gateway page is refreshed or their expired link is used. Their local
access remains denied at the exact expiration time regardless of when cleanup
runs. The Gateway also attempts to remove the expired LayerV qURL and retains
the guest record under **Recently ended guests** for the same 30-day review
period.

The **Security events** section records action attempts made with an
unrecognized token, expired-link attempts, action rate limiting, unapproved
entity or action requests, and actions rejected by Home Assistant. Read-only
page loads and automatic status polls never create invalid-token events.
Attributable events also appear in the individual guest's activity view.
Tokens, request bodies, headers, and IP addresses are not stored.

The Gateway page includes a read-only health panel showing the installed
version, Gateway status, whether LayerV API access is configured, and current
page and guest-link counts. It never displays credentials or access URLs.

## Installation

1. Add this repository to the Home Assistant App Store.
2. Install **LayerV Home Assistant Gateway**.
3. Optionally choose a stable connector ID and entity include/exclude policies.
4. Start the App and open its Web UI.
5. On **Connect to LayerV**, enter a dedicated API key for this installation.
   Recommended scopes are **Read qURLs**, **Create, update & delete qURLs**, and
   **Bootstrap LayerV qURL Connector agents**.
6. After the connector is registered, the Web UI reloads into the Gateway.

## Configuration reference

### `connector_id`

Optional stable name for this Home Assistant installation's LayerV connector.
Leave it empty to generate one automatically. After the first successful
registration, do not change it unless you intentionally reset the App and its
connector state.

### `include_domains`

Optional comma-separated allowlist of Home Assistant domains, for example
`light,cover,lock`. When set, only matching domains (or entities included by
another include rule) appear in the entity picker.

### `include_areas`

Optional comma-separated Home Assistant area IDs. Entities assigned to those
areas appear in the picker.

### `exclude_domains`

Comma-separated domains that must never appear in the picker. Exclusions take
priority over inclusions. Cameras and alarm control panels are excluded by
default.

### `exclude_entities`

Optional comma-separated entity IDs to hide, for example
`lock.front_door,camera.driveway`. Exclusions take priority over inclusions.

### `qurl_max_lifetime_days`

Maximum lifetime the Gateway will offer or accept for a newly created qURL.
Set this to the limit of your LayerV plan: `3` for the Free plan or up to `30`
for plans that permit longer lifetimes. The default is `3`.

The lifetime begins when the qURL is created. Presets and custom whole-number
durations are supported in minutes, hours, or days. This setting is a local
safety ceiling; LayerV remains authoritative and may reject a duration that
exceeds the account's actual plan.

After changing App configuration, save it and restart the App.

## Security and persistence

- Guest access is separate from App administration.
- Home Assistant Ingress is accepted only from the Supervisor Ingress proxy;
  the gateway itself listens only inside the App container.
- The gateway remains authoritative for every allowed entity, service, and
  parameter; browser requests cannot expand a page's permissions.
- Guest access tokens are shown once and persisted only as SHA-256 hashes.
- Page definitions, token hashes, qURL revocation identifiers, connector
  identity, and required secrets persist under `/data`.
- Guest activity is stored in an owner-only SQLite database under `/data`.
  Protect Home Assistant backups accordingly. Revoked-guest records are
  automatically purged after 30 days, and each guest is limited to the 1,000
  most recent actions.
- Security events are retained for 30 days and capped at the 1,000 most recent
  events per page. The Web UI displays the latest 100.
- Version 0.1.36 enforces an explicit AppArmor allowlist for the packaged
  runtime, Gateway data, ordinary TCP/UDP networking, and process supervision.
  Access outside this policy is denied and recorded by Home Assistant.
- Published images include an SBOM, build provenance, and a keyless Cosign
  signature tied to the release workflow.
- App backups contain LayerV credentials and connector private state. Protect
  them as secrets.

### AppArmor enforcement and rollback

The restricted profile is enforced. After an App update, verify startup, page
editing, preview, one temporary guest action, revocation, activity history, and
one App restart. Another LayerV connection reset is not required.

If the App fails only after this enforcement update, review the Home Assistant
host audit journal for `apparmor="DENIED"` events associated with
`layerv_ha_gateway`. Share only sanitized operation names and paths; never
include API keys, tokens, qURLs, access links, request bodies, or Connector
private state. Restore the LayerV App-only backup or reinstall the preceding
version without deleting App data while the missing permission is investigated.

On first start, the App uses the bundled LayerV Connector to find or create its
tunnel resource and write the LayerV-issued route identifiers under `/data`.
The connector then registers its identity and stores durable agent state there.
Normal restarts reuse both the route and agent state instead of creating another
connector.

The App passes `/data/connector-state` directly to the connector as its agent
state directory. Do not replace it with a symlink or share it with another
connector instance.

The LayerV key remains in protected App storage because the gateway uses it to
manage qURLs. The Connector receives the protected key-file path during route
registration and its first agent bootstrap after onboarding or an intentional
reset. Once the expected agent identity, key, tunnel, and configuration files
exist, normal Connector starts receive no LayerV API credential and authenticate
with the persistent identity under `/data/connector-state`. If bootstrap was
interrupted and state is incomplete, the next start receives the protected
key-file path again so bootstrap can finish. The key value is never placed in a
command argument or copied into App options.

## Resetting the LayerV connection

Use **Reset LayerV connection** at the bottom of the Gateway page only when
this installation must register as a new connector. The reset first revokes
every local guest link, then attempts to delete the corresponding remote qURLs.
It removes the old LayerV credential, connector identity, route configuration,
and connector state, while preserving every access-page definition and its
selected Home Assistant resources.

The App then returns to **Connect to LayerV**. Enter a dedicated API key to
register the new connector, and create new guests for the preserved pages.
Existing guests and links cannot be restored.

Do not delete App data or change the connector ID after registration. App
backups contain LayerV credentials and connector private state and must be
protected accordingly.

## Troubleshooting

- If the Web UI does not open, verify the App is running and review its log.
- If no entities appear, review the include/exclude options and restart.
- If Home Assistant reports an update but the update dialog is stale, reload
  Supervisor instead of deleting the App.
- Never paste LayerV API keys, activation qURLs, access links, or preview tokens
  into public support messages.
- Report suspected vulnerabilities privately using the process in
  `SECURITY.md`; never put live credentials in a public issue.

## License and branding

The gateway source code is licensed under the MIT License. LayerV names,
wordmarks, logos, and other brand assets are not included in that license.
See `BRAND_ASSETS.md` in the repository.
