# Changelog

All notable changes to the Kudosity SMS Extension for Gemini CLI will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-30

### Added

- Initial release — ported from the [Kudosity Claude Code plugin](https://github.com/kudosity/kudosity-claude-sms-plugin)
- **Skills:**
  - `send-sms` — Send SMS to single recipients (V2 API) or contact lists/multiple numbers (V1 API)
  - `send-mms` — Send multimedia messages with images, GIFs, video, or audio (V2 API)
  - `create-list` — Create contact lists and add contacts (V1 API)
  - `setup-webhook` — Configure webhooks for delivery status, inbound messages, and more (V2 API)
- **Commands:**
  - `/setup` — Verify API credentials against V1 and V2 APIs
- **Sub-Agents:**
  - `kudosity-assistant` — Specialized messaging assistant for multi-step operations
- **Context:**
  - `GEMINI.md` — Persistent instructions with API routing rules, auth methods, and behavior guidelines
- **MCP Server:**
  - Connection to `developers.kudosity.com/mcp` for API documentation lookup
- **Extension Settings:**
  - `KUDOSITY_API_KEY` — Stored securely in system keychain
  - `KUDOSITY_API_SECRET` — Stored securely in system keychain