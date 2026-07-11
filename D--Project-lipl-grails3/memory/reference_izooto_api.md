---
name: reference-izooto-api
description: "iZooto push notification API reference — endpoints, auth, payload structure, targeting options, and template/campaign findings (researched 2026-05-29)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f79d8d7e-c4a9-41ef-8681-9bd40cfa35c7
---

## API Endpoint

`POST https://apis.izooto.com/v1/notifications`

Note: host is `apis.izooto.com` (not `api.izooto.com`).

## Authentication

Header: `Authentication-Token: {api_key}`  
(NOT `Authorization: bearer ...` — different from OneSignal)

API key obtained from iZooto Panel → Settings → General → Keys → API Key (requested, delivered over email).

## Targeting Modes

Three modes via the `target` object:

| Mode | `target` value | Notes |
|---|---|---|
| All subscribers | `{"type": "all"}` | Fully documented |
| Audience/segment | `{"type": "audience", "value": 1381}` | Use GET `/v1/audience` to list segment IDs |
| Individual subscriber | `{"type": "subscriber", "value": "{subid}"}` | Referenced in iZooto blog but wiki.izooto.com/api/ is 404 — **confirm with iZooto support before implementing** |

The `subid` individual targeting API likely exists but may be enterprise-gated or undocumented publicly. This is critical for transactional notifications (order events, wallet, etc.) — must be confirmed before implementation.

## Full Request Body (Push to All / Push to Audience)

```json
{
  "campaign_name": "string (required)",
  "title": "string (required)",
  "message": "string (required)",
  "icon_url": "https://...",
  "badge_icon_url": "https://...",
  "banner_url": "https://...",
  "landing_url": "https://...",
  "actions": [
    { "text": "Buy Now", "url": "https://...", "icon": "https://..." }
  ],
  "utm_source": "izooto",
  "utm_medium": "push_notifications",
  "utm_campaign": "promotion",
  "utm_term": "",
  "utm_content": "",
  "ttl": 86400,
  "override_tag": "string (web only)",
  "req_interaction": true,
  "additional_params": { "key": "value" },
  "open_in_app": true,
  "target": {
    "type": "all"
  }
}
```

- `additional_params` — Android/iOS only; key-value custom data (equivalent to OneSignal's `data`/`custom_data`)
- `req_interaction` — web push only
- `open_in_app` — Android/iOS only

## No Template/Campaign ID — Critical Finding

iZooto has NO equivalent to OneSignal's `template_id`. Two different things are called "templates":

1. **Visual templates** (NewsRoom, Timer, Default) — layout choices in the dashboard UI only; cannot be referenced by API
2. **Campaigns** — each API call creates a new one-shot campaign via `campaign_name`; you cannot pre-create a campaign and trigger it by ID

**Impact on OneSignal → iZooto migration:** The `OnesignalTemplate` domain (storing title/body + `#key#` substitution logic) must be kept. iZooto requires full title + body inline on every API call. The `templateId` field currently sent to OneSignal has no equivalent — send `notification.title` and `notification.description` directly.

## Key Differences from OneSignal

| Aspect | OneSignal | iZooto |
|---|---|---|
| Auth header | `Authorization: {REST_API_KEY}` | `Authentication-Token: {api_key}` |
| App identifier | `app_id` in payload | Not needed |
| Per-user targeting | `include_aliases: {external_id: [userId]}` | `subid` (confirm with support) |
| Template reuse | `template_id` UUID → dashboard template | No equivalent; content always inline |
| Segment targeting | `included_segments: ["segment"]` | `target: {type:"audience", value: id}` |
| Custom data | `data: {...}` or `custom_data: {...}` | `additional_params: {...}` |
| Deep link | `app_url` | `landing_url` |

## Sources

- https://help.izooto.com/docs/introduction-to-push-apis
- https://help.izooto.com/docs/push-to-all
- https://help.izooto.com/docs/push-to-an-audience
- https://help.izooto.com/docs/android-push-notifications-templates
- https://izooto.com/blog/personalise-web-push-notifications-with-izooto-api
