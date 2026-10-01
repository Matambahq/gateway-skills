---
type: llm
weight: 2
focus:
  source: file
  path: webhook.js
---

PASS if the code verifies the signature as Standard Webhooks does: HMAC-SHA256 over `webhook-id`, `webhook-timestamp` and the raw body joined with "." in that order, keyed with the base64-decoded part of the secret after "whsec_", compared against each space-separated "v1,<signature>" entry of the `webhook-signature` header, and rejecting a timestamp more than 5 minutes from the current time.
FAIL if any of these is missing or different, such as signing the re-serialised JSON, using the secret text as the key, or checking only the first signature.
