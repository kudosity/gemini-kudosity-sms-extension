---
name: setup-webhook
description: Create and manage webhooks on the Kudosity platform. Use when the user wants to receive notifications for SMS delivery status, inbound messages, MMS events, link hits, or opt-outs.
---

# Create & Manage Webhooks

Webhooks allow users to receive HTTP POST notifications when events occur, such as SMS delivery status changes, inbound messages, link clicks, and opt-outs.

## Authentication

- Header: `x-api-key: {KUDOSITY_API_KEY}`
- If the user hasn't configured credentials, direct them to run `/setup`

## API Details

- **Base URL**: `https://api.transmitmessage.com`
- **Content-Type**: `application/json`
- All requests use `curl` commands

## Create a Webhook

**Endpoint**: `POST /v2/webhook`

Required parameters:
- `name` (string): Webhook name, 2-100 characters
- `url` (string): HTTPS URL that accepts JSON-encoded POST requests

Optional parameters:
- `filter` (object): Filter which events trigger the webhook
  - `event_type` (array): Types of events to subscribe to
  - `sender` (array): Filter by sender number
  - `status` (array): Filter by message status (for status events only)
  - `message_ref` (array): Filter by message reference
  - `campaign_id` (array): Filter by campaign ID
- `rate_limit` (integer): Max requests per second to your URL (max 10,000, 0 = system default)

### Available Event Types

| Event Type | Description |
|-----------|-------------|
| `SMS_STATUS` | SMS delivery status changes (SENT, ACCEPTED, DELIVERED, FAILED, SOFT_BOUNCE, HARD_BOUNCE, OTHER) |
| `SMS_INBOUND` | Inbound SMS received from a recipient |
| `MMS_STATUS` | MMS status changes (SENT, FAILED) |
| `MMS_INBOUND` | Inbound MMS received from a recipient |
| `LINK_HIT` | Recipient clicked a tracked link |
| `OPT_OUT` | Recipient opted out via link or STOP message |

### Filter Logic

- **Within a filter array** (e.g., multiple senders): conditions are combined with **OR**
- **Between different filters** (e.g., sender AND status): conditions are combined with **AND**

### Example: Create a webhook for SMS delivery status and inbound messages

```bash
curl -s -X POST "https://api.transmitmessage.com/v2/webhook" \
  -H "x-api-key: ${KUDOSITY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "SMS Events",
    "url": "https://myapp.com/webhooks/sms",
    "filter": {"event_type": ["SMS_STATUS", "SMS_INBOUND"]}
  }'
```

### Example: Create a webhook filtered to specific statuses

```bash
curl -s -X POST "https://api.transmitmessage.com/v2/webhook" \
  -H "x-api-key: ${KUDOSITY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Failed SMS Alerts",
    "url": "https://myapp.com/webhooks/failures",
    "filter": {"event_type": ["SMS_STATUS"], "status": ["FAILED", "HARD_BOUNCE"]}
  }'
```

## List All Webhooks

**Endpoint**: `GET /v2/webhook`

```bash
curl -s "https://api.transmitmessage.com/v2/webhook" \
  -H "x-api-key: ${KUDOSITY_API_KEY}"
```

## Webhook Payload Examples

### SMS Status Event
```json
{
  "event_type": "SMS_STATUS",
  "timestamp": "2021-05-06T05:16:07Z",
  "status": {
    "type": "SMS",
    "id": "a51ebe4e-a412-440e-a8d9-464e68a521cc",
    "message_ref": "ncc5009d",
    "recipient": "61435790000",
    "sender": "61481074190",
    "status": "DELIVERED"
  }
}
```

### SMS Inbound Event
```json
{
  "event_type": "SMS_INBOUND",
  "timestamp": "2021-05-06T05:16:33Z",
  "mo": {
    "type": "SMS",
    "id": "alss-2way-605b31c7-d2c49104",
    "message": "Yes, I'm interested",
    "recipient": "61481074190",
    "sender": "61435790000"
  }
}
```

### Opt-Out Event
```json
{
  "event_type": "OPT_OUT",
  "timestamp": "2021-05-06T05:16:20Z",
  "opt_out": {
    "source": "SMS_INBOUND",
    "source_message": {
      "type": "SMS",
      "id": "a51ebe4e-a412-440e-a8d9-464e68a521cc",
      "message": "Your promo message with opt-out link",
      "message_ref": "ncc5009d",
      "recipient": "61435790000",
      "sender": "61481074190"
    }
  }
}
```

## Important Notes

- Webhook URLs **must use HTTPS**
- The `event_type` field at the top level of the request body is **deprecated** — always use `filter.event_type` instead
- Rate limit defaults to the system limit if set to 0 or not specified
- Multiple status events can be triggered for a single message
- For inbound events, the `sender` filter is applied to the `recipient` field (the number that received the inbound message)
- Always confirm with the user which event types they need before creating the webhook