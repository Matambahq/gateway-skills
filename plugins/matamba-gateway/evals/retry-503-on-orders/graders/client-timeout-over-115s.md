---
type: regex
target:
  source: file
  path: order_client.py
pattern: 'timeout[^\n]{0,60}?\b(11[6-9]|1[2-9][0-9]|[2-9][0-9]{2})(\.[0-9]+)?\b'
flags: i
---
