# Changelog

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
