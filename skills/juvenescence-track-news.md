---
name: juvenescence-track-news
description: >-
  Track Juvenescence's news stream — press releases, in-the-news pickups, conference reports,
  publications and videos — and pull the full text of any item, using the anonymously readable
  WordPress content API behind juvlabs.com.
api: juvenescence:juvenescence-posts-api
operations:
  - listCategories
  - listPosts
  - getPost
---

# Track Juvenescence news

Juvenescence publishes no developer program. Everything below runs against the site's own
WordPress REST API at `https://juvlabs.com/wp-json`, which needs **no credentials**.

## 1. Resolve the category you care about

Call `listCategories` (`GET /wp/v2/categories`). Five of the seven categories carry posts:

| id | slug | name | posts |
|----|------|------|-------|
| 5 | `press-releases` | Press Releases | 13 |
| 6 | `in-the-news` | In the News | 17 |
| 50 | `juv-on-the-road` | Juv on the Road | 17 |
| 52 | `publications` | Publications | 5 |
| 53 | `videos` | Videos | 5 |

Do not hard-code these IDs — re-resolve them, because `count` moves and terms can be added.

## 2. List the posts

Call `listPosts` (`GET /wp/v2/posts`) with `categories=<id>`. Useful parameters:

- `per_page` — 1 to 100, default 10. Over 100 is a hard `400 rest_invalid_param`, not a clamp.
- `after` / `modified_after` — ISO 8601, for an incremental pull.
- `orderby=date&order=desc` — the default, newest first.
- `_fields=id,date,modified,slug,link,title,categories,tags` — trims a ~10 KB item to a few hundred bytes.

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, and follow the RFC 8288 `Link`
header's `rel="next"` rather than incrementing `page` blindly.

## 3. Fetch the full item

Call `getPost` (`GET /wp/v2/posts/{id}`). `content.rendered` and `excerpt.rendered` are HTML, not
plain text — strip tags before handing them to a model. Add `_embed=true` to inline the featured
media and the category/tag terms in the same round trip.

## 4. For an incremental watch

Persist the highest `modified` you have seen and pass it back as `modified_after`. Responses are
edge-cached for 10 minutes (`cache-control: max-age=600`), so polling faster than that gains
nothing. `robots.txt` asks for a 10-second crawl delay — honour it.

## Conventions and failure modes

- **Errors** are the WordPress envelope, not RFC 9457: `{ code, message, data.status }`. Branch on
  `code`; `message` is localised.
- `rest_invalid_param` (400) and `rest_post_invalid_id` (404) are deterministic — fix the request,
  do not retry.
- **No rate-limit headers** are returned and no limit is documented, so there is no runtime signal
  to back off on. Be conservative.
- **`author` is a dead end**: `/wp/v2/users` returns `403 aios_user_lists_forbidden` on this
  install. Use `_embed` to get the author name from `_links` instead.

See `conventions/juvenescence-conventions.yml` and `errors/juvenescence-problem-types.yml`.
