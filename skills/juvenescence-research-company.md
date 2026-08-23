---
name: juvenescence-research-company
description: >-
  Answer questions about Juvenescence — its pipeline, science, leadership and portfolio companies —
  by searching and reading the corporate pages and news on juvlabs.com through its content API,
  instead of scraping rendered HTML.
api: juvenescence:juvenescence-search-api
operations:
  - searchContent
  - getPage
  - listPages
  - getPost
  - listTags
  - getOembed
---

# Research Juvenescence from its own content API

Juvenescence Limited is a clinical-stage, AI-enabled drug developer working on medicines that target
core mechanisms of aging. This skill answers questions about it from primary source text, with no
credentials.

Base URL: `https://juvlabs.com/wp-json`

## 1. Start with search, not with a collection walk

`searchContent` (`GET /wp/v2/search?search=<query>`) is the cheapest entry point. It returns a flat
`{ id, title, url, type, subtype }` projection across posts, pages, the portfolio type and taxonomy
terms. A search for `aging` returned 46 hits.

- `type=post` (default) searches content; `type=term` searches taxonomy terms.
- `subtype=page` narrows to corporate pages; `subtype=post` to news.

Then dereference each hit against the collection named by `subtype`.

## 2. Read the corporate pages directly when you know what you want

`getPage` (`GET /wp/v2/pages/{id}`). Stable IDs observed on 2026-08-23 — re-resolve with
`listPages?slug=<slug>` rather than trusting them indefinitely:

| slug | id | holds |
|------|----|-------|
| `our-pipeline` | 3809 | the clinical programme table |
| `science` | 4524 | the mechanism-of-aging thesis |
| `our-approach` | 5786 | strategy and modality mix |
| `our-team` | 4139 | leadership index |
| `contact-us` | 4275 | contact routes |

Leadership biographies are **individual pages**, not a structured person type — `jim-mellon`,
`dr-gregory-bailey-md`, `dr-declan-doogan-md`, `dr-richard-marshall-cbe-md-phd`,
`eileen-jennings-brown`, `professor-eric-verdin`, `professor-lynne-cox-bio`. Resolve them with
`listPages?slug=<slug>`.

## 3. The portfolio graph lives in the tag vocabulary

The `portfolio` custom post type is registered but **empty**. The real portfolio graph is the post-tag
vocabulary. Call `listTags` (`GET /wp/v2/tags?per_page=100`) and read the company names: AgeX, Chrysea
Labs, Evgen Pharma, LyGenesis, MDI Therapeutics, Morphoceuticals, Napa Therapeutics, NetraMark Corp.,
Relation Therapeutics, Selah Therapeutics, Souvien Therapeutics, plus The Buck Institute as a research
collaborator. Only five tags currently carry posts, so treat the vocabulary as the roster and the
`count` as coverage, not as the roster itself.

## 4. Resolve a bare URL

`getOembed` (`GET /oembed/1.0/embed?url=<juvlabs.com URL>`) turns any juvlabs.com URL into its title,
author and embed HTML without fetching and parsing the page.

## Grounding rules

- `content.rendered` is HTML. Strip tags before summarising.
- Quote the `link` field as the citation for anything you assert.
- The `publications` category (id 52) carries paper abstracts with DOIs in the body text — cite the
  DOI, and note that these are the Ro5 AI drug-discovery papers that came with the June 2025
  acquisition.
- This API carries **corporate communications**, not clinical data. It is not a source for trial
  results, dosing or safety information; the press releases summarise those, and the underlying
  registrations live with the competent authorities, not here.

See `data-model/juvenescence-data-model.yml` for the full entity graph.
