# Changelog

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
