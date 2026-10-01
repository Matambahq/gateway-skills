---
description: A limit buy in naira must reach the API as integer kobo, with an Idempotency-Key and the key read from the environment.
tags: [orders, money]
max_turns: 25
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

Write a TypeScript function for our Node.js backend that places a limit buy of 10 DANGCEM at ₦577 for customer account 1000000001 through the Matamba Gateway API. Save it as place_order.ts in the current directory.
