---
name: create-list
description: Create contact lists and add recipients for messaging campaigns.
---

# Create Contact Lists & Add Contacts

You can create contact lists and add contacts using the Kudosity V1 API.

## Authentication

All list operations use **V1 Basic Auth**:
- Header: `Authorization: Basic {credentials}`
- Credentials: base64 encode `{KUDOSITY_API_KEY}:{KUDOSITY_API_SECRET}`
- The user must have `KUDOSITY_API_KEY` and `KUDOSITY_API_SECRET` environment variables set
- If the user hasn't configured credentials, direct them to run `/setup`

## API Details

- **Base URL**: `https://api.transmitsms.com`
- **Content-Type**: `application/x-www-form-urlencoded`
- All requests use `curl` commands with Basic Auth

## Step 1: Create a List

**Endpoint**: `POST /add-list.json`

Required parameters:
- `name` (string): A unique name for the list

Optional parameters:
- `field_1` through `field_10`: Custom field names (firstname and lastname are created by default)

Example curl command:
```bash
curl -s -X POST "https://api.transmitsms.com/add-list.json" \
  -u "${KUDOSITY_API_KEY}:${KUDOSITY_API_SECRET}" \
  -d "name=My Campaign List&field_1=email&field_2=postcode"
```

The response returns the list `id` — store this for adding contacts.

## Step 2: Add Contacts to the List

**Endpoint**: `POST /add-to-list.json`

Required parameters:
- `list_id` (integer): The list ID returned from Step 1
- `msisdn` (string): Mobile number in E.164 international format

Optional parameters:
- `countrycode` (string): 2-letter ISO country code (e.g., `AU`, `US`, `GB`, `NZ`) — auto-formats local numbers to international
- `first_name` (string): Contact's first name
- `last_name` (string): Contact's last name
- `field_1` through `field_10`: Values for custom fields defined on the list

Example curl command:
```bash
curl -s -X POST "https://api.transmitsms.com/add-to-list.json" \
  -u "${KUDOSITY_API_KEY}:${KUDOSITY_API_SECRET}" \
  -d "list_id=4213644&msisdn=0491570156&countrycode=AU&first_name=John&last_name=Doe&field_1=john@example.com"
```

## Important Notes

- If a contact already exists in the list, it will be ignored (not updated). Use `/edit-list-member.json` to update existing contacts.
- Use the `countrycode` parameter to avoid formatting issues with local numbers.
- The list can hold up to 10 custom fields beyond the default firstname and lastname.
- Country code examples: Australia=`AU`, New Zealand=`NZ`, United Kingdom=`GB`, United States=`US`
- Number format examples: AU local `0491570156` → international `61491570156`