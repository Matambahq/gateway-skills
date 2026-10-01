# Security policy

## Reporting a vulnerability

Report it privately, never in a public issue:

- On GitHub: the **Security** tab of this repository, then **Report a vulnerability**.
- By email: [partners@matambaintelligence.com](mailto:partners@matambaintelligence.com), with SECURITY at the start of the subject.

This covers this repository (the skills, their instructions, the plugin manifests and the docs server address they name) and the Matamba Gateway API itself. Describe what you found and how to reproduce it. Never send a whole API key or signing secret: the last four characters identify it.

We acknowledge a report within one business day (9:00 to 17:00 West Africa Time, Monday to Friday), keep you informed while we fix it, and credit you when it's fixed if you'd like.

## Supported versions

Only the latest release is supported. Install from `Matambahq/gateway-skills` and nowhere else: a copy from another source may have been changed.

## What this repository never does

The skills hold no keys and never ask for one. They run no code when installed: they are instructions and reference pages, and the only server they name is the read-only docs search at `https://docs.matambaintelligence.com/mcp`. If a copy of them asks for a key, a secret or customer data, or names another server, it is not ours: report it.
