---
name: juvenescence-harvest-media
description: >-
  Enumerate the Juvenescence media library — brand marks, leadership portraits, press and conference
  imagery — and resolve each item to a direct source URL and the right size variant.
api: juvenescence:juvenescence-media-api
operations:
  - listMedia
  - getMediaItem
---

# Harvest Juvenescence media

466 media items are anonymously readable at `https://juvlabs.com/wp-json/wp/v2/media`.

## 1. Enumerate

`listMedia` (`GET /wp/v2/media`):

- `media_type=image` or `mime_type=image/png` to filter.
- `per_page=100` is the maximum; page with the `Link` header's `rel="next"`.
- `search=logo` to find brand marks.
- `_fields=id,slug,title,alt_text,mime_type,source_url,media_details` keeps responses small — the
  full `media_details` block is the bulk of the payload.

## 2. Resolve one item

`getMediaItem` (`GET /wp/v2/media/{id}`) returns `source_url` (the original file) and
`media_details.sizes`, a map of named size variants each with its own `source_url`, `width` and
`height`. Pick the variant by dimension rather than resizing the original.

## 3. Attribution and reuse

These are Juvenescence's copyrighted brand and press assets, served for the company's own site. The
API makes them enumerable; it does not license them. Check
`https://juvlabs.com/terms-of-service/` before any redistribution, and do not treat public
readability as permission.

## Failure modes

- `404 rest_post_invalid_id` — the ID does not exist or is not attached. Re-resolve from the collection.
- `400 rest_invalid_param` — almost always `per_page` over 100.
- No rate-limit headers are returned; `robots.txt` asks for a 10-second crawl delay. Pace a 466-item
  walk accordingly.
