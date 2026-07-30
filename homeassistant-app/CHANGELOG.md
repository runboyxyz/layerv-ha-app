# Changelog

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
