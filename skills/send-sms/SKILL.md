---
name: send-sms
description: Send SMS messages to a recipient or contact list, with optional scheduling and delivery tracking.
---

# Send SMS Messages

Kudosity supports two ways to send SMS depending on the use case:
- **Single recipient** → V2 API (simpler, JSON-based)
- **List-based send / multiple recipients** → V1 API (supports list_id and comma-separated numbers)

## Authentication

**V2 API** (single recipient):
- Header: `x-api-key: {KUDOSITY_API_KEY}`

**V1 API** (list-based or multi-recipient):
- Header: `Authorization: Basic {base64(KUDOSITY_API_KEY:KUDOSITY_API_SECRET)}`

If the user hasn't configured credentials, direct them to run `/setup`.

## Option A: Send to a Single Recipient (V2 API)

**Endpoint**: `POST https://api.transmitmessage.com/v2/sms`
**Content-Type**: `application/json`

All API calls use `curl` commands.

Required parameters:
- `message` (string): The SMS body text
- `sender` (string): The sender number (must be assigned to the account) or alphanumeric sender ID (max 11 chars)
- `recipient` (string): Destination number (local or E.164 international format)

Optional parameters:
- `message_ref` (string, max 500 chars): Your reference ID, returned in webhooks
- `track_links` (boolean): Enable link tracking with shortened URLs

Example curl command:
```bash
curl -s -X POST "https://api.transmitmessage.com/v2/sms" \
  -H "x-api-key: ${KUDOSITY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"message": "Your order has shipped!", "sender": "61481074185", "recipient": "61491570156"}'
```

## Option B: Send to a List or Multiple Numbers (V1 API)

**Endpoint**: `POST https://api.transmitsms.com/send-sms.json`
**Content-Type**: `application/x-www-form-urlencoded`

Required parameters:
- `message` (string): The SMS body (up to 612 alphanumeric characters)
- One of:
  - `to` (string): Single number or up to 500 comma-separated numbers
  - `list_id` (integer): ID of a contact list to send to

Optional parameters:
- `from` (string): Sender ID — a virtual number, short code, or alphanumeric sender (max 11 chars)
- `countrycode` (string): 2-letter ISO country code to auto-format local numbers
- `send_at` (datetime): Schedule send in ISO8601 format `YYYY-MM-DD HH:MM:SS` (UTC)
- `validity` (integer): Message expiry in minutes (0 = max validity, auto-expires after 72hrs)
- `dlr_callback` (URL): Delivery receipt callback URL
- `reply_callback` (URL): Reply notification callback URL
- `tracked_link_url` (URL): URL to convert to a tracked shortened link (use `[tracked-link]` in message)
- `replies_to_email` (email): Forward replies to this email address

Example curl command (list-based):
```bash
curl -s -X POST "https://api.transmitsms.com/send-sms.json" \
  -u "${KUDOSITY_API_KEY}:${KUDOSITY_API_SECRET}" \
  -d "message=Sale starts tomorrow!&list_id=4213644&from=61481074185"
```

## Choosing Between V1 and V2

| Scenario | Use | API |
|----------|-----|-----|
| Single recipient, simple send | V2 | `api.transmitmessage.com/v2/sms` |
| Send to a contact list | V1 | `api.transmitsms.com/send-sms.json` |
| Send to multiple comma-separated numbers | V1 | `api.transmitsms.com/send-sms.json` |
| Schedule a future send | V1 | `api.transmitsms.com/send-sms.json` |
| Need delivery/reply callbacks inline | V1 | `api.transmitsms.com/send-sms.json` |

## Message Length

- **GSM encoding** (standard characters): 160 chars per single SMS, 153 chars per part for multi-part
- **Unicode** (emoji, non-Latin characters): 70 chars per single SMS, 67 chars per part for multi-part
- Maximum: 612 alphanumeric characters (V1) or multi-part as needed (V2)

## Important Notes

- Always ask the user which sender number to use if not specified
- The sender must be a number assigned to the account, or an approved alphanumeric sender ID
- Alphanumeric senders cannot receive replies — max 11 characters, no spaces
- If opt-out is required with an alphanumeric sender, use `[unsub-reply-link]` in the message (V1) or `[opt-out-link]` (V2)