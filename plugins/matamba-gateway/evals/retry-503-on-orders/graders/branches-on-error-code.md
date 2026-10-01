---
type: llm
focus:
  source: file
  path: order_client.py
---

PASS if the code reads the error's `code` field from the JSON error body and treats `outcome_not_recorded` as "don't send again, retrieve the order it names", and treats `order_not_sent` as the one 503 that may be sent again with a new key.
FAIL if it handles every 503 the same way, or decides only from the HTTP status or the `message` text.
