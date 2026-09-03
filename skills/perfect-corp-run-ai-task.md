---
name: perfect-corp-run-ai-task
description: >-
  Run any of the 60+ YouCam AI features on an image or video end to end — upload, create the task,
  poll to a terminal state, and retrieve the result before its URL expires. This is the one workflow
  every YouCam feature shares; learn it once and every feature works the same way.
api: Perfect Corp YouCam AI REST API
base_url: https://yce-api-01.makeupar.com
operations:
  - POST /s2s/v2.0/file
  - POST /s2s/v2.0/task/{feature}
  - GET /s2s/v2.0/task/{feature}/{task_id}
  - POST /s2s/v2.0/task/delete
generated: '2026-09-02'
method: generated
source: >-
  openapi/perfect-corp-file-openapi.yml, openapi/perfect-corp-task_management-openapi.yml, the 62
  per-feature contracts in openapi/, and https://docs.perfectcorp.com/develop/quick_start_guide
---

# Run a YouCam AI task

Every YouCam feature — skin analysis, virtual try-on, image generation, video enhancement — is the
same asynchronous three-step pattern. The published contracts carry **no `operationId`**, so address
operations by method and path.

## 0. Authenticate

Send your API key as a bearer token on every request:

```
Authorization: Bearer YOUR_API_KEY
```

Two documented mistakes that both return 401: omitting the literal `Bearer ` prefix, and leaving the
angle brackets in `Bearer <YOUR_API_KEY>`.

## 1. Upload the input file — two phases, not one

`POST /s2s/v2.0/file` does **not** upload anything. It reserves a slot and hands you a destination.

```json
{ "files": [ { "content_type": "image/jpg", "file_name": "selfie.jpg", "file_size": 50000 } ] }
```

The response gives you, per file, a `file_id` and a `requests[]` entry with `url`, `method` and
`headers`. **You must then send the bytes yourself** to that URL with that method and those headers.
Skipping this is the documented cause of a 404 from the AI operations.

Limits: 10 MB for images, 100 MB for video.

If the source is already a public URL you can skip this step entirely and pass `src_file_url` on
task creation instead of `src_file_id`.

## 2. Create the task

`POST /s2s/v2.0/task/{feature}` with the `file_id` and the feature's parameters. Read the exact
request schema from that feature's contract in `openapi/` — the shared envelope is
`BasicRunTaskV2` / `BasicRunTaskV2SrcFileId`, and each feature adds its own fields.

Some features require a detection pass first: `POST /s2s/v2.0/task/{feature}/pre-process`, then feed
its output to the main task. This applies to AI Face Lift, AI Face Reshape, AI Body Reshape, AI Teeth
Whitening, AI Face Swap and the Fitzpatrick analyzer.

Features with a style catalogue (hairstyle, bangs, beard, hair extension, hair volume, look, fabric,
avatar, studio, headshot, video style transfer) take a style id. List valid ids from
`GET /s2s/v2.0/task/template/{feature}` — paginate with `page_size` (max 20) and `starting_token`,
following `next_token`. A wrong id returns `InvalidStyle` or `InvalidStyleGroup`.

The response returns a `task_id` and a `polling_interval`.

## 3. Poll to a terminal state

`GET /s2s/v2.0/task/{feature}/{task_id}` on the returned `polling_interval` until `task_status` is
`success` or `error`.

**Check the body, not just the status line.** An engine failure arrives as **HTTP 200** with
`task_status: "error"` and an `error_code` such as `error_no_face`, `error_nsfw_content_detected`,
`error_multiple_people` or `error_large_face_angle`. See `errors/perfect-corp-problem-types.yml` for
the full vocabulary and what each one means for the input.

If you stop polling and let the task time out, a later status check returns `InvalidTaskId`.

Prefer a webhook to a polling loop for long-running video work — see the
`perfect-corp-receive-task-webhook` skill.

## 4. Retrieve the result within two hours

A success payload carries a download URL. **That URL is valid for 2 hours.** After it expires, re-query
the `task_id` for a fresh link; the `task_id` itself stays valid for 30 days.

Analysis features often accept `format=json` or `format=zip` — JSON for scores you will act on, ZIP
for the packaged mask images.

## 5. Clean up when you are done

`POST /s2s/v2.0/task/delete` removes a finished task **and its input files and generated outputs**.
This is the only reversal operation the API publishes. It reverses storage, not spend: units consumed
are not returned, and there is no refund operation.

Everything is auto-deleted after 30 days regardless.

## Rules to hold on to

- **Retries are not safe.** There is no `Idempotency-Key` and no idempotency semantics. A retried task
  creation charges units again and produces a second `task_id`. If a create times out, poll the
  original `task_id` rather than re-posting.
- **You cannot cancel.** No abort or stop operation exists. Once created, a task runs to a terminal
  state.
- **Check the price first.** `GET /s2s/v2.0/credit/feature-cost` returns the unit cost of every
  feature; `GET /s2s/v1.0/client/credit` returns your balance. Running out mid-workflow surfaces as
  `CreditInsufficiency` on the next create.
- **Pace yourself.** 250 requests per 300 seconds per IP *and* per token; both must hold. Aim for
  ~5 requests/second. Exhaustion is a `429` with **no** `Retry-After` and no rate-limit headers — back
  off on your own clock.
- **Do not parse `task_id` as a JavaScript number.** IDs exceed 2^53-1 and `JSON.parse` silently
  rounds them, which makes distinct tasks look identical. Use a big-integer JSON parser.
- Quote the `x-request-id` response header when you contact support.
