# LayerV Home Assistant Gateway

[LayerV](https://layerv.ai) lets you share selected Home Assistant controls
without port forwarding, VPN accounts, or full Home Assistant user access.

Create a purpose-limited page, select exactly which entities and actions it may
use, then issue an expiring LayerV qURL to each guest. Every guest link can be
revoked independently.

The App runs the LayerV gateway and qURL Connector together. Home Assistant
supplies local API access automatically, while LayerV provides the protected
external route. No inbound router port or long-lived Home Assistant token is
required.

Learn more at [LayerV.ai](https://layerv.ai).

## License

The gateway source code is available under the MIT License. LayerV names,
wordmarks, logos, and other brand assets are excluded from that license; see
`BRAND_ASSETS.md`.
