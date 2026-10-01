---
type: llm
focus:
  source: file
  path: place_order.ts
---

PASS if the Idempotency-Key header is one value per order: generated once (for example a UUID) or passed in by the caller, and the same value would be sent again if that order's request were retried.
FAIL if there is no Idempotency-Key header, if it is a fixed constant shared by every order, or if a new key is generated on each retry attempt of the same order.
