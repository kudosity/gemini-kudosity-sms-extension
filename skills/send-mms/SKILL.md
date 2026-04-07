---
name: send-mms
description: Send MMS messages with images, GIFs, video, or audio.
---

# Send MMS Messages

MMS messages are sent via the Kudosity V2 API and support images, GIFs, videos, and audio attachments.

## Authentication

- Header: `x-api-key: {KUDOSITY_API_KEY}`
- If the user hasn't configured credentials, direct them to run `/setup`

## API Details

- **Endpoint**: `POST https://api.transmitmessage.com/v2/mms`
- **Content-Type**: `application/json`
- All requests use `curl` commands

## Parameters

Required:
- `sender` (string): The sender number (must be assigned to the account)
- `recipient` (string): Destination number (local or E.164 international format)
- `content_urls` (array of strings): URLs to the media files to attach

Optional:
- `subject` (string): Message subject — max 20 characters, ASCII only
- `message` (string): Body text — max 1,000 characters. Use `[opt-out-link]` for opt-out if required
- `message_ref` (string, max 500 chars): Your reference ID, returned in webhooks
- `track_links` (boolean): Enable link tracking with shortened URLs

## Example curl Command

```bash
curl -s -X POST "https://api.transmitmessage.com/v2/mms" \
  -H "x-api-key: ${KUDOSITY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "New Arrival",
    "message": "Check out our latest product!",
    "sender": "61481074185",
    "recipient": "61435795809",
    "content_urls": ["https://example.com/product.jpg"]
  }'
```

## Supported Media

| Format | Type | Media Type | Limit |
|--------|------|------------|-------|
| JPEG, JPG | Image | image/jpeg | Up to 400 KB |
| GIF (incl. animated) | Image | image/gif | Up to 400 KB |
| PNG | Image | image/png | Up to 400 KB |
| BMP | Image | image/bmp | Up to 400 KB |
| MP3 | Audio | audio/mpeg | Up to 400 KB |
| WAV | Audio | audio/wav | Up to 400 KB |
| MPEG, MPG, MP4 | Video | video/mpeg4 | Up to 400 KB |

## Recommended Image Dimensions

For best visibility on mobile devices, use one of these recommended sizes:
- **480px × 480px**
- **640px × 640px**

> Note: Some older handset models have limited support for certain image formats, so results may vary.

## Important Notes

- **Only one media file can be attached at a time**, up to 400 KB
- Media URLs **must be absolute paths** (e.g., `https://example.com/image.jpg`), not relative paths
- **Currently MMS can only be sent to Australia**
- The `subject` field is limited to 20 characters and ASCII characters only
- Always ask the user which sender number to use if not specified
- The sender must be a number assigned to their account