# LayerV Home Assistant Gateway

This App runs the gateway and LayerV qURL Connector together. Home Assistant
provides API access automatically; do not create a long-lived Home Assistant
token.

## Installation

1. Add this repository to the Home Assistant App Store.
2. Install **LayerV Home Assistant Gateway**.
3. Enter a LayerV API key under **Configuration**. The key needs connector and
   qURL read/write permissions.
4. Optionally choose a stable connector ID and entity include/exclude policies.
5. Save, start the App, and open its Web UI.

On first start, the App uses the bundled LayerV Connector to find or create its
tunnel resource and write the LayerV-issued route identifiers under `/data`.
The connector then registers its identity and stores durable agent state there.
Normal restarts reuse both the route and agent state instead of creating another
connector.

The App passes `/data/connector-state` directly to the connector as its agent
state directory. Do not replace it with a symlink or share it with another
connector instance.

The configured LayerV key remains in protected App storage because the gateway
uses it to manage qURLs. The connector receives the protected key-file path on
startup; it reuses completed agent state when available and uses the key only
when bootstrap or recovery is required.

Do not delete App data or change the connector ID after registration. App
backups contain LayerV credentials and connector private state and must be
protected accordingly.
