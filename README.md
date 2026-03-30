# Kudosity SMS Extension for Gemini CLI

Send SMS, MMS, manage contact lists, and configure webhooks using the [Kudosity](https://kudosity.com) messaging platform — directly from Gemini CLI.

## Overview

This extension integrates Gemini CLI with the Kudosity messaging APIs, enabling you to send SMS and MMS messages, create and manage contact lists, and configure webhooks through natural language commands.

All API operations are executed **locally on your machine** by Gemini CLI using `curl` with your own API credentials. The extension connects to a remote MCP server for API documentation lookup only — no message data or credentials are sent through it.

## Features

| Skill | Description |
|-------|-------------|
| `send-sms` | Send SMS to individuals or contact lists (V1 + V2 API) |
| `send-mms` | Send multimedia messages with images, GIFs, video, or audio (V2 API) |
| `create-list` | Create contact lists and add recipients (V1 API) |
| `setup-webhook` | Configure delivery status notifications, inbound messages, and more (V2 API) |

The extension also includes:
- `/setup` — credential verification command
- `kudosity-assistant` — a specialized sub-agent for coordinating multi-step messaging operations

## Prerequisites

1. **Kudosity account** — [Sign up for your free account](https://kudosity.com/signup)
2. **API credentials** — Found in your account under **Developers → API Settings**
3. **Gemini CLI** — [Install Gemini CLI](https://geminicli.com)

## Installation

### From the Gemini CLI extension gallery

```bash
gemini extensions install https://github.com/kudosity/gemini-kudosity-sms-extension
```

### For local development

```bash
git clone https://github.com/kudosity/gemini-kudosity-sms-extension.git
cd gemini-kudosity-sms-extension
gemini extensions link .
```

## Setup

### 1. Configure your API credentials

When you install the extension, Gemini CLI will prompt you for your API Key and Secret. These are stored securely in your system keychain.

To reconfigure credentials later:

```bash
gemini extensions config gemini-kudosity-sms-extension
```

Or set environment variables manually in your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export KUDOSITY_API_KEY="your_api_key"
export KUDOSITY_API_SECRET="your_api_secret"
```

### 2. Verify your credentials

Run the setup command in Gemini CLI:

```
/setup
```

This will verify both your V1 and V2 API credentials and show your account balance.

## Usage Examples

### Create a contact list

> "Create a contact list called 'March Campaign' with fields for email and postcode, then add John Doe (0491570156) and Jane Smith (0435795809)"

### Send an SMS

> "Send an SMS to 0491570156 from 61481074185 saying 'Your order has shipped!'"

> "Send an SMS to my 'March Campaign' list saying 'Sale starts tomorrow!'"

### Send an MMS

> "Send an MMS to 0435795809 with the image at https://example.com/product.jpg, subject 'New Arrival'"

### Set up a webhook

> "Set up a webhook at https://myapp.com/sms-status to receive SMS delivery status and inbound messages"

## Commands

| Command | Description |
|---------|-------------|
| `/setup` | Credential verification — checks that your API Key and Secret are configured and validates them against both V1 and V2 APIs |

## Skills

| Skill | Description |
|-------|-------------|
| `send-sms` | Send SMS to a single recipient (V2 API) or to contact lists / multiple numbers (V1 API). Supports scheduling, link tracking, and delivery callbacks. |
| `send-mms` | Send multimedia messages with images, GIFs, video, or audio attachments (V2 API). |
| `create-list` | Create contact lists and add contacts for campaign sends (V1 API). |
| `setup-webhook` | Configure webhooks for delivery status, inbound messages, MMS events, link hits, and opt-outs (V2 API). |

## Sub-Agents

| Agent | Description |
|-------|-------------|
| `kudosity-assistant` | A specialized messaging assistant that coordinates multi-step Kudosity operations. Understands API routing rules, authentication methods, and can chain multiple operations together. |

> **Note:** Sub-agents are an experimental feature in Gemini CLI. The extension's `GEMINI.md` context file provides equivalent instructions as a fallback.

## API Coverage

| Operation | API Version | Auth Method |
|-----------|------------|-------------|
| Create list | V1 | Basic Auth |
| Add contact to list | V1 | Basic Auth |
| Send SMS (single recipient) | V2 | x-api-key |
| Send SMS (to list / multiple) | V1 | Basic Auth |
| Send MMS | V2 | x-api-key |
| Create/manage webhooks | V2 | x-api-key |

The full API specifications are available as OpenAPI documents:
- **V1**: [`api_documentation.yml`](https://developers.kudosity.com/openapi/api_documentation.yml) — Lists, contacts, bulk SMS
- **V2**: [`public-openapi.yaml`](https://developers.kudosity.com/openapi/public-openapi.yaml) — Single SMS, MMS, webhooks

## Architecture & Trust Boundaries

This extension has two distinct components with different trust boundaries:

### MCP server (remote, read-only)

The extension connects to the public Kudosity MCP server at [`developers.kudosity.com/mcp`](https://developers.kudosity.com/mcp). This server provides **API documentation and endpoint metadata only**. It does not execute API operations, receive message content, or have access to your credentials.

MCP tools available:
- `list-specs`, `list-endpoints`, `get-endpoint`, `search-endpoints` — browse and search API documentation
- `route-kudosity-operations` — determines which API version to use for a given operation

### API execution (local, authenticated)

All actual API operations (sending messages, creating lists, configuring webhooks) are executed **locally by Gemini CLI** on your machine using `curl` commands. API calls go directly from your machine to the Kudosity API servers:

- **V1 API** (`api.transmitsms.com`) — Basic Auth with `-u` flag
- **V2 API** (`api.transmitmessage.com`) — `x-api-key` header

Your credentials are never sent to the MCP server or to Gemini. They are used only in direct API calls to Kudosity.

## Security

- **Credentials** are stored securely in your system keychain (when configured via `gemini extensions config`) or as environment variables. They are not embedded in any extension file, sent to Gemini, or transmitted to any remote service other than the Kudosity API during authenticated requests.
- **API calls** are executed locally by Gemini CLI using `curl`. Requests go directly from your machine to the Kudosity API — they do not pass through the MCP server or any intermediary.
- **MCP server** access is read-only — it serves API documentation and endpoint metadata only. No credentials, message content, or user data are sent to it.
- **No telemetry or analytics** are collected by this extension.

## Extension Structure

```
├── gemini-extension.json          # Extension manifest
├── GEMINI.md                      # Persistent context/instructions
├── skills/
│   ├── create-list/SKILL.md       # Contact list management
│   ├── send-sms/SKILL.md         # SMS sending
│   ├── send-mms/SKILL.md         # MMS sending
│   └── setup-webhook/SKILL.md    # Webhook configuration
├── agents/
│   └── kudosity-assistant.md      # Specialized messaging sub-agent
├── commands/
│   └── setup.toml                 # Credential verification command
├── .gitignore
├── CHANGELOG.md
├── LICENSE
├── PROJECT.md
└── README.md
```

## Support & Resources

- **API Documentation**: [developers.kudosity.com](https://developers.kudosity.com)
- **MCP Server**: [developers.kudosity.com/mcp](https://developers.kudosity.com/mcp)
- **V1 API Spec (OpenAPI)**: [api_documentation.yml](https://developers.kudosity.com/openapi/api_documentation.yml)
- **V2 API Spec (OpenAPI)**: [public-openapi.yaml](https://developers.kudosity.com/openapi/public-openapi.yaml)
- **Kudosity Support**: [kudosity.com](https://kudosity.com)

## License

MIT — see [LICENSE](LICENSE)