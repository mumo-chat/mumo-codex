# mumo — Codex plugin

**Multi-model deliberation panel for OpenAI Codex.** When Codex is about to make an architecture choice, design tradeoff, or security-sensitive change you want a second opinion on, mumo runs a panel of frontier models in parallel — Claude, GPT, Gemini, Grok, Qwen, Kimi, GLM — and returns a cross-model claim map showing where they agree and where they split.

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

### 2. Set MUMO_API_KEY for Codex

The plugin's `.mcp.json` declares `bearer_token_env_var: MUMO_API_KEY`, which tells Codex to read the token from your environment. On macOS, set it from Terminal so the Codex IDE extension launched from Finder/Dock inherits it (GUI apps don't read your shell profile):

```bash
launchctl setenv MUMO_API_KEY mmo_live_YOUR_KEY_HERE
```

Optional: also add `export MUMO_API_KEY=mmo_live_YOUR_KEY_HERE` to `~/.zshrc` if you want terminal sessions to have the key too.

### 3. Add the mumo marketplace

If you don't have the Codex CLI yet, install it from npm:

```bash
npm i -g @openai/codex
```

Then register the marketplace:

```bash
codex plugin marketplace add mumo-chat/mumo-codex
```

Or from a local clone:

```bash
git clone https://github.com/mumo-chat/mumo-codex
codex plugin marketplace add ./mumo-codex
```

This adds `[marketplaces.mumo]` to `~/.codex/config.toml`. **Adding the marketplace does not auto-enable the plugin** — that's the next step.

### 4. Install the mumo plugin from inside Codex

The plugin browser is interactive and lives inside a Codex session. Open Codex (`codex` to start a new session, or any active session), then run:

```
/plugins
```

In the plugin browser:

1. Switch to the **mumo** marketplace tab (the marketplace you registered in step 3).
2. Select **mumo**, press Enter.
3. Choose **Install plugin**.

When the install completes, `~/.codex/config.toml` ends up with `[plugins."mumo@mumo"]` + `enabled = true`, and the plugin's skill + MCP server are registered.

**Fallback:** if the plugin browser flow isn't working on your Codex build, you can register the MCP server directly. This gives you the seven mumo tools but skips the skill (the agent loses the deliberation-loop instructions and has to discover the tools on its own):

```bash
codex mcp add mumo \
  --url https://mumo.chat/api/mcp \
  --bearer-token-env-var MUMO_API_KEY
```

### 5. Restart Codex (or start a new thread)

Start a new Codex thread (or fully restart Codex) so the freshly-installed MCP server attaches. After restart, the seven mumo tools become available: `create_deliberation`, `wait_for_round`, `append_round`, `get_session`, `list_sessions`, `list_models`, `get_credit`.

### 6. Verify with a low-cost call

Before running a full deliberation, smoke-test that auth and tool routing are working. Either run:

```bash
codex mcp list
```

…and confirm `mumo` is listed, or ask Codex in a thread:

> Ask mumo to list available models.

That hits `list_models` (read-only, no credit consumed). If it returns a list, you're ready to run a real deliberation.

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
