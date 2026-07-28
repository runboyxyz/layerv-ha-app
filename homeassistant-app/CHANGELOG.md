# Changelog

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
