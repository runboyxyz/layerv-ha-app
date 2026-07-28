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

First-run setup happens inside authenticated Home Assistant Ingress. The
LayerV credential is written directly to an owner-only secret file instead of
Home Assistant App options.

Learn more at [LayerV.ai](https://layerv.ai).

## Security

Guest links are independently expiring and revocable. The gateway stores guest
bearer tokens only as hashes, enforces every permitted entity and action on the
server, accepts administration through Home Assistant Ingress, and runs under
a custom AppArmor profile. Published images include provenance, an SBOM, and a
keyless Cosign signature.

See `DOCS.md` for credential handling and `SECURITY.md` for private
vulnerability reporting.

## License

The gateway source code is available under the MIT License. LayerV names,
wordmarks, logos, and other brand assets are excluded from that license; see
`BRAND_ASSETS.md`.
