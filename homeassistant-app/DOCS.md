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

On first start, the App registers the connector through
`POST /v1/connectors`, stores its generated resource ID and connector identity
under `/data`, and removes the temporary bootstrap key after identity state is
present. Normal restarts do not create another connector.

Do not delete App data or change the connector ID after registration. App
backups contain LayerV credentials and connector private state and must be
protected accordingly.

