---
description: A 503 or a timeout on POST /v1/orders must be retried with the same Idempotency-Key, branching on error.code, never by placing a second order.
tags: [orders, errors, idempotency]
max_turns: 25
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

Our Python service places orders with POST /v1/orders on the Matamba Gateway. Sometimes the call times out or comes back 503, and we're worried about buying twice or dropping orders. Write order_client.py with a place_order function that handles this properly. Save it in the current directory.
