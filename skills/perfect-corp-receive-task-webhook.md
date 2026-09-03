---
name: perfect-corp-receive-task-webhook
description: >-
  Receive and verify YouCam task-completion webhooks instead of polling. Covers the Standard Webhooks
  signature scheme Perfect Corp uses, the header set, deduplication, and what the notification does
  and does not contain.
api: Perfect Corp YouCam AI REST API
operations:
  - GET /s2s/v2.0/task/{feature}/{task_id}
generated: '2026-09-02'
method: generated
source: https://docs.perfectcorp.com/develop/webhook
---

# Receive YouCam task-completion webhooks

Webhooks replace the polling loop for long-running tasks. Perfect Corp implements the
[Standard Webhooks](https://github.com/standard-webhooks/standard-webhooks/blob/main/spec/standard-webhooks.md)
specification, so an off-the-shelf Standard Webhooks library verifies these correctly and you should
prefer one over hand-rolling the HMAC.

## 1. Register the endpoint

Endpoints are created in the API Console at `https://yce.makeupar.com/api-console/en/webhook/`. There
is **no REST operation** to create, list or delete a webhook endpoint — an agent cannot manage its own
subscriptions. Up to 10 endpoints may be configured concurrently.

Your endpoint must accept `POST` over HTTPS. Store the `whsec_`-prefixed secret the console shows you;
it is displayed for you to copy, not retrievable via API.

## 2. Verify the signature before trusting anything

Three headers arrive with every delivery:

| Header | Meaning |
|---|---|
| `webhook-id` | Unique delivery id, **stable across retries** |
| `webhook-timestamp` | Unix epoch seconds when the webhook was sent |
| `webhook-signature` | `v1,<base64 HMAC-SHA256>` |

To verify by hand: strip the `whsec_` prefix from your secret, base64-decode the remainder to get the
key bytes, then HMAC-SHA256 the signed content

```
{webhook-id}.{webhook-timestamp}.{raw-minified-json-body}
```

and compare, in constant time, against the base64 value after `v1,`. `v1` is currently the only
supported signature version. Sign the **raw** body you received — re-serializing the JSON changes the
bytes and the signature will not match.

## 3. Handle the payload

```json
{
  "created_at": 1761112848,
  "data": { "task_id": "…", "task_status": "success" }
}
```

That is the whole notification. `task_status` is `success` or `error`, and there is **no result in the
payload** — call `GET /s2s/v2.0/task/{feature}/{task_id}` to fetch it, and remember the download URL
you get back is valid for 2 hours.

There is also no event `type` or name field: one shape, one meaning. If you need to route by feature,
carry your own mapping from `task_id` to the feature you created it with, because the notification does
not tell you which endpoint produced it.

## 4. Deduplicate on `webhook-id`

`webhook-id` is documented as consistent across retries and is the value to use for idempotency. Record
processed ids and drop repeats — Perfect Corp publishes no retry schedule, timeout, or redelivery API,
so you cannot predict how many times a delivery may arrive.

## Testing

Perfect Corp ships no simulator of its own and points at the Standard Webhooks project's public
simulator at `https://www.standardwebhooks.com/simulate` for payload testing.
