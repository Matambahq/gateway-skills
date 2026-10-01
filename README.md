# Matamba Gateway skills for coding agents

Teach your coding agent to integrate with the [Matamba Gateway](https://docs.matambaintelligence.com) correctly: open brokerage accounts for your customers, fund them, and trade Nigerian equities through one API.

**Version 1.0.0. Matches the Matamba Gateway docs release of 1 October 2026.**

It works in Claude Code, OpenAI Codex (CLI, IDE extension and the ChatGPT desktop app), Cursor, and any other tool that reads [Agent Skills](https://agentskills.io). It is one skill folder, in the open standard format, packaged for each tool.

## What you get

| Part | What it does |
|---|---|
| `matamba-gateway` skill | The integration rules your agent follows whenever it writes or reviews code that calls the gateway: sandbox and live hosts, keys, `Idempotency-Key`, amounts in kobo, orders, account applications and their four documents, funding, uncertain results, webhooks, errors and limits. Each rule links to its docs page, and the full pages ship with it in `references/`. |
| `quickstart` skill | A command that walks you through the [sandbox quickstart](https://docs.matambaintelligence.com/quickstart) in your own programming language: from your sandbox key to a filled order in five requests. |
| `matamba-gateway-docs` MCP server | A read-only search of the current docs at `https://docs.matambaintelligence.com/mcp`, so your agent can look up anything the skill doesn't cover. |

It holds no API keys and never asks for one. The quickstart reads your sandbox key from the `MATAMBA_API_KEY` environment variable, refuses anything but a sandbox key, and never prints it or writes it to a file. Don't paste a key into your agent's chat: if you do, [revoke it](https://docs.matambaintelligence.com/support#if-a-key-leaks).

## Install

### Claude Code

```bash
claude plugin marketplace add Matambahq/gateway-skills
claude plugin install matamba-gateway@matamba
```

Start a new session. The skill loads when your work touches the gateway. Run the quickstart with `/matamba-gateway:quickstart`, adding your language if you like: `/matamba-gateway:quickstart python`.

### OpenAI Codex and the ChatGPT desktop app

```bash
codex plugin marketplace add Matambahq/gateway-skills
```

Then open the plugin browser (`/plugins` in the Codex CLI, or **Plugins** in the ChatGPT desktop app), choose the **Matamba** marketplace, and install **matamba-gateway**. Start a new chat. The skill loads when your work touches the gateway; to call on it or the quickstart directly, type `$` and pick it, or run `/skills`.

To use the skill without the plugin, copy its folder into your repository and add the docs server:

```bash
git clone --depth 1 https://github.com/Matambahq/gateway-skills /tmp/gateway-skills
mkdir -p .agents/skills
cp -R /tmp/gateway-skills/plugins/matamba-gateway/skills/matamba-gateway .agents/skills/
codex mcp add matamba-gateway-docs --url https://docs.matambaintelligence.com/mcp
```

Use `~/.agents/skills/` instead of `.agents/skills/` to have it in every repository.

### Cursor

Copy the skill into your repository, as for Codex above: Cursor reads `.agents/skills/` (and `~/.agents/skills/` or `~/.cursor/skills/` for every project). Then add the docs server to `.cursor/mcp.json`, or `~/.cursor/mcp.json` for every project:

```json
{
  "mcpServers": {
    "matamba-gateway-docs": {
      "url": "https://docs.matambaintelligence.com/mcp"
    }
  }
}
```

To install both skills and the server together as a plugin, copy `plugins/matamba-gateway` to `~/.cursor/plugins/local/matamba-gateway` and run **Developer: Reload Window**.

### Any other Agent Skills tool

Copy `plugins/matamba-gateway/skills/matamba-gateway` into the folder your tool reads skills from, and point its MCP settings at `https://docs.matambaintelligence.com/mcp` (Streamable HTTP, no authentication).

## Use it

Ask for what you need in your own words. For example:

- "Write a function that places a limit buy of 10 DANGCEM at ₦577 for account 1000000001."
- "Open an account for a customer: submit the application and upload their documents."
- "Handle a 503 on `POST /v1/orders`."
- "Verify the Matamba webhook signature in our Express server."

The agent applies the rules: amounts as integer kobo, a saved `Idempotency-Key` reused on every retry, phone numbers as `0` and 10 digits, all four documents under 1,000,000 bytes, your key read from the environment, and so on.

## What's in this repository

```text
.claude-plugin/marketplace.json     Claude Code marketplace
.agents/plugins/marketplace.json    Codex and ChatGPT marketplace
plugins/matamba-gateway/
  .claude-plugin/plugin.json        Claude Code manifest
  .mcp.json                         Claude Code: the docs server
  plugin.json                       Agent Plugins manifest (Codex, ChatGPT, Cursor)
  mcp.json                          Agent Plugins: the docs server
  skills/matamba-gateway/           the skill: SKILL.md, references/, agents/openai.yaml
  skills/quickstart/                the quickstart
  evals/                            eval cases for `claude plugin eval`
scripts/generate.py                 rebuilds references/ from the docs, and checks them
.github/                            the checks GitHub runs on every change, and code owners
SECURITY.md                         how to report a vulnerability
```

## For maintainers

Everything under `references/`, and each `GENERATED` block in the `SKILL.md` files, is generated from the gateway's partner docs (the `docs/partner` directory, with its pages and `openapi.json`). Never edit them by hand.

```bash
python3 scripts/generate.py --docs PATH/TO/docs/partner          # rebuild
python3 scripts/generate.py --docs PATH/TO/docs/partner --check  # verify; writes nothing
```

`--check` exits 1 when the committed files differ from what the docs generate, when the docs add a page the generator doesn't map, when a docs link here points at a page or anchor that doesn't exist, when the manifests disagree, when a `SKILL.md` breaks the Agent Skills rules, or when the leak guard finds a term that must never appear in this repository. Exit 2 means it could not run, which is never a pass. The leak guard's term list is private and kept outside this repository: pass it with `--leak-terms FILE`; without it the check exits 2. Run it before every release, and release the skill together with the docs it was generated from. GitHub runs the checks that need no docs (`--repo-only`) on every change.

Then validate the Claude Code packaging, and run the evals (they call the model with your own credentials):

```bash
claude plugin validate plugins/matamba-gateway --strict
claude plugin validate . --strict
claude plugin eval plugins/matamba-gateway --allow-tools Write Edit --no-publish
```

When you release, raise `version` in both `plugins/matamba-gateway/plugin.json` and `plugins/matamba-gateway/.claude-plugin/plugin.json` (`--check` fails if they differ), and update the version line at the top of this file.

## Security

Report a vulnerability privately: see [SECURITY.md](SECURITY.md). Install only from `Matambahq/gateway-skills`.

## Support

Email [partners@matambaintelligence.com](mailto:partners@matambaintelligence.com), as [Get access and support](https://docs.matambaintelligence.com/support) describes. Never send a whole API key or signing secret.

## Licence

MIT. See [LICENSE](LICENSE). The skill holds no keys and no customer data.
