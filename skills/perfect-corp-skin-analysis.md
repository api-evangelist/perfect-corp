---
name: perfect-corp-skin-analysis
description: >-
  Run a YouCam AI Skin Analysis on a front-facing selfie and turn the returned concern scores into a
  product or routine recommendation. Covers the v2.0/v2.1 split, JSON vs ZIP output, mask URLs, and the
  input-quality failures that dominate this feature.
api: Perfect Corp YouCam AI REST API
operations:
  - POST /s2s/v2.0/file
  - POST /s2s/v2.1/task/skin-analysis
  - GET /s2s/v2.1/task/skin-analysis/{task_id}
  - POST /s2s/v2.0/task/skin-analysis
  - GET /s2s/v2.0/task/skin-analysis/{task_id}
generated: '2026-09-02'
method: generated
source: openapi/perfect-corp-ai_skin_analysis-openapi.yml, https://docs.perfectcorp.com/reference/ai_skin_analysis
---

# AI Skin Analysis

Scores facial skin concerns — wrinkles, pores, acne, oiliness, dark circles, texture, hydration and
more — from a single front-facing selfie, and returns per-concern mask images alongside the scores.

## Which version to call

The contract publishes **two path versions side by side**, tagged `V2.0` and `V2.1`:

- `POST /s2s/v2.0/task/skin-analysis` + `GET /s2s/v2.0/task/skin-analysis/{task_id}`
- `POST /s2s/v2.1/task/skin-analysis` + `GET /s2s/v2.1/task/skin-analysis/{task_id}`

v2.1 (release v1.12.1, 2026-06-09) carries updated engines and raises the maximum skincare output
resolution to 2560 px with automatic input resizing. Prefer v2.1 for new work; v2.0 remains published
and no deprecation date is stated for it.

## Steps

1. Upload the selfie with `POST /s2s/v2.0/file` and complete the second-phase upload to the returned
   `requests[].url` (see the `perfect-corp-run-ai-task` skill — the File API alone does not upload).
2. `POST /s2s/v2.1/task/skin-analysis` with the `file_id` and the concerns you want scored. Choose the
   output shape with `format`:
   - `format=json` — scores and mask URLs directly in the response body. Use this when an agent will
     act on the numbers.
   - `format=zip` — a packaged archive of result assets behind a single URL.
3. Poll `GET /s2s/v2.1/task/skin-analysis/{task_id}` on the returned `polling_interval` until
   `task_status` is `success` or `error`.
4. Read the results and fetch any asset URLs **within 2 hours**.

## Reading the result

With `format=json`, `results.output[]` carries one entry per concern and region:

```json
{ "type": "hd_wrinkle", "region": "forehead", "raw_score": 20.1, "ui_score": 20,
  "mask_urls": ["https://…/hd_wrinkle_forehead.jpg"] }
```

- `type` — the concern (HD and SD concern families exist; HD types are prefixed `hd_`).
- `region` — `whole` plus per-region breakdowns such as `forehead`.
- `raw_score` — the model's continuous score.
- `ui_score` — the rounded value intended for display. **Show `ui_score` to a person; reason over
  `raw_score`.**
- `mask_urls` — overlay images highlighting where the concern was detected.

These scores are Perfect Corp's own scale, not a clinical standard. The one feature in the platform
that emits a recognised third-party vocabulary is **AI Fitzpatrick Skin Type Analysis**
(`POST /s2s/v2.0/task/fitzpatrick-scale-analyzer`), which returns the standard Fitzpatrick I–VI
phototype — call that alongside skin analysis when a downstream system already speaks Fitzpatrick.

## Input failures to expect

Skin analysis is the feature most sensitive to input quality. All of these arrive as **HTTP 200** with
`task_status: "error"`:

| `error_code` | Fix |
|---|---|
| `error_no_face` | No face detected — need a front-facing selfie |
| `error_multiple_people` | Crop to one subject |
| `error_large_face_angle` | Face is turned too far; ask for a straight-on shot |
| `error_pose` / `error_face_parsing` | Detection failed; retry with better lighting and framing |
| `error_nsfw_content_detected` | Input rejected |
| `exceed_max_filesize` / `error_exceed_max_image_size` | Images cap at 10 MB |
| `error_decode_image` | Unsupported or corrupt encoding |

Validate framing before spending a unit: a rejected task still costs one.

## Cost and cleanup

Check the unit price with `GET /s2s/v2.0/credit/feature-cost` before a batch, and your balance with
`GET /s2s/v1.0/client/credit`. When you are finished, `POST /s2s/v2.0/task/delete` removes the task
with its selfie and all generated masks — worth doing deliberately here, because the input is a
photograph of a person's face. Everything is auto-deleted after 30 days regardless.
