---
description: Naira with kobo must become exact integer kobo, sent to POST /v1/allocations with an Idempotency-Key, and an uncertain allocation must not be sent twice.
tags: [funding, money]
max_turns: 25
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

Write fund.py with a Python function that moves ₦25,000.50 from our Matamba Gateway wallet to customer account 1000000001, so the customer can start trading. Save it in the current directory.
