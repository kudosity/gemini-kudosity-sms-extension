# Kudosity Gemini CLI Extension — Project Plan

## Project Overview

| Field | Value |
|-------|-------|
| **Project** | Port Kudosity SMS Claude Code plugin → Gemini CLI Extension |
| **Source Repo** | `https://github.com/kudosity/kudosity-claude-sms-plugin` |
| **Target Repo** | `https://github.com/kudosity/gemini-kudosity-sms-extension` |
| **Local Path** | `/Users/tbc/repo/gemini-kudosity-sms-extension` |
| **Type** | Migration (not a rewrite) |
| **Extension Type** | Pure content — no Node.js code, no build step, no dependencies |

## Source Plugin Assessment

The Claude Code plugin is a content-only plugin consisting entirely of markdown files that instruct Claude how to interact with the Kudosity messaging APIs via `curl` commands.

### Components

| Component | Claude File | Purpose |
|-----------|------------|---------|
| MCP Server Config | `.mcp.json` | Connects to `https://developers.kudosity.com/mcp` for API doc lookup |
| Plugin Manifest | `.claude-plugin/plugin.json` | Plugin name, version, author metadata |
| Marketplace Entry | `.claude-plugin/marketplace.json` | Claude marketplace catalog entry |
| Agent | `agents/kudosity-assistant.md` | Specialized messaging assistant with API routing |
| Setup Command | `commands/setup.md` | Interactive credential configuration wizard |
| Send SMS Skill | `skills/send-sms/SKILL.md` | SMS sending (V1 + V2 API) |
| Send MMS Skill | `skills/send-mms/SKILL.md` | MMS sending (V2 API) |
| Create List Skill | `skills/create-list/SKILL.md` | Contact list management (V1 API) |
| Setup Webhook Skill | `skills/setup-webhook/SKILL.md` | Webhook configuration (V2 API) |

### Claude-Specific vs Portable

| Part | Status | Notes |
|------|--------|-------|
| `.claude-plugin/` directory | ❌ Claude-specific | Replaced by `gemini-extension.json` |
| `.mcp.json` format | ❌ Claude-specific | Replaced by `mcpServers` in manifest |
| `commands/setup.md` format | ❌ Claude-specific | Gemini uses `.toml` format |
| Agent frontmatter (`model`, `effort`, `maxTurns`) | ❌ Claude-specific | Remove; Gemini uses settings.json overrides |
| All 4 `SKILL.md` files | ✅ Portable | Gemini uses identical format |
| Agent body content | ✅ Portable | Instructions transfer directly |
| MCP server URL | ✅ Portable | Gemini supports HTTP MCP via `httpUrl` |

---

## Phase 1 — Planning & Research ✅

- [x] Analyze Claude Code plugin structure and all files
- [x] Read Gemini CLI extension docs (writing-extensions, reference, releasing)
- [x] Read Gemini CLI custom commands format (TOML-based)
- [x] Read Gemini CLI skills format (SKILL.md with YAML frontmatter)
- [x] Read Gemini CLI sub-agents format (agents/*.md)
- [x] Confirm Gemini supports HTTP MCP via `httpUrl`
- [x] Map all Claude concepts → Gemini equivalents
- [x] Create this project plan

### Key Findings

| Claude Concept | Gemini Equivalent | Notes |
|---------------|-------------------|-------|
| `.mcp.json` | `mcpServers` in `gemini-extension.json` | Use `httpUrl` for streamable HTTP |
| `.claude-plugin/plugin.json` | `gemini-extension.json` | Single manifest replaces both |
| `.claude-plugin/marketplace.json` | GitHub topic `gemini-cli-extension` | Auto-discovered daily |
| `agents/*.md` | `agents/*.md` | Same dir, different frontmatter |
| `commands/*.md` | `commands/*.toml` | TOML format required |
| `skills/*/SKILL.md` | `skills/*/SKILL.md` | **Identical format** |
| Env vars in `~/.zshrc` | `settings` array in manifest | Stored in system keychain |

---

## Phase 2 — Core Extension Setup ✅

- [x] Create `gemini-extension.json` manifest
- [x] Create `GEMINI.md` context file
- [x] Create `.gitignore`

### Design Decisions

**MCP Server:** Use `httpUrl` for the remote Kudosity MCP server:
```json
"mcpServers": {
  "kudosity": {
    "httpUrl": "https://developers.kudosity.com/mcp"
  }
}
```

**API Credentials:** Use `settings` array — Gemini prompts on install and stores in keychain:
```json
"settings": [
  { "name": "API Key", "envVar": "KUDOSITY_API_KEY", "sensitive": true },
  { "name": "API Secret", "envVar": "KUDOSITY_API_SECRET", "sensitive": true }
]
```

---

## Phase 3 — Feature Migration ✅

- [x] Copy `skills/send-sms/SKILL.md` (unchanged)
- [x] Copy `skills/send-mms/SKILL.md` (unchanged)
- [x] Copy `skills/create-list/SKILL.md` (unchanged)
- [x] Copy `skills/setup-webhook/SKILL.md` (unchanged)
- [x] Convert `commands/setup.md` → `commands/setup.toml`
- [x] Adapt `agents/kudosity-assistant.md` (remove Claude-specific frontmatter)

### Command Conversion Notes

Claude `commands/setup.md` uses markdown with YAML frontmatter. Gemini requires `.toml` format with a `prompt` field. The setup wizard content transfers into the TOML `prompt` string. Since Gemini handles credential storage via `settings`, the setup command focuses on **verification** rather than credential saving.

### Agent Adaptation Notes

Remove Claude-specific frontmatter fields: `model: sonnet`, `effort: medium`, `maxTurns: 30`. Keep the markdown body unchanged. Note that Gemini sub-agents are **experimental**.

---

## Phase 4 — Documentation ✅

- [x] Create `README.md` (adapted for Gemini CLI)
- [x] Create `CHANGELOG.md`
- [x] Copy `LICENSE` (MIT, unchanged)

---

## Phase 5 — Testing & Validation

- [ ] Link extension locally: `gemini extensions link .`
- [ ] Verify extension loads: `gemini extensions list`
- [ ] Test setup command: `/setup`
- [ ] Test MCP server connection
- [ ] Test skill activation (send-sms, send-mms, create-list, setup-webhook)

> **Status:** Ready for manual testing.

---

## Phase 6 — Publishing ✅

- [x] Create GitHub repo at `github.com/kudosity/gemini-kudosity-sms-extension`
- [x] Push all files
- [x] Add `gemini-cli-extension` topic to repo (+ sms, mms, messaging, kudosity, gemini-cli)
- [ ] Verify gallery listing (auto-discovered daily by crawler)

### Gallery Requirements (from Gemini docs)

1. Public GitHub repository
2. `gemini-cli-extension` topic on the repo
3. `gemini-extension.json` at the repo root
4. Gallery crawler indexes tagged repos daily

---

## Target File Structure

```
gemini-kudosity-sms-extension/
├── gemini-extension.json          # Extension manifest
├── GEMINI.md                      # Persistent context/instructions
├── README.md                      # Documentation
├── LICENSE                        # MIT license
├── CHANGELOG.md                   # Version history
├── PROJECT.md                     # This file — project tracking
├── .gitignore                     # Git ignore rules
├── agents/
│   └── kudosity-assistant.md      # Specialized messaging sub-agent
├── commands/
│   └── setup.toml                 # Credential verification command
└── skills/
    ├── send-sms/
    │   └── SKILL.md               # SMS sending skill
    ├── send-mms/
    │   └── SKILL.md               # MMS sending skill
    ├── create-list/
    │   └── SKILL.md               # Contact list management skill
    └── setup-webhook/
        └── SKILL.md               # Webhook configuration skill
```

---

## Porting Notes & Caveats

| Issue | Impact | Mitigation |
|-------|--------|------------|
| Agent frontmatter (`model`, `effort`, `maxTurns`) are Claude-specific | Gemini sub-agents don't support these | Remove; use `settings.json` overrides |
| Sub-agents are **experimental** in Gemini | May not work in all environments | `GEMINI.md` provides fallback context |
| Commands are `.toml` not `.md` | Setup wizard needs format conversion | Content stays the same, just wrapped in TOML |
| Claude's setup writes to `~/.zshrc` | Gemini handles secrets via keychain | Setup command focuses on verification |
| No `package.json` needed | No Node.js dependencies | Pure content extension |
| Skill references `/kudosity-sms:setup` | Gemini command namespace different | Update to `/setup` (or `/kudosity-sms:setup` if namespaced) |