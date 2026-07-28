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
3. Add a named user and choose an expiration time.
4. Send that user the activation qURL and one-time-displayed access link.
5. Revoke that user—or every user on the page—whenever needed.

Each user receives an independent LayerV qURL. Revoking one user does not
interrupt anyone else.

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
- A custom AppArmor profile restricts filesystem and network access beyond the
  container boundary.
- Published images include an SBOM, build provenance, and a keyless Cosign
  signature tied to the release workflow.
- App backups contain LayerV credentials and connector private state. Protect
  them as secrets.

On first start, the App uses the bundled LayerV Connector to find or create its
tunnel resource and write the LayerV-issued route identifiers under `/data`.
The connector then registers its identity and stores durable agent state there.
Normal restarts reuse both the route and agent state instead of creating another
connector.

The App passes `/data/connector-state` directly to the connector as its agent
state directory. Do not replace it with a symlink or share it with another
connector instance.

The LayerV key remains in protected App storage because the gateway uses it to
manage qURLs. The connector receives the protected key-file path on startup; it
reuses completed agent state when available and uses the key only when
bootstrap is required.

## Resetting the LayerV connection

Use **Reset LayerV connection** at the bottom of the Gateway page only when
this installation must register as a new connector. The reset first revokes
every local user link, then attempts to delete the corresponding remote qURLs.
It removes the old LayerV credential, connector identity, route configuration,
and connector state, while preserving every access-page definition and its
selected Home Assistant resources.

The App then returns to **Connect to LayerV**. Enter a dedicated API key to
register the new connector, and create new users for the preserved pages.
Existing users and links cannot be restored.

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
