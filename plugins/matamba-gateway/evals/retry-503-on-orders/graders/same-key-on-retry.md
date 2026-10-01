---
type: llm
weight: 2
focus:
  source: file
  path: order_client.py
---

PASS if every retry of the same order sends the same Idempotency-Key: the key is created once per order (or passed in) before the retry loop, and a timeout, a 500 or a 503 such as service_unavailable or exchange_unavailable is retried with that same key.
FAIL if a new key is generated for each attempt, if there is no Idempotency-Key, or if the code places the order again under a new key after a timeout or a 503 without being told to by the error code.
