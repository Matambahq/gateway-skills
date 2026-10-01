---
description: An application needs the reference codes, mothers_maiden_name, Nigerian phone numbers and all four documents sent as raw JPEG or PNG under 1,000,000 bytes.
tags: [applications]
max_turns: 30
timeout_seconds: 900
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

Write a Python module for our backend that opens a Matamba Gateway brokerage account for one of our customers: submit their application and upload their documents. We already hold the customer's details and scanned documents in our own database. Save it as open_account.py in the current directory.
