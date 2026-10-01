---
type: llm
weight: 2
focus:
  source: file
  path: fund.py
---

PASS if the request's `amount_kobo` is the integer 2500050 and it is computed exactly: written as that integer, built with integer arithmetic, or converted with a decimal type (such as Python's Decimal) and checked to be whole.
FAIL if the amount is sent in naira, as a float or a string, or if it comes from multiplying a binary float such as 25000.50 * 100.
