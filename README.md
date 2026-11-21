# mumo — Codex plugin

**Multi-model deliberation panel for OpenAI Codex.** When Codex is about to make an architecture choice, design tradeoff, or security-sensitive change worth checking across labs, mumo runs a panel of frontier models in parallel — Claude, GPT, Gemini, Grok, DeepSeek, Kimi, and more — and returns each model's full response plus typed cross-model reactions — what each model keeps, challenges, or wants explored in the others' claims.

## What's in the box

- **`.codex-plugin/plugin.json`** — Codex plugin manifest (name, version, MCP pointer, interface metadata).
- **`.mcp.json`** — mumo MCP server config (HTTP transport against `https://mumo.chat/api/mcp/oauth`, connected with a browser sign-in; no API key).
- **`skills/mumo/SKILL.md`** — the canonical skill teaching Codex how to use mumo: when to invoke, the deliberation loop (create → wait → read → summarize → append/stop), how to read the reactions, how to write your own, and when to verify session creation.
- **`skills/mumo/agents/openai.yaml`** — Codex-native skill metadata (display name, brand color, MCP dependency declaration, implicit-invocation policy).
- **`skills/mumo/playbooks/`** — four cognitive-shape playbooks loaded on demand: `contested-decision`, `design-review`, `uncertainty-expansion`, `red-team`.
- **`skills/mumo/references/`** — six reference docs: `reading-reactions`, `writing-reactions`, `model-selection`, `synthesis`, `takeaway`, `operating-notes`.

## Install

You need a mumo account; sign up at [mumo.chat](https://mumo.chat) (every account gets free credit each month). No API key and no environment variable.

### 1. Add the mumo marketplace

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

### 2. Install the mumo plugin

From a terminal:

```bash
codex plugin add mumo@mumo
```

**Or install from the Codex app:**

1. Open the Codex desktop app.
2. In the side nav, click **Plugins**.
3. Click the publisher dropdown (defaults to **Built by OpenAI**) and select **mumo**.
4. On the mumo plugin card under **Productivity**, click the **+** button.
5. In the install dialog, click **Install mumo**.

When the install completes, `~/.codex/config.toml` ends up with `[plugins."mumo@mumo"]` + `enabled = true`, and the plugin's skill + MCP server are registered.

The plugin browser is also available inside a Codex CLI session: run `/plugins`, switch to the mumo marketplace tab, select mumo, press Enter, and choose **Install plugin**.

### 3. Sign in to mumo

From a terminal:

```bash
codex mcp login mumo
```

1. Your browser opens to **Connect Codex to mumo**. Sign in to mumo if asked, confirm your account, and click **Continue**.
2. On the consent screen, click **Allow access**. The screen shows an address on this computer (`127.0.0.1`): that is Codex waiting for the sign-in to finish.
3. When the browser says authentication is complete, close the tab.

Codex stores the sign-in and renews it. To disconnect later, run `codex mcp logout mumo`, or remove **Codex** under **Settings → API Keys → Connected apps** at [mumo.chat/settings/api-keys](https://mumo.chat/settings/api-keys).

### 4. Start a new thread and verify

Start a new Codex thread (or restart Codex) so the server attaches. The mumo tools become available: `create_deliberation`, `ask_a_model`, `wait_for_round`, `append_round`, `get_session`, `list_sessions`, `list_models`, `share_session`, `get_credit`.

Check the sign-in:

```bash
codex mcp list
```

`mumo` should show `OAuth` in the Auth column. Then ask Codex in a thread:

> Ask mumo to list available models.

That hits `list_models` (read-only, no credit consumed). If it returns a list, you're ready to run a real deliberation.

### Using an API key instead

For CI or an unattended agent, skip the plugin and register the key server directly, with a key from [Settings → API Keys](https://mumo.chat/settings/api-keys) (keys start with `mmo_live_`) exported as `MUMO_API_KEY`:

```bash
codex mcp add mumo \
  --url https://mumo.chat/api/mcp \
  --bearer-token-env-var MUMO_API_KEY
```

This gives you the tools without the plugin's skill. On macOS, a Codex app launched from Finder or the Dock does not read your shell profile, so also run `launchctl setenv MUMO_API_KEY "$MUMO_API_KEY"`.

### Upgrading from 0.8.0 or earlier

Earlier versions read `MUMO_API_KEY` from your environment. From 0.8.1 the plugin signs in instead, so after updating, run `codex mcp login mumo` once. The environment variable is no longer used by the plugin and can be removed; the key stays valid on your account until you revoke it.

## Using the panel

In a Codex session, name `mumo` explicitly the first time so the agent reaches for the panel instead of answering directly:

> Ask mumo to compare Postgres and MongoDB for our event store given 50k events/day, a Postgres-experienced team, and a 3-month runway. What would we regret 6 months in?

Codex calls `create_deliberation`, then `wait_for_round`. The completed round returns each model's response plus the models' reactions to each other's claims, showing where the panel agrees and where it splits. The skill teaches Codex to read the reactions first and summarize the round for you, then, if you want another round, append one steered with its own typed reactions (KEEP / EXPLORE / CHALLENGE / CORE / SHIFT).

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
