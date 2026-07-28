# Changelog

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
