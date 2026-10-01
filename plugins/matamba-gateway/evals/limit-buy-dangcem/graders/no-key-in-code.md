---
type: regex
target:
  source: file
  path: place_order.ts
pattern: 'mgw_(test|live)_[A-Za-z0-9_-]{8,}'
match: not_contains
---
