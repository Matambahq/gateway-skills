---
type: regex
target:
  source: file
  path: fund.py
pattern: 'mgw_(test|live)_[A-Za-z0-9_-]{8,}'
match: not_contains
---
