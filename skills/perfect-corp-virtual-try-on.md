---
name: perfect-corp-virtual-try-on
description: >-
  Drive the YouCam virtual try-on family — clothing, shoes, hats, scarves, bags, and jewellery
  (bracelet, earrings, necklace, ring, watch) — from a product image and a customer photo. Covers the
  two-input pattern, the template catalogue, and the required-parameter traps.
api: Perfect Corp YouCam AI REST API
operations:
  - POST /s2s/v2.0/file
  - POST /s2s/v2.0/task/2d-vto/ring
  - GET /s2s/v2.0/task/2d-vto/ring/{task_id}
  - POST /s2s/v2.0/file/2d-vto/ring
generated: '2026-09-02'
method: generated
source: >-
  openapi/perfect-corp-ring_vto-openapi.yml, perfect-corp-ai_clothes-openapi.yml,
  perfect-corp-ai_shoes-openapi.yml, perfect-corp-ai_hat-openapi.yml, perfect-corp-ai_scarf-openapi.yml,
  perfect-corp-ai_bag-openapi.yml, perfect-corp-ai_bracelet-openapi.yml, perfect-corp-ai_earrings-openapi.yml,
  perfect-corp-ai_necklace-openapi.yml, perfect-corp-ai_watch-openapi.yml, perfect-corp-ai_fabric-openapi.yml
---

# Virtual try-on

The try-on family renders a product onto a customer photo from a single 2D product image. It splits
into two shapes.

## Apparel and accessories

`ai_clothes`, `ai_shoes`, `ai_hat`, `ai_scarf`, `ai_bag`, `ai_fabric` — one task path each under
`/s2s/v2.0/task/…`. Upload the model photo and the garment image with the File API, then create the
task with both file ids.

- **AI Clothes** takes a category selector: `Full Body`, `Upper Body`, `Lower Body`, `Shoes`,
  `Outerwear`, or `Auto` for AI-assisted styling. Outerwear (jackets and vests) was added in v1.14.1.
- **AI Shoes, AI Hat, AI Scarf and AI Bag require an explicit `gender` parameter.** This is the most
  common `InvalidParameters` cause on these four — the MCP tool descriptions call it out specifically.
- **AI Fabric** applies a texture from a template rather than a supplied image. List valid templates
  from its template operation before creating the task.

## Jewellery and watches

`ring_vto`, `ai_bracelet`, `ai_earrings`, `ai_necklace`, `ai_watch` — these sit under
`/s2s/v2.0/task/2d-vto/{item}` and several publish an **item-specific file endpoint** alongside the
generic one, e.g. `POST /s2s/v2.0/file/2d-vto/ring`. Use the item-specific upload where the contract
declares one; it registers the product image in the form the renderer expects.

Placement controls are per item — finger selection and knuckle positioning for rings, ear-side
selection and occlusion for earrings, neck and clavicle tracking for necklaces, strap alignment and
case edge detection for watches — and are declared in each feature's request schema. Read the schema
from the matching file in `openapi/` rather than assuming they are shared: they are not.

## Working through templates

Style-driven features expose `GET /s2s/v2.x/task/template/{feature}`. Paginate with `page_size`
(1–20, default 20) and `starting_token`, following `next_token` until it is absent. Cache the ids —
a stale or invented one returns `InvalidStyle` or `InvalidStyleGroup`, and the failed create still
costs a unit.

Note that Perfect Corp addresses products by its **own internal style ids**. No GTIN or other retail
product identifier is accepted anywhere in the try-on contracts, so a retailer has to maintain a
bilateral mapping between its catalogue and Perfect Corp's ids.

## Then

Poll, read the result, and clean up exactly as in `perfect-corp-run-ai-task`. Input failures here are
usually `error_no_face`, `error_pose`, `error_no_shoulder` (shoulders not visible — common on
necklace and scarf try-on) or `error_unsupport_ratio`.

## Through MCP instead

All of these are available as tools on `https://mcp-api-01.makeupar.com/mcp/fashion` with the same
parameters — `AI-Clothes-Virtual-Try-On`, `AI-Shoes-Virtual-Try-On`, `AI-Ring-Virtual-Try-On` and so
on. See `mcp/perfect-corp-tool-crosswalk.yml` for the tool-to-operation binding. One thing to know
before choosing MCP: task deletion is exposed by **no** MCP tool, so an agent working through MCP alone
has no way to remove the customer photos it uploaded.
