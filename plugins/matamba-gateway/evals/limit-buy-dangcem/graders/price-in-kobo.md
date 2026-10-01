---
type: llm
weight: 2
focus:
  source: file
  path: place_order.ts
---

PASS if the order's `price_kobo` is the integer 57700 (₦577 in kobo): written as that integer, built with integer arithmetic, or converted exactly from a decimal or a string without binary floating point.
FAIL if the price is sent as 577 or any other naira figure, as a float or a string, or if it comes from multiplying a binary float that could misround.
