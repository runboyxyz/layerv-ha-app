# Access Pages for Home Assistant

Access Pages creates protected, revocable links to narrowly scoped Home
Assistant pages. A guest sees only the entities and actions you approve—not
your normal Home Assistant dashboard, account, or administrative controls.
External access is powered by [LayerV](https://layerv.ai).

The App runs the gateway and LayerV qURL Connector together. Home Assistant
provides local API access automatically, so you do not need to create a
long-lived Home Assistant token or expose an inbound router port.

## How it works

1. Create a reusable access page, such as **Cat Sitter**.
2. Add Home Assistant entities and approve the permitted actions.
3. Add a named guest and choose an expiration time.
4. Send that guest the qURL shown by the App; new invitations include their
   Gateway bootstrap in that single link. Existing legacy invitations retain
   their original activation/access-link flow.
5. Revoke that guest—or every guest on the page—whenever needed.

Each guest receives an independent LayerV qURL. Revoking one guest does not
interrupt anyone else.

### Sharing and optional email verification

The guest-link result can be copied, handed to the phone's native share sheet,
or displayed as a QR code for an in-person recipient. QR codes are generated
locally in the browser with the bundled encoder; the links are not uploaded to
a QR service. Anyone who receives or photographs a QR code receives its bearer
capability, so revoke it if delivery is uncertain.

For higher assurance, configure email under **Gateway health → Configure
email**, then enable **Require guest verification** when creating a guest. The
invitation is sent to the saved address, and the guest must enter a six-digit
code before any state or action API is released. This proves access to the
mailbox, not a person's legal identity or a unique physical device. This check
runs **at the Gateway after LayerV admission**, before protected state or
actions. `target_path` scopes the invitation's route; it does not verify identity.
The checkbox is enabled when the current invitation runtime and SMTP delivery
are configured. Opening the guest dialog preserves that availability.

Google sign-in is not implemented in this LayerV Gateway. The separate Nova
Gateway's None/Google/email selector depends on its own NHP Server plugin and
signed identity handoff; those components have not been migrated here. LayerV's
current public qURL API has no external ASP configuration or equivalent
pre-admission identity challenge/continuation contract. Restoring upstream
verification would require LayerV support for that contract, including binding
verified identity to the invited guest and withholding AC admission until the
check succeeds. Gateway email verification remains independently supported.

SMTP requires certificate-validated STARTTLS or implicit TLS. The password is
stored in owner-only App data, never returned by the API, and is not available
to any page endpoint process. Codes expire after ten minutes, allow at most five
incorrect attempts, and may be resent after sixty seconds. Successful
verification lasts at most twelve hours and never beyond the guest deadline;
revocation invalidates the associated sessions.

### Nearby-only controls

When editing an access page, enable **Require proximity for actions** and set
an allowed distance from Home to reduce accidental commands while a guest is
away. Guests can still view live status from anywhere. On their first action,
the page explains the check before the browser requests location permission.
A successful reading is reused for up to five minutes.

The Gateway sends the reading to the HA policy broker, which compares it with
Home Assistant's registered location and authoritative page radius. Home's
coordinates are not returned to the guest browser or public Gateway process,
and guest coordinates are not stored in page data, activity history, or audit
events. Browser location can be spoofed, so this is an accidental-action safety
feature rather than proof of physical presence.

## Guest activity

Configure an administrator alert email and SMTP delivery, or register a device
with the Home Assistant Companion App. Gateway Health shows the available
destinations. In **Configure email & alerts**, select which registered mobile
targets may be used for guest alerts. When creating a guest, enable **Alert me
about guest activity**, choose from the administrator-approved email/mobile
destinations, and select first
successful login, every successful entity action, and/or failed or blocked
actions. The public guest cannot select or change alert recipients.

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
2. Install **Access Pages for Home Assistant**.
3. Optionally choose a stable connector ID and entity include/exclude policies.
4. Start the App and open its Web UI.
5. On **Connect to LayerV**, enter a dedicated API key for this installation.
   Recommended scopes are **Read qURLs**, **Create, update & delete qURLs**, and
   **Bootstrap LayerV qURL Connector agents**.
6. After the connector is registered, the Web UI reloads into the Gateway.

## Configuration reference

### `resource_isolation`

Values: `guest` (default) or `page`.

`guest` allocates one LayerV resource/CRID for each guest grant on a page.
The mapping is **one guest grant → one resource → one qURL**. Revocation disables
the Gateway grant and revokes that guest's whole LayerV resource, automatically
revoking its qURL. Other guests have separate destinations and are preserved.
The LayerV API contract covers all qURLs on a revoked resource; in this design
there is only one qURL on each guest resource. This also prevents new requests
through its previously consumed invitation. The Gateway immediately invalidates
the guest grant/session, persists unfinished upstream cleanup, and retries
pending enforcement. An upstream outage does not leave the local grant active.
The revocation order is: save local revocation, delete the guest's qURL, then
delete its resource. Resource deletion proceeds even if the qURL deletion is
pending, and unfinished cleanup survives a restart. Deleting the qURL first is
an additional barrier to opening the invitation; it is not a guarantee of faster
termination of established LayerV connections.

`page` shares one LayerV resource between the page's modern guest invitations.
Each invitation still has its own qURL, bootstrap secret, and Gateway session.
Guest revocation removes only its qURL at LayerV and its grant at the Gateway;
revoking the shared resource would remove every guest on that page.
The broker therefore uses individual qURL revocation for guest removal in this
mode. Previously issued invitations keep this behavior even if the configuration
is subsequently changed to `guest`.

Gateway enforcement applies immediately to new protected operations once local
revocation is saved. An action already dispatched to Home Assistant cannot be
undone. LayerV enforcement can propagate later: isolated production tests
observed established HTTP streams and WebSockets closing within approximately
30 seconds after resource deletion. After individual qURL deletion, established
connections remained open for at least 60 seconds, although new requests to the
revoked invitation returned 404. These observations are not a guaranteed LayerV
deadline. Natural LayerV session expiry also did not terminate established
connections in the short-lived test. The Gateway explicitly queues upstream
cleanup at grant expiry; guest mode deletes the resource, while page mode
deletes the qURL. Per-page mode therefore has weaker upstream connection revocation;
the Gateway still denies the revoked guest's subsequent protected operations.

Both modes use one Connector daemon with a session per resource. Per-page mode
uses fewer LayerV resources, but same-NAT browsers can share upstream admission
to admitted paths. Gateway sessions remain mandatory in both modes. Separate
resources also do not authenticate individual browsers within the same NAT.
In production tests, fresh browsers behind the same public IP could reach an
already admitted guest prefix without their own qURL. They cannot gain Gateway
authorization without the correct unconsumed bootstrap secret, required guest
verification, and a session bound to the grant and page. Per-guest mode gives
guests different resource-host URLs and prevents one resource's admission from
opening another resource before that resource is admitted. It is a separate
upstream destination/revocation boundary, not browser identity enforcement.

Apply configuration changes with an App restart. The setting applies to new
invitations; existing grants retain their recorded resource and revocation mode.
It does not migrate existing links. Creating or saving a page allocates no
LayerV resource in either mode. A resource is allocated when creating an
invitation: one per guest in `guest` mode, or the first shared resource for that
page in `page` mode. Later page-mode invitations reuse that page resource.

Switching from `guest` to `page` preserves existing guest resources. The first
new page-mode invitation needs an additional shared resource unless a pool
already exists. Switching from `page` to `guest` preserves existing shared
invitations; only new invitations receive their own resources. To change an
existing invitation's isolation, revoke it and create a replacement.

Shared page resources remain allocated for reuse after their last invitation
is revoked or expires, including after switching to `guest`. They are retired
when the page is deleted. Individual guest resources are retired on guest
revocation or expiry. Upstream capacity is released only once cleanup succeeds.
Switching modes does not automatically consolidate, split, or evict resources.

Resource and qURL limits are separate LayerV limits. Count existing isolated
guest resources, retained page resources, legacy resources, and resources used
by other services against the account's resource limit. Each guest invitation
still needs a qURL in either mode. For example, a 10-resource limit with eight
existing guest resources leaves room for only two additional page pools;
creating other empty pages consumes no slots.

LayerV enforces these limits during creation. If allocation or qURL creation
fails, the Gateway reports an upstream error and does not save or return a new
guest invitation. Existing guests remain unchanged. An exclusively allocated
guest resource is retired if its qURL creation fails; failed cleanup is retried
durably. A shared page resource is retained, and uncertain invitation creation
is reconciled without deleting other page guests. Free capacity, allow pending
cleanup to finish, and retry creating the invitation. There is no automatic
guest eviction or conversion to another isolation mode.

New invitations use the shared Connector runtime. Existing legacy invitations
retain their original Connector and identifiers until revoked or expired.

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

Camera entities can be assigned as read-only resources. Each camera has its own
**Still-image refresh** choice (15 seconds, 30 seconds, 1 minute, 2 minutes,
5 minutes, or manual only) in the resource editor; this option is not shown for
other entity types. Existing cameras and new cameras default to 30 seconds.
Manual-only cameras load one initial still and provide a **Refresh image**
button, which is also available for automatically refreshed cameras. The
gateway enforces the saved interval independently for each camera and guest or
preview session. Images are proxied through the page authorization boundary
with `no-store` caching and are never saved to the App data directory. Only
still images are exposed; live video and audio are not available.

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

The reviewer-facing package is
[ARCHITECTURE.md](../ARCHITECTURE.md), [SECURITY.md](../SECURITY.md), and
[REVIEW_GUIDE.md](../REVIEW_GUIDE.md).

- Guest access is separate from App administration.
- Home Assistant Ingress is accepted only from the Supervisor Ingress proxy;
  the gateway itself listens only inside the App container.
- Every guest page endpoint, the Ingress admin gateway, and the Home Assistant
  broker run under distinct Linux identities. Each lightweight compiled guest
  endpoint receives only a capability valid for its page; page data and alert
  recipients remain in the trusted gateway.
  The shared Connector flow prepares one loopback endpoint per saved page;
  legacy-only operation prepares endpoints while guests remain active.
  The public process has no admin token, Supervisor token, LayerV API key,
  LayerV lifecycle credential, discovery credential, or policy-write
  credential.
- Home Assistant actions and LayerV lifecycle operations pass through narrow
  brokers that independently resolve saved policy. Browser requests cannot
  supply an arbitrary HA service/entity tuple or raw LayerV resource/qURL ID.
- Brokers read a sanitized authoritative policy store. Guest grants, token
  hashes, qURL links, and activity history are not published into it.
  The trusted LayerV broker also has read-only access to saved Gateway grants
  to confirm invitation creation committed before retaining an allocation.
  Guest endpoints have no access to that store or the native device state.
- Guest access tokens are shown once and persisted only as SHA-256 hashes.
- Page definitions, token hashes, qURL revocation identifiers, connector
  identity, and required secrets persist under `/data`.
- Guest activity is stored in an admin-only SQLite database under
  `/data/guest-runtime`. Page endpoints submit constrained, page-authenticated
  activity events through the trusted internal broker and cannot read the
  database.
  Protect Home Assistant backups accordingly. Revoked-guest records are
  automatically purged after 30 days, and each guest is limited to the 1,000
  most recent actions.
- Security events are retained for 30 days and capped at the 1,000 most recent
  events per page. The Web UI displays the latest 100.
- Version 0.1.38 enforces an explicit AppArmor allowlist for the packaged
  runtime, Gateway data, ordinary TCP/UDP networking, and process supervision.
  Access outside this policy is denied and recorded by Home Assistant.
- Version 0.1.42 adds separate process UIDs and Unix file permissions inside
  that AppArmor boundary. All processes still share the App container's
  loopback network and one AppArmor profile; this is defense in depth, not the
  same boundary as separate containers or coordinated Apps.
  The root supervisor retains only the identity and file-ownership
  capabilities needed to prepare persistent directories and drop child UIDs.
  Child processes lose those effective capabilities when their UID is
  changed.
- Published images include an SBOM, build provenance, and a keyless Cosign
  signature tied to the release workflow.
- App backups contain LayerV credentials and connector private state. Protect
  them as secrets.

### AppArmor enforcement and rollback

The restricted profile is enforced. After an App update, verify startup, page
editing, preview, one temporary guest action, revocation, activity history, and
one App restart. Another LayerV connection reset is not required.

The activity database allowlist includes SQLite's rollback-journal, WAL, and
shared-memory sidecars. The allowlist also covers the temporary and final files
used to schedule a connection reset atomically. These files are transient
implementation
details under `/data`; they contain activity/security history and must remain
covered by the same backup and credential-handling precautions.

If the App fails only after this enforcement update, review the Home Assistant
host audit journal for `apparmor="DENIED"` events associated with
`layerv_ha_gateway`. Share only sanitized operation names and paths; never
include API keys, tokens, qURLs, access links, request bodies, or Connector
private state. Restore the LayerV App-only backup or reinstall the preceding
version without deleting App data while the missing permission is investigated.

The shared runtime enrolls a device on first publication and stores native
state privately in `/data/layerv-broker`. New invitations allocate resources
according to `resource_isolation`; normal restarts reuse their identities and
routes. The AppArmor profile permits the pinned `qurl` executable and the
broker-owned Unix IPC socket in addition to the retained legacy Connector.

Before upgrading from 0.1.101, take an App-only backup. Existing legacy links
keep their original identifiers and behavior; revoke and replace them to adopt
the new session flow. A rollback to 0.1.101 requires restoring that pre-upgrade
backup and the preceding image, rather than interpreting modern invitation
records with the older application. Revoke new invitations before rolling back;
restoring a backup cannot restore LayerV resources already deleted upstream.

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
  the individual App page after **Check for updates**. A Supervisor reload may
  be unavailable on an older Supervisor; do not delete the App or repository
  merely to refresh update metadata.
- If reset does not proceed after typing `RESET`, use the error shown inside
  the confirmation dialog. An AppArmor denial for
  `.reset-connection.request.tmp` indicates an outdated profile.
- If guest activity cannot load or a revoked guest record cannot be deleted,
  check for an AppArmor denial involving
  `guest-activity.sqlite3-journal`, `-wal`, or `-shm`.
- A `401` on an administrator preview normally means the preview was opened
  outside its active Home Assistant Ingress session. Open Preview from the
  Gateway UI in the same Home Assistant session.
- “This access link has expired or been revoked” is the expected guest-facing
  result after expiration, revocation, reset, or token alteration.
- Never paste LayerV API keys, activation qURLs, access links, or preview tokens
  into public support messages.
- Report suspected vulnerabilities privately using the process in
  `SECURITY.md`; never put live credentials in a public issue.

For diagnosis, record only the App version, operation, timestamp, HTTP status,
and sanitized AppArmor operation/path. Page files, the activity database,
backups, Connector state, and App logs may contain sensitive operational data.

## License and branding

The gateway source code is licensed under the MIT License. LayerV names,
wordmarks, logos, and other brand assets are not included in that license.
See `../BRAND_ASSETS.md` in the repository.

### Guest lifetime and admission renewal

The qURL lifetime and the LayerV admission duration are separate clocks. The
Gateway can issue a three-day grant on a plan permitting three-day qURLs. Each
LayerV admission is limited to at most 24 hours, or the requested grant lifetime
when shorter. Production accepted a three-day qURL with 24-hour admission and
rejected a 25-hour admission duration.

Renewable invitations explicitly use `one_time_use: false`. The guest retains
one qURL and reopens it in the original browser when upstream admission expires.
The Gateway bootstrap is consumed only once. Its Secure, HttpOnly cookie lasts
until the grant deadline; a renewed admission in that bound browser redirects
to the clean guest prefix without exchanging the bootstrap again. A different
browser cannot exchange an already-consumed bootstrap. Clearing the cookie
requires the owner to revoke and issue a new invitation. Required email-code
verification remains a separate authorization step with a maximum 12-hour
session, so a valid guest cookie does not bypass re-verification.

Single-use invitations remain available for grants of at most 24 hours. They
use LayerV's native `one_time_use: true` setting, rather than relying only on
the Gateway bootstrap. Production testing admitted the first browser, denied
a fresh browser's second qURL redemption, and preserved the original browser's
protected Gateway session within its admission lifetime. Same-NAT direct access
still requires the appropriate Gateway cookie.
Single-use invitations
cannot renew admission with their consumed qURL. Revocation and the current grant
expiry are checked on every protected Gateway operation in either flow.

The guest identity cookie uses `SameSite=Lax` so it accompanies the safe
navigation from `qurl.link` back to the resource host when renewing admission.
It remains Secure, HttpOnly and scoped to the exact guest prefix. Mutating guest
requests require `X-Guest-Request: 1`, which cross-site forms cannot supply; guest
CORS access is not granted. Email-verification cookies retain their separate
policy and expiry.

Minting failures preserve safe broker error messages and Connector operation/
exit-code diagnostics. No raw Connector stderr, credentials, or upstream
response detail is exposed. A failure reports no successful new guest link;
existing guests remain unchanged. Check permissions, quota, or runtime readiness
according to the reported error; do not reset existing guest credentials as a
generic troubleshooting step.

For an unclassified Connector login failure (exit 1), the minting error also
includes the Connector error line with credentials, URLs, email addresses, and
long opaque values redacted. This diagnostic identifies enrollment failures
without requiring access to Docker. It does not change enrollment behavior.
