# Changelog

## 0.1.58

- Send verified guests an invitation email containing the LayerV activation
  qURL and their one-time access link when the guest record is created.
- Show invitation delivery status first and place manual QR, copy, and share
  actions under backup sharing options after successful email delivery.
- Send guest verification codes through a direct authenticated loopback broker
  connection that does not depend on HTTP proxy discovery.

## 0.1.57

- Add locally stored, TLS-only SMTP configuration and a test-email action.
- Add optional per-guest email verification with six-digit, single-use codes,
  ten-minute expiry, resend throttling, and a five-attempt limit.
- Require a short-lived HttpOnly, Secure, SameSite session for every guest API
  after verification and invalidate that session when access is revoked.
- Keep SMTP credentials in the isolated administrator runtime; the public guest
  process can request only an email to the address saved with its active grant.

## 0.1.56

- Prefer the native share sheet on capable iPhone, iPad, Android, and desktop
  browsers instead of displaying unreliable email and SMS WebView links.
- Keep email and SMS compose controls as fallbacks when the browser does not
  provide native sharing; QR and copy controls remain available everywhere.

## 0.1.55

- Open email and Messages through real top-level links so iOS can hand their
  URL schemes out of the Home Assistant Ingress frame.
- Follow Apple's supported SMS URL format and copy the complete two-link
  message for the administrator to paste into Messages.

## 0.1.54

- Serve the locally bundled QR encoder from its nested static directory while
  preserving the existing resolved-path boundary against directory traversal.

## 0.1.53

- Generate activation and guest-access QR codes entirely in the administrator's
  browser without sending either bearer link to an external QR service.
- Open native share, email, and text-message compose screens with both links
  clearly numbered for the guest.
- Keep optional email addresses and mobile numbers ephemeral; the Gateway does
  not save, log, or transmit recipient details.

## 0.1.52

- Preserve proximity and other action-denial messages on the guest page
  instead of incorrectly replacing HTTP 403 responses with the expired or
  revoked-link screen.

## 0.1.51

- Add an optional page-level proximity safety check that keeps live status
  visible remotely while requiring guests to be near Home before actions.
- Explain location use before the browser permission prompt and reuse a recent
  successful reading for up to five minutes.
- Validate location freshness and accuracy in the Gateway, and calculate the
  distance inside the HA policy broker without exposing Home's coordinates to
  the guest browser or public Gateway process.

## 0.1.50

- Remove the ineffective optional public port mapping; the qURL Connector
  reaches the guest Gateway through container loopback and Home Assistant
  Ingress remains the administrator entry point.
- Persist Connector audit logs in a Connector-owned directory under App data
  instead of silently falling back to no logging when `/var/log` is not
  writable.
- Add real-socket regression coverage proving that internal brokers reject
  missing or incorrect credentials before invoking privileged backends.

## 0.1.49

- Run first-time LayerV credential onboarding under a dedicated unprivileged
  identity and pass the credential to the trusted App supervisor through a
  one-time inherited pipe.
- Run initial qURL Connector registration under the dedicated Connector
  identity and reject duplicate onboarding submissions.
- Handle shutdown signals during onboarding so the App stops cleanly without
  waiting for container termination.
- Immediately remove stale entity cards and values when a guest link expires
  or is revoked.
- Keep every qURL lifetime control anchored to its own visible label on touch
  devices and narrow browser layouts.

## 0.1.48

- Allow the root App runner to stop the fixed Ingress child after it has
  dropped to its dedicated unprivileged UID.
- Prevent fresh-install onboarding from failing with `PermissionError` while
  cleaning up its owned processes.

## 0.1.47

- Keep the guest activity database group-readable and group-writable for the
  isolated admin and guest processes.
- Prevent guest-page access from locking the admin process out of activity
  summaries and page management.

## 0.1.46

- Assign cooperating processes their required shared storage group as the
  primary GID instead of relying on supplementary container groups.
- Restore deterministic guest reads of existing page definitions and broker
  reads of authoritative policy.
- Continue restricting page and activity data to the admin and guest process
  identities.

## 0.1.45

- Publish the complete process-isolation AppArmor profile with the public
  Home Assistant App Store metadata.
- Permit creation and use of the isolated policy, activity, admin-runtime,
  and LayerV broker stores during upgrade.
- Permit the root supervisor to assign the fixed child-process identities
  before dropping their privileges.

## 0.1.44

- Permit the root App supervisor to create the explicitly allowlisted
  top-level isolation stores when upgrading an existing Home Assistant
  `/data` volume.
- Include the private LayerV broker store in fresh image initialization.
- Restore the 0.1.42 policy migration without broadening access to arbitrary
  `/data` descendants.

## 0.1.43

- Add `/app` to the packaged Python module path so the isolation supervisor
  can import the Gateway modules regardless of the working directory selected
  by Home Assistant Supervisor.
- Restore App startup after the 0.1.42 process-isolation update.

## 0.1.42

- Split the public guest gateway from the Home Assistant Ingress admin
  gateway.
- Move the Supervisor token and LayerV API key into narrow HA and LayerV
  policy brokers.
- Add a broker-owned authoritative policy store containing no guest grants,
  token hashes, qURL links, or activity history.
- Run admin, guest, broker, policy, Connector, and Ingress processes under
  separate Linux users with restrictive file ownership.
- Give the public guest process only the HA action-broker credential; it
  receives no admin, HA, LayerV, discovery, or policy-publication credential.
- Migrate existing pages, active qURL revocation mappings, and guest activity
  into their isolated stores without deleting configured pages.

## 0.1.41

- Close slow or incomplete client requests after a 15-second connection
  deadline.
- Reject more than 64 request headers or more than 32 KiB of aggregate header
  data with HTTP 431.
- Cap active request threads and the queued connection backlog at 64 each.
- Quietly rate-limit repeated unauthorized page reads without adding noisy
  security-history rows; normal authenticated mobile polling remains unlimited.
- Add real-socket adversarial coverage for slow clients, header abuse,
  connection saturation, invalid-read floods, and concurrent valid polling.

## 0.1.40

- Keep primary values and units visible for read-only sensors on narrow mobile
  screens.
- Give read-only entity state a full-width stacked mobile row so long values
  wrap instead of being hidden or truncated.
- Add regression coverage for the read-only mobile-state treatment.

## 0.1.39

- Add real threaded-HTTP adversarial tests for cross-page token isolation,
  forged targets, malformed bodies, replay, concurrency, and revocation races.
- Linearize guest actions with individual revocation, revoke-all, page
  deletion, and LayerV connection reset so no new action can succeed after
  local revocation returns.
- Reject non-object JSON bodies and apply Gateway security headers to
  unsupported-method and parser errors.
- Stop returning Home Assistant service response bodies to guest browsers.
- Increase the bounded HTTP accept queue and use daemon request threads for
  predictable shutdown under concurrent traffic.

## 0.1.38

- Permit the atomic `.reset-connection.request.tmp` file under the enforced
  AppArmor profile so confirmed LayerV connection resets can be scheduled.
- Show reset progress and API failures inside the open confirmation dialog
  instead of only in the obscured page-level status area.
- Add regression coverage for both the temporary and final reset request paths.

## 0.1.37

- Permit SQLite's `guest-activity.sqlite3-journal` rollback sidecar under the
  enforced AppArmor profile.
- Restore guest-activity loading, revoked-guest deletion, retention cleanup,
  and other activity-store transactions that need SQLite's default rollback
  journal.
- Add regression coverage for both rollback-journal and WAL sidecar paths.

## 0.1.36

- Promote the explicit AppArmor filesystem, network, and signal allowlist from
  complain mode to enforcement after Home Assistant startup, update, reset,
  onboarding, Connector bootstrap, page, guest, action, history, and restart
  acceptance testing produced no unexplained AppArmor events.
- Keep the former blanket `file`, `network`, and `capability` grants removed.
- Update regression coverage and operator documentation for enforced denial
  behavior and rollback.

## 0.1.35

- Supply the protected LayerV API-key file to the qURL Connector when its
  durable agent state is absent or incomplete, as required for the first
  bootstrap after onboarding or **Reset LayerV connection**.
- Stop supplying the bootstrap key-file path on later App starts once the
  Connector's agent identity, keys, tunnel identities, and configuration are
  complete.
- Add regression coverage for fresh, partial, and complete Connector state.

## 0.1.34

- Replace the blanket AppArmor `file`, `network`, and `capability` grants with
  an explicit first-pass allowlist for the packaged runtime, App data,
  ordinary TCP/UDP networking, and process supervision.
- Run the candidate profile in AppArmor complain mode so Home Assistant records
  missing permissions without blocking existing pages, guests, or Connector
  state during the acceptance-test phase.
- Document how to collect sanitized AppArmor audit evidence and promote the
  profile to enforcement only after complete onboarding, reset, guest-action,
  expiration, restart, and failure-path testing.
- Add regression coverage preventing blanket AppArmor permissions from being
  restored accidentally.

## 0.1.33

- Stop classifying read-only guest-page loads and automatic status polls as
  invalid-token security incidents.
- Record an invalid-token security event only when a client attempts a Home
  Assistant action with an unrecognized token.
- Remove the false-positive invalid-token rows generated by version 0.1.32.
- Add regression coverage proving reload and polling requests cannot create
  invalid-token history.

## 0.1.32

- Automatically remove expired grants from active page data on the next
  Gateway refresh or expired-link attempt and clean up their LayerV qURLs.
- Retain expired guests and their activity for the normal 30-day review period
  before automatic deletion.
- Add per-page and per-guest security history for invalid tokens, expired-link
  attempts, rate limiting, unapproved actions, and Home Assistant rejections.
- Exclude tokens, request bodies, headers, IP addresses, and arbitrary fields
  from security history, with 30-day and 1,000-event storage limits.

## 0.1.31

- Route revoked-guest deletion through the correct HTTP `DELETE` handler.
- Rename the action to **Delete guest record** and remove both the retained
  revoked-guest entry and all of its activity history.

## 0.1.30

- Add per-guest activity history for successful and failed Home Assistant
  actions, including time, entity, approved action, and safe action parameters.
- Keep the Gateway authoritative for activity attribution and exclude preview
  actions, credentials, access links, headers, and unrestricted request data.
- Show activity from each active guest and retain revoked-guest history for 30
  days, with an immediate **Delete history** option.
- Store activity in an owner-only SQLite database under persistent App data and
  automatically purge expired revoked-guest records.

## 0.1.29

- Treat LayerV `404 Not Found` and `410 Gone` deletion responses as successful,
  idempotent qURL revocation while preserving genuine remote failures.
- Add a read-only Gateway health panel showing the running version, Gateway
  state, LayerV API configuration, page count, and active guest-link count.
- Remove guest labels from qURL creation and revocation audit events while
  retaining non-secret page, grant, and qURL identifiers.

## 0.1.28

- Show a clear expired-or-revoked message when an open guest page loses
  access instead of exposing a JSON parsing error.
- Stop status polling and disable guest controls after access ends.
- Safely handle both JSON and plain-text error responses from the Gateway or
  LayerV edge while continuing to retry temporary failures.
- Rename user-facing “user” terminology to “guest” to reflect that a qURL is
  an access grant rather than a Home Assistant user account.

## 0.1.27

- Stop providing the LayerV API key to the Connector after its initial
  registration; normal starts now use only persistent Connector identity.
- Give the Gateway, Connector, Ingress proxy, and onboarding server explicit
  minimal environments so unrelated Home Assistant and App secrets are not
  inherited by sibling processes.
- Reject page definitions containing entities or actions that do not exactly
  match the Gateway's server-side Home Assistant discovery allowlist.
- Add regression coverage for Supervisor-token and LayerV-key separation.
- Add adversarial tests for cross-page resources, unapproved actions, forged
  Home Assistant targets, extra service parameters, and token separation.

## 0.1.26

- Remove unused JavaScript bindings, HTML hooks, and admin and guest CSS rules.
- Add regression checks for orphaned DOM references, JavaScript functions, and
  CSS classes.
- Add pinned CI checks for static analysis, secrets, dependencies,
  configuration, and the built Home Assistant App image.
- Add weekly Dependabot checks for GitHub Actions and Docker base images.

## 0.1.25

- Remove the unused legacy `/door` interface and pre-deployment plaintext-token
  migration path.
- Add a Home Assistant configuration ceiling for qURL lifetimes, defaulting to
  the LayerV Free-plan limit of three days.
- Support custom whole-number lifetimes in minutes, hours, or days and hide
  presets above the configured plan limit.

## 0.1.24

- Add a preview-only toolbar with a **Back to Gateway** action.
- Keep the toolbar out of issued guest access pages and provide an
  Ingress-root fallback when browser history is unavailable.

## 0.1.23

- Poll the Ingress upstream during reset and reload only after onboarding is
  actually ready, eliminating the remaining transient 404 race.
- After API-key submission, wait for the registered Gateway to become healthy
  before reloading back into its administration page.

## 0.1.22

- Open previews in the current Home Assistant App view so mobile Companion App
  WebViews retain their authenticated Ingress session.
- Use the Home Assistant Back action to return from a preview to the Gateway.

## 0.1.21

- Keep Home Assistant Ingress available while a LayerV connection reset
  transitions from the Gateway to onboarding.
- Preserve the internal Ingress authentication token across the transition so
  the existing App page reloads directly into **Connect to LayerV**.

## 0.1.20

- Add a confirmed LayerV connection reset that revokes every user locally
  before attempting remote qURL cleanup.
- Preserve access-page definitions while removing the old connector
  credential, identity, configuration, and state.
- Return directly to authenticated onboarding so a new connector can be
  registered and new user links issued.
- Remove the API-key recovery field from Home Assistant App configuration.

## 0.1.19

- Add a first-run **Connect to LayerV** screen inside authenticated Home
  Assistant Ingress.
- Write onboarding credentials directly to the owner-only secret file without
  placing them in Home Assistant App options.
- Retain the Configuration API-key field only as a recovery override.
- Shorten the Home Assistant sidebar title to **LayerV Gateway**.

## 0.1.18

- Replace the initial path-level AppArmor rules with a compatibility profile so
  Python can import packaged modules and the connector can initialize its
  private container audit log.

## 0.1.17

- Allow Python's shared runtime library under `/usr/local/lib` in the AppArmor
  profile so the App can start while remaining confined.

## 0.1.16

- Stop persisting plaintext guest bearer tokens and scrub legacy page records
  automatically at startup.
- Accept administrative Ingress traffic only from Home Assistant's trusted
  proxy and keep the gateway listener on the container loopback interface.
- Withhold upstream API response bodies and connector registration output that
  could contain authentication material.
- Add a custom AppArmor profile, a private vulnerability-reporting policy, and
  keyless Cosign signing for published container images.
- Document how to remove the duplicate LayerV API key from App options after
  its protected key file has been initialized.

## 0.1.15

- Link the App, gateway dashboard, guest footer, and documentation to
  LayerV.ai.
- Use a layered monochrome sidebar icon that echoes the LayerV mark.
- License the gateway code under MIT while reserving LayerV brand assets.

## 0.1.14

- Add LayerV App Store logo and icon assets.
- Explain LayerV, guest access, security, and every configuration option.
- Add branded context to the admin start page.
- Add sticky Users and Save page actions at the bottom of the editor.

## 0.1.13

- Stack range labels, sliders, and actions into touch-friendly mobile rows.
- Keep entity names and state text from squeezing or overflowing narrow cards.
- Adapt action, choice, select, climate, and parameter controls down to
  small-phone widths.

## 0.1.12

- Copy generated links from inside the active user dialog instead of an inert
  page element.

## 0.1.11

- Copy generated links synchronously while browser click permission is active.
- Make newly generated URLs directly selectable if browser policy blocks copying.

## 0.1.10

- Load guest preview assets and APIs through the Home Assistant Ingress path.
- Fall back when the Ingress Clipboard API exists but rejects writes.
- Display newly generated activation and access links as readable rows.

## 0.1.9

- Open preview pages inside the current Ingress session.
- Copy links on non-secure LAN origins where the Clipboard API is unavailable.
- Present generated activation and access links in a readable grid.

## 0.1.8

- Build browser asset and API URLs from Home Assistant's validated
  `X-Ingress-Path` instead of the outer App page URL.

## 0.1.7

- Save existing pages through an Ingress-compatible authenticated POST
  endpoint while retaining PUT compatibility.
- Report non-JSON proxy responses with their HTTP status and content type.

## 0.1.6

- Let private Home Assistant Ingress supply admin authentication without
  requiring an admin token in the browser URL.
- Preserve explicit token authentication for direct `/admin` access.

## 0.1.5

- Keep admin styles, scripts, and API requests under the Home Assistant
  Ingress URL prefix.

## 0.1.4

- Open the admin shell when Home Assistant requests the App Ingress root.
- Permit same-origin Home Assistant framing only on the private Ingress
  response while retaining the public gateway's frame prohibition.

## 0.1.3

- Keep the configured LayerV key file available to the embedded connector
  during startup so placement-cache files cannot be mistaken for a completed
  agent identity.
- Continue storing the key only in protected App storage and pass its file path,
  not its value, to the connector process.

## 0.1.2

- Store connector identity directly in `/data/connector-state` instead of
  presenting that directory through a symlink rejected by the connector.
- Pass the persistent state directory to connector registration and runtime.
- Preserve sanitized connector failure details without exposing API keys.

## 0.1.1

- Replace the unsupported `/v1/connectors` onboarding request with the LayerV
  Connector's supported tunnel registration flow.
- Reuse the saved route and agent identity on restart.
- Keep connector registration failures free of API response bodies and tokens.

## 0.1.0

- Add the initial Home Assistant App package.
- Use Supervisor-provided Home Assistant API authentication.
- Register and persist a LayerV qURL Connector during first-run setup.
- Separate Home Assistant Ingress administration from public qURL traffic.
