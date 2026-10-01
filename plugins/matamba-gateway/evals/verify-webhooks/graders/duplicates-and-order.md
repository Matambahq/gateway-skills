---
type: llm
focus:
  source: file
  path: webhook.js
---

PASS if the handler ignores an event it has already processed (by `webhook-id` or the event's `id`) and ignores an event for an order whose `version` is lower than the last one it applied, and answers with a 2xx status quickly.
FAIL if a repeated or out-of-order event would be applied again, or if the handler does slow work before answering.
