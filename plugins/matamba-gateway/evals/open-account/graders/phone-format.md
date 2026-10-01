---
type: llm
focus:
  source: file
  path: open_account.py
---

PASS if the code sends `phone` and `next_of_kin_phone` as a Nigerian number written as "0" followed by 10 digits (or "+234" followed by the same 10 digits), normalising or validating the customer's number into that form before sending.
FAIL if it sends the numbers unchecked in whatever form they are stored, or in any other format such as "234..." without "+", or with spaces or dashes.
