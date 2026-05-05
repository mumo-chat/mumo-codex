# Changelog

## 0.3.4 — 2026-05-05

GUI-first install flow + skill subtitle alignment.

- **README install step 4** rewritten to lead with the Codex desktop app flow (side nav → Plugins → publisher dropdown → select mumo → "+" on the plugin card → "Install mumo"). The CLI `/plugins` flow is now an alternative for headless / terminal-only workflows. Most users live in the GUI; the install card is cleaner there.
- **`agents/openai.yaml` `short_description`** updated from "Multi-model deliberation panel for high-regret decisions" (Hermes-flavored, predated v0.3.3 copy revision) to "Multi-model deliberation panel. Diverse AI voices on contested decisions." Aligns the Skill subtitle in the install card's "Includes" section with the top-level plugin subtitle.

## 0.3.3 — 2026-05-05

End-to-end install confirmed: marketplace discovery, plugin install via in-session `/plugins`, skill loading, MCP server registration, and `list_models` round-trip all working. This release locks in copy + branding for the install card and rewrites the README install flow to reflect what actually works.

- **Logo** added at `plugins/mumo/assets/logo.png` (sourced from mumo-cursor). `interface.logo` and `interface.composerIcon` in `plugin.json` now point at it; previously the install card showed Codex's generic placeholder.
- **Copy revised for IDE-native audience.** Codex is the agent, so the prior "your local agent is decision-bound by its backbone model" framing (carried over from the Hermes kernel) was sideways. Now: short description matches the mumo-mcp / mumo-cursor family tagline ("Multi-model deliberation panel. Diverse AI voices on contested decisions."); long description leads with "When Codex is about to make an architecture choice..."; default prompt is "Have mumo pressure-test this design before I commit." (sharper, code-task-flavored, matches the prompt-chip pattern in OpenAI's bundled plugins).
- **Marketplace.json shortDescription** aligned with plugin.json so the install card and the marketplace catalog show consistent copy.
- **README install flow rewritten** to match what actually unblocked end-to-end install: `/plugins` is invoked **inside a live Codex session** (not as a top-level subcommand); user picks the mumo marketplace tab, selects mumo, hits Enter, chooses "Install plugin." Step 5 is "restart or new thread." Step 6 is now an explicit low-cost verification — `codex mcp list` or "Ask mumo to list available models" — before running a real (credit-consuming) deliberation.
- **Author block** now includes `url: "https://mumo.chat"` matching the family convention.
- **Keywords** broadened from `[deliberation, multi-model, mcp, decision-support, agents]` to the family-shared list `[mcp, deliberation, multi-model, claude, gpt, gemini, panel, architecture, design-review, code-review]`.

Honest postmortem on what unblocked discovery: somewhere across v0.3.0 → v0.3.2 + the in-session `/plugins` realization, mumo became visible. Most plausible single contributor: v0.3.2's restoration of `mcpServers: "./.mcp.json"` in `plugin.json` (which v0.3.1 had removed based on `browser-use` not having it; turns out `computer-use` does, and apparently Codex needs the explicit pointer to attach the MCP server). But the move-to-`plugins/<name>/` (v0.3.0) and `author` object shape (v0.3.1) and `defaultPrompt` array (v0.3.0) were all real shape diffs from canonical too. Not isolating further; leaving the postmortem honest.

## 0.3.2 — 2026-05-05

Manifest fix from comparing against OpenAI's bundled `computer-use` plugin. `plugin.json` now points at the root `.mcp.json` via `mcpServers: "./.mcp.json"`; without that pointer, Codex can discover the plugin shell but may not attach the MCP server/tools.

This corrects v0.3.1's removal of the same field. v0.3.1's reasoning was that OpenAI's `browser-use` plugin doesn't include `mcpServers` in its manifest, so the field looked optional. Sampling a second OpenAI-bundled plugin (`computer-use`) showed `browser-use` is the outlier, and Codex MCP plugins do need the explicit pointer in practice — `browser-use` may have a non-MCP path for its tools, or be relying on a default that doesn't apply to remote HTTP servers. Either way, the v0.3.1 entry's "Codex auto-discovers `.mcp.json` at the plugin root" claim was wrong; treat that line as historical/incorrect.

## 0.3.1 — 2026-05-05

`plugin.json` schema fixes from comparing to OpenAI's `browser-use` plugin manifest at `~/.codex/.tmp/bundled-marketplaces/openai-bundled/plugins/browser-use/.codex-plugin/plugin.json`. Three diffs from the canonical shape:

- **`author` is an object, not a string.** Was `"author": "mumo"`, now `"author": {"name": "mumo"}`. OpenAI's manifest uses the object form, and any strict JSON-schema validator rejects a string here. This is the most likely fatal validation diff.
- **Removed top-level `mcpServers` field.** OpenAI's `browser-use` doesn't reference `.mcp.json` from the manifest; Codex auto-discovers `.mcp.json` at the plugin root. Keeping the manifest field around could be either ignored or rejected depending on schema strictness.
- **Added `interface.capabilities: ["Interactive"]`.** OpenAI uses `["Interactive", "Read", "Write"]`; we declare just `Interactive` since mumo surfaces tools the agent invokes — no filesystem reads/writes from the plugin itself.
- **Added `interface.screenshots: []`** to match OpenAI's shape.

## 0.3.0 — 2026-05-05

Three structural fixes from inspecting OpenAI's bundled marketplace at `~/.codex/.tmp/bundled-marketplaces/openai-bundled` (canonical Codex format on disk; the published docs are partially aspirational). v0.2.x had Codex registering the marketplace but listing zero plugins from it; this fixes that.

- **Plugin moved to repo-root `plugins/<name>/`.** v0.2.x kept the plugin under `.agents/plugins/mumo/` next to `marketplace.json`. OpenAI's bundled marketplace puts plugins at `plugins/<name>/` (repo root) with `marketplace.json` separately at `.agents/plugins/marketplace.json`. The Codex docs implied plugin paths resolve relative to `marketplace.json`'s directory; they actually resolve relative to **repo root**. So our v0.2.x `source.path: "./mumo"` was looking for `<repo>/mumo/`, which didn't exist, and Codex silently dropped the plugin.
- **`source.path` now `./plugins/mumo`** to match the new layout (and OpenAI's convention).
- **`.mcp.json` now wraps in `mcpServers`.** Was `{"mumo": {...}}`, now `{"mcpServers": {"mumo": {...}}}`. The Codex docs showed both forms; the bundled examples uniformly use the wrapped form.
- **`interface.defaultPrompt` is an array.** Was a string; OpenAI bundled manifests use an array. Codex may accept the string but matching the canonical shape removes a guessing axis.
- **NOT switching to `.claude-plugin/`.** Codex tries to parse Claude Code marketplaces but logs `unsupported source` for several entries. Claude shape is back-compat, not native. We stay on `.codex-plugin/plugin.json`.

## 0.2.2 — 2026-05-05

Install docs fix. Adding the marketplace and enabling the plugin are two distinct steps in Codex; v0.2.x docs conflated them, so users who ran `codex plugin marketplace add mumo-chat/mumo-codex` and then restarted saw zero mumo tools — the plugin sat in the marketplace catalog but was never enabled.

- **README + install page:** step 3 (Add the marketplace) and step 4 (Enable the plugin via `codex /plugins`) are now separate. `[plugins."mumo@mumo"]` + `enabled = true` is what the user is verifying landed in `~/.codex/config.toml`.
- **Fallback path documented:** for Codex builds where the plugin browser doesn't enable mumo correctly, `codex mcp add mumo --url https://mumo.chat/api/mcp --bearer-token-env-var MUMO_API_KEY` registers the MCP server directly. Tools surface but the skill is skipped — the agent loses the deliberation-loop instructions and has to discover the tools on its own.
- **`codex mcp list`** added to step 5 as the verification step.

## 0.2.1 — 2026-05-05

`marketplace.json` moved to its canonical path `.agents/plugins/marketplace.json` (was at repo root in v0.2.0; Codex still rejected it with `marketplace root does not contain a supported manifest`). Plugin moved alongside it at `.agents/plugins/mumo/`. `source.path` is now `./mumo` (relative to `.agents/plugins/`).

## 0.2.0 — 2026-05-05

Two install-blocker fixes from first user attempt. Structural; the install command shape changes.

- **Repo is now a marketplace, not a bare plugin.** `codex plugin marketplace add owner/repo` requires the repo to contain a `marketplace.json` listing one or more plugins; v0.1.x failed at install with `marketplace root does not contain a supported manifest`. (v0.2.0 placed it at repo root; v0.2.1 moves it to the canonical `.agents/plugins/marketplace.json` after that didn't resolve the error.) The plugin moved into `plugins/mumo/` and `marketplace.json` at the repo root references it via `source.path: "./plugins/mumo"`.
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
