---
type: llm
focus:
  source: file
  path: fund.py
---

PASS if the code does not send the allocation again under a new Idempotency-Key when the answer is `uncertain`, a timeout, a 5xx or `503 outcome_not_recorded`: it retries with the same key, or retrieves the allocation and waits for it to resolve.
FAIL if any of those outcomes leads to a second allocation under a new key, or if it treats `uncertain` as a failure to retry.
