# Changelog

## 0.2.0 — 2026-05-05

Two install-blocker fixes from first user attempt. Structural; the install command shape changes.

- **Repo is now a marketplace, not a bare plugin.** `codex plugin marketplace add owner/repo` requires the repo root to contain a `marketplace.json` listing one or more plugins; v0.1.x failed at install with `marketplace root does not contain a supported manifest`. The plugin moved into `plugins/mumo/` and `marketplace.json` at the repo root references it via `source.path: "./plugins/mumo"`.
- **Wrong marketplace-source format in README.** v0.1.x used `codex plugin marketplace add github:mumo-chat/mumo-codex`. The CLI accepts `owner/repo`, a full git URL, or a local path — no `github:` shorthand. README and install page now use `codex plugin marketplace add mumo-chat/mumo-codex`. Original form was copied from a planning doc without verifying against the actual CLI parser.
- **Plugin manifest** bumped to `0.2.0` to match the structural change.

## 0.1.2 — 2026-05-05

Setup-doc fix and version reconciliation.

- **README:** added the `npm i -g @openai/codex` prerequisite to step 3. The Codex CLI ships separately from the IDE extension; users who only had the IDE installed hit `zsh: command not found: codex` on the marketplace command.
- **`plugin.json` version:** bumped to `0.1.2` so the plugin manifest matches the CHANGELOG. Through 0.1.1 the manifest read `0.1.0` because the prior bump (launchctl docs) didn't touch behavior. From here on, every CHANGELOG entry comes with a manifest bump.

## 0.1.1 — 2026-05-05

macOS GUI-launch fix in setup docs. Pure documentation; no skill or manifest behavior change.

- **README + SKILL.md:** `launchctl setenv MUMO_API_KEY ...` is now the canonical macOS step. Shell `export` is documented as optional, "if you want terminal sessions to have the key too." Mirrors the same fix the mumo-cursor install flow already uses. Reason: macOS GUI apps launched from Finder/Dock/Spotlight don't inherit shell environment variables, so a Codex IDE extension launched from Finder couldn't see a `MUMO_API_KEY` set only via `~/.zshrc`. `launchctl setenv` puts it in the launchd session that all GUI apps inherit.

## 0.1.0 — 2026-05-05

Initial release. Codex plugin for mumo's MCP server.

- `.codex-plugin/plugin.json` — Codex plugin manifest with `mcpServers` pointer and interface metadata (display name, brand color, default prompt, category).
- `.mcp.json` — HTTP transport against `https://mumo.chat/api/mcp`, `bearer_token_env_var: MUMO_API_KEY`, `enabled_tools` allowlist scoped to mumo's seven tools.
- `skills/mumo/SKILL.md` — kernel teaching the deliberation loop, snippet doctrine, claim-map reading, when-to-invoke triggers, and verification discipline (real session IDs are UUIDs; verify with `list_sessions` if uncertain).
- `skills/mumo/agents/openai.yaml` — Codex-native sidecar with `allow_implicit_invocation: true` and an MCP dependency declaration so Codex prompts for setup on first use.
- `skills/mumo/playbooks/` — four cognitive-shape playbooks: `contested-decision`, `design-review`, `uncertainty-expansion`, `red-team`.
- `skills/mumo/references/` — five reference docs: `claim-maps`, `snippets`, `model-selection`, `synthesis`, `operating-notes`.
- README install steps: export `MUMO_API_KEY`, run `codex plugin marketplace add github:mumo-chat/mumo-codex`, restart Codex.

Derived from the v0.2.x skill kernel that originated in mumo-mcp and mumo-cursor and was adapted for Hermes in mumo-hermes. Codex's plugin format (manifest + `.mcp.json` + skills + Codex-native YAML sidecar) is closest to mumo-mcp; the HTTP transport uses Codex's `bearer_token_env_var` env-var pointer rather than a literal Bearer header in YAML, so the API key lives in the user's shell environment, not in any committed config file.
