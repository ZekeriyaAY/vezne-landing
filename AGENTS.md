# AGENTS.md — vezne-landing

Marketing/landing site for **Vezne**, a personal finance tracker.
Companion apps are `../vezne-api/` (FastAPI backend) and
`../vezne-web/` (React SPA, the actual product at `app.vezne.app`) —
both private repos. This repo is the public-facing site at the apex
domain.

This file guides any AI coding agent working in this repo.

## Public repository — handle with care

Unlike its sibling repos, **this repo is published publicly**
(github.com — public visibility). Nothing committed here should include:

- Personal data — real names, emails, or other user/customer PII
  (this includes example data, screenshots, and testimonials without
  explicit consent).
- Credentials, API keys, tokens, internal hostnames/IPs.
- Unpublished business detail — vendor pricing/fee rates, contract
  terms, legal-risk notes, anything from the workspace-root
  `LAUNCH-PLAN.md` that isn't meant to be public yet.

Finished marketing copy, shipped product features, and finalized
public pricing are exactly what belongs here — that's the site's job.
When in doubt about whether something is safe to commit, ask.

## Project context

Static Astro site, no framework runtime. `src/pages/index.astro` is
the real landing page (hero, how-it-works, feature overview, supported
import formats) linking out to `app.vezne.app`. `/terms` and
`/privacy` (`src/layouts/LegalLayout.astro`) are live with draft,
non-lawyer-reviewed legal text migrated from `vezne-web`. `/faq` is
live with grouped `<details>`/`<summary>` Q&A — no client-side JS.
All pages share `src/layouts/SiteLayout.astro` (header, footer, base
colors/typography); `LegalLayout.astro` builds on top of it for
long-form legal content. Pricing/blog content is still pending — see
`PLAN.md`.

UI is intentionally kept in sync with `vezne-web`'s design system, not
invented independently — colors here are hand-copied from its `copper`
palette tokens (`vezne-web/src/index.css`, `--background`,
`--foreground`, `--muted-foreground`, `--primary`, `--card`, etc.) and
its `public/favicon.svg` gradient, so the marketing site and the app
look like the same product. There's no shared token file or build-time
import between the two repos (deliberately — no code dependency, see
"Workspace" below), so values must be **copied by hand** and re-checked
against `vezne-web/src/index.css` if they ever drift. Typography is
not yet in sync: `vezne-web` uses `Geist Variable`
(`@fontsource-variable/geist`); this repo currently falls back to
system fonts.

## Stack

- Astro (static output only, no islands/framework components yet)
- Deployed via Cloudflare Pages
- No test framework configured

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Architecture

```
src/pages/   - file-based routing; new routes are src/pages/<name>.astro
src/layouts/ - SiteLayout.astro (base, all pages), LegalLayout.astro (builds on SiteLayout)
public/      - static assets served as-is (favicon.svg today)
```

## Plan workflow

`PLAN.md` tracks actionable landing-site work only — content and
pages someone could pick up today, plus what's blocked and why (see
that file for the current groupings). Cross-cutting business/infra
decisions (hosting, payment providers, legal engagement) stay in the
workspace-root `LAUNCH-PLAN.md` and only surface here once they
unblock something concrete.

## Workspace

This repo lives inside a larger workspace (`../`) alongside
`vezne-api` and `vezne-web` — see `../AGENTS.md`. It has no code
dependency on either sibling (no shared API contract); the only
connection is the CTA link out to `app.vezne.app`. The
workspace-root `LAUNCH-PLAN.md` is not itself committed to any git
repo (the root folder isn't version-controlled), which is why it can
hold sensitive vendor/legal detail that must stay out of this public
repo.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)
