# LayerV Home Assistant Gateway

Share selected Home Assistant controls without port forwarding, VPN accounts,
or full Home Assistant user access.

Create a purpose-limited page, select exactly which entities and actions it may
use, then issue an expiring LayerV qURL to each guest. Every guest link can be
revoked independently.

The App runs the LayerV gateway and qURL Connector together. Home Assistant
supplies local API access automatically, while LayerV provides the protected
external route. No inbound router port or long-lived Home Assistant token is
required.
