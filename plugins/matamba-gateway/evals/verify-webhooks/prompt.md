---
description: A webhook handler must verify the Standard Webhooks signature over the raw body with the decoded whsec_ secret, answer fast, and ignore duplicates and stale events.
tags: [webhooks]
max_turns: 25
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

Add an Express route that receives Matamba Gateway webhook events and marks our order record as filled when the gateway tells us an order has filled. Assume a markOrderFilled(orderId, filledQuantity) function already exists in ./orders.js. Save the route as webhook.js in the current directory.
