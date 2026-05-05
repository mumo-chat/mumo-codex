# mumo — Codex plugin

**Multi-model deliberation for OpenAI Codex.** When your Codex agent is decision-bound by its backbone model's confidence, mumo gives it on-demand access to a panel of frontier models — Claude, GPT, Gemini, Grok, Qwen, Kimi, GLM — for the hard calls.

For Claude Code, see [`mumo-chat/mumo-mcp`](https://github.com/mumo-chat/mumo-mcp). For Cursor, see [`mumo-chat/mumo-cursor`](https://github.com/mumo-chat/mumo-cursor). For VS Code, see [`mumo-chat/mumo-vscode`](https://github.com/mumo-chat/mumo-vscode). For Hermes Agent, see [`mumo-chat/mumo-hermes`](https://github.com/mumo-chat/mumo-hermes).

## What's in the box

- **`.codex-plugin/plugin.json`** — Codex plugin manifest (name, version, MCP pointer, interface metadata).
- **`.mcp.json`** — mumo MCP server config (HTTP transport against `https://mumo.chat/api/mcp`, reads `MUMO_API_KEY` from the environment).
- **`skills/mumo/SKILL.md`** — the canonical skill teaching Codex how to use mumo: when to invoke, the deliberation loop (create → wait → read → snippet → append/stop), how to read claim maps, snippet doctrine, and when to verify session creation.
- **`skills/mumo/agents/openai.yaml`** — Codex-native skill metadata (display name, brand color, MCP dependency declaration, implicit-invocation policy).
- **`skills/mumo/playbooks/`** — four cognitive-shape playbooks loaded on demand: `contested-decision`, `design-review`, `uncertainty-expansion`, `red-team`.
- **`skills/mumo/references/`** — five reference docs: `claim-maps`, `snippets`, `model-selection`, `synthesis`, `operating-notes`.

## Install

### 1. Get an API key

Sign up at [mumo.chat](https://mumo.chat) and create a platform key at [Settings → API Keys](https://mumo.chat/settings/api-keys). Keys start with `mmo_live_`.

### 2. Export the key

```bash
export MUMO_API_KEY=mmo_live_YOUR_KEY_HERE
```

Add it to your shell profile (`.zshrc`, `.bashrc`, etc.) so Codex picks it up across sessions.

### 3. Install the plugin

```bash
codex plugin marketplace add github:mumo-chat/mumo-codex
```

Or from a local clone:

```bash
git clone https://github.com/mumo-chat/mumo-codex
codex plugin marketplace add ./mumo-codex
```

### 4. Restart Codex

Restart Codex (CLI or IDE extension) so the MCP server registers and `MUMO_API_KEY` propagates. After restart, the seven mumo tools become available: `create_deliberation`, `wait_for_round`, `append_round`, `get_session`, `list_sessions`, `list_models`, `get_credit`.

## Using the panel

In a Codex session, name `mumo` explicitly the first time so the agent reaches for the panel instead of answering directly:

> Ask mumo to compare Postgres and MongoDB for our event store given 50k events/day, a Postgres-experienced team, and a 3-month runway. What would we regret 6 months in?

Codex calls `create_deliberation`, then `wait_for_round`. The completed round returns each model's prose plus a cross-model claim map showing where the panel agrees and where it splits. The skill teaches Codex to read the claim map first, then react with typed snippets (KEEP / EXPLORE / CHALLENGE / CORE / SHIFT) and either append a follow-up round or stop and synthesize for you.

## When mumo is worth the latency tax

The skill encodes the trigger taxonomy in detail. In short:

- Architecture decisions with non-obvious tradeoffs
- Plan or design review before commitment
- Pre-launch pressure tests
- Stuck debugging after repeated failed attempts
- Pre-commit adversarial review on risky diffs (auth, payments, migrations)
- Strategy questions with multiple defensible framings
- Explicit user requests

Skip mumo for routine refactors, formatting, syntax help, or anything where "just write a test" is cheaper than discussion.

## Verifying the call actually fired

Autonomous agents occasionally fabricate tool-call results — claiming a deliberation was sent when it wasn't. Real mumo session IDs are UUIDs (e.g. `2acdab34-2484-4bc5-a24f-bf917fe81477`). If a `create_deliberation` response doesn't contain a UUID-format `session_id`, the call did not happen. Verify by calling `list_sessions`. The skill teaches this discipline; this README is the user-facing reminder.

## Links

- Product — https://mumo.chat
- Install guide — https://mumo.chat/install/codex
- MCP reference — https://mumo.chat/docs/mcp
- REST API — https://mumo.chat/docs/api
- Codex plugins docs — https://developers.openai.com/codex/plugins
- Issues — https://github.com/mumo-chat/mumo-codex/issues

## License

MIT
