# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The Sol Design Archive (soldesignarchive.com) — a Jekyll site cataloguing
Portuguese and international graphic design artefacts, mostly mid-century book
covers, posters and ephemera. Around 930 artefacts, roughly half published.

Records are usually written by LUNA, the local cataloguing tool in
`../luna` (see that repo's CLAUDE.md), not by hand. LUNA reads and writes these
files directly, so **changing the shape of front matter here means changing
LUNA too.**

This is a real archive of real objects. Treat the data as records, not as
sample content: an origin, a publisher or a date is a claim about a physical
thing, and normalising it is an editorial act. Fix genuine inconsistencies;
ask before merging entities.

## Running it

```bash
bundle exec jekyll serve -i -u    # http://127.0.0.1:4000
```

`-u` is `--unpublished`, so `published: false` records render locally. They are
excluded from a real build — check anything that depends on publication state
with a plain `bundle exec jekyll build`.

Three environment traps, all of which have cost a session before:

- **`GEM_HOME` must be set.** The gems live in `~/.gem/ruby/3.1.3`, but Ruby's
  default user dir is `~/.gem/ruby/3.1.0`. Ricardo's shell exports it; a
  process spawned without it resolves nothing and reports every gem missing.
- **A UTF-8 locale is required.** Filenames like `Plano de Educação Popular.md`
  make Jekyll's URL encoder raise `"\xC3" from ASCII-8BIT to UTF-8` under the
  C locale. Set `LANG=en_US.UTF-8`.
- **`EADDRINUSE` on 4000 means a server is already running**, very often one
  LUNA started. `lsof -nP -iTCP:4000 -sTCP:LISTEN` names it. Don't kill a
  process without checking whose it is.

**Incremental builds go stale.** `-i` regenerates only what changed, and it
does not understand that renaming a publisher invalidates the footer of every
page. Verify anything site-wide against a clean `jekyll build --destination`
to a scratch directory, never against `_site`.

## Deployment

GitHub Actions (`.github/workflows/pages.yml`), building with this repo's own
Gemfile on Jekyll 4.2.1. Pages' built-in "legacy" builder runs the
`github-pages` gem, which pins Jekyll 3.10 and admits only its own plugin list
— it could not resolve this Gemfile, and every automatic build from April to
September 2026 failed.

Consequences worth knowing before touching build config:

- `Gemfile.lock` must list `x86_64-linux` as well as `arm64-darwin-21`. A
  deployment-mode install refuses to run on a platform the lock doesn't name,
  so a Mac-only lock stops CI before Jekyll starts. Use
  `bundle lock --add-platform x86_64-linux`.
- Plugins are no longer restricted to the github-pages allowlist, but a plugin
  listed in `_config.yml` that isn't in the Gemfile is a hard build error.
- `CNAME` in the repo root carries the custom domain.

## How the collections fit together

`_artefacts` is the substance. Every other collection — `_authors`,
`_publishers`, `_origins`, `_formats`, `_disciplines`, `_tags`, `_years` — is a
category page listing the artefacts that name it.

An artefact names a category **by that file's front-matter `name`**, and the
category layout finds its artefacts with an exact string match:

```liquid
{% assign filtered_artefacts = pub_artefacts | where: 'origin', page.name %}
```

Three things follow, and each has produced a real bug:

**The `name` is authoritative, the filename is not.** They routinely differ,
because filenames can't hold every character: `_publishers/Alfred A-dot- Knopf.md`
declares `Alfred A. Knopf`, and `SPN-barra-SNI-barra-SEIT.md` declares
`SPN/SNI/SEIT`. Templates reverse the substitution (`-dot-` → `.`,
`-barra-` → `/`) when displaying a name.

**Never build a category URL by guessing it from the name.** The permalink is
`/<collection>/:path/` — the *filename*. Resolve the document and use its own
`url`:

```liquid
{% assign origin_page = site.origins | where: 'name', origin | first %}
href="{{ origin_page.url | default: origin_fallback }}"
```

Guessing produced `/origins/United-states/` for a page served from
`/origins/united-states/`, which worked only because macOS filesystems are
case-insensitive and would have 404'd in production.

**A misspelt value fails silently.** It doesn't error; the artefact simply
never appears on that category page, and the category link goes nowhere.
LUNA's Overview reports these as "referenced with no file" and "nothing points
to it" — that dashboard is the fastest way to audit the collection.

`published: false` on a category file keeps its page off the site. The
convention is that a category with no published artefact behind it isn't
published either.

## Artefact front matter

```yaml
ref_group: "030"        # quoted - leading zeros matter
ref_id: "0229"          # quoted, four digits
title: 3 Tragedies
author_name: Alvin Lustig
publisher: "New Directions"
year: y1955             # "y" prefix, or "unknown-date"
decade: 1950s
origin: United States
formats: [book-cover, book]
disciplines: [graphic-design, typography, illustration, lettering]
tags: [fiction, theatre, "Garcia Lorca"]
layout: artefact
status: complete        # offline/onboarding/scan/production/publish/complete/wip/hidden
published: true
image_count:
date_added: 2023-03-10
```

Serialisation is inconsistent across the collection by history, not by design:
list fields appear as bare scalars, `["quoted arrays"]` and block lists in
roughly equal measure. All three parse identically. Match the file you are
editing rather than reformatting it, and never rewrite front matter wholesale
— a YAML round-trip reflows every unrelated field.

Images are `images/sol-{ref_group}-{ref_id}-{slug}.jpg` with thumbnails at
`images/sm/…-sm.jpg`. An artefact can have several (`…-2.jpg`), so the
thumbnail count exceeds the artefact count.

`pages/back-office/` holds the site's own admin views — `db.html`,
`db-status.html`, `db-refs.html`, `db-authors.html`, `staging.html`. LUNA's
Database panel is a rebuild of `db.html`.

## Gotchas

`_site`, `.jekyll-cache` and `.jekyll-metadata` are gitignored; `.DS_Store` is
tracked and noisy — leave it out of commits.

`.forestry/` and `.cloudcannon/` are legacy CMS configs, not in use.

Author records are mostly skeletal: of 209 files, 4 have an image and about
half have any prose. `rel_authors` / `rel_publishers` / `rel_tags` are declared
almost everywhere and filled almost nowhere; `_layouts/author.html` reads those
names, so a file spelling them `related_*` renders nothing.
