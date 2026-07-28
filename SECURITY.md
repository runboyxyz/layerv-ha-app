# Security policy

## Reporting a vulnerability

Do not report suspected vulnerabilities in a public issue or include live API
keys, qURLs, Home Assistant tokens, page data, connector state, or logs in a
public report.

Report vulnerabilities privately to the LayerV security contact published at
[LayerV.ai](https://layerv.ai). Include the affected App version, impact,
reproduction steps using synthetic credentials, and any relevant sanitized
logs. LayerV will acknowledge the report, investigate it, and coordinate
remediation and disclosure with the reporter.

## Security model

- Home Assistant authenticates administrators through Ingress.
- The gateway creates independent, expiring guest grants for each named user.
- Every page, entity, action, and action parameter is enforced server-side.
- Guest bearer tokens are displayed once and stored only as SHA-256 hashes.
- Local revocation takes effect before remote qURL cleanup is attempted.
- The LayerV API key and connector identity are installation-specific secrets.

## Credential response

If a LayerV API key, activation qURL, access link, preview token, Home Assistant
token, or connector state is exposed, revoke or rotate it immediately. Revoke
affected guest grants in the Gateway UI. Do not rely on deleting a message,
browser history entry, screenshot, backup, or log as revocation.
