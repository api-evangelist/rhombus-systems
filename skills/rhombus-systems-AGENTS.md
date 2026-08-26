> ## Documentation Index
> Fetch the complete documentation index at: https://api-docs.rhombus.community/llms.txt
> Use this file to discover all available pages before exploring further.

# AGENTS

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Rhombus Developer Documentation site built with **Mintlify**. Documents the complete Rhombus Systems API (all endpoints) with auto-generated content fetched live from the production OpenAPI spec on every Mintlify build.

* **Live site**: [https://api-docs.rhombus.community/](https://api-docs.rhombus.community/)
* **API base URL**: [https://api2.rhombussystems.com](https://api2.rhombussystems.com)
* **OpenAPI spec source**: [https://api2.rhombussystems.com/api/openapi/public.json](https://api2.rhombussystems.com/api/openapi/public.json)

## Development Commands

```bash theme={null}
# Start local dev server (run from repo root where docs.json lives)
mint dev

# Start on custom port
mint dev --port 3333

# Validate all links
mint broken-links

# Update Mintlify CLI
mint update

# Content quality check
python3 scripts/check-content-quality.py
```

**Prerequisites**: Node.js 22.x (mint CLI fails on Node 25+; see `.nvmrc`), Mintlify CLI (`npm i -g mint`), Python 3.11+ (for scripts).

## Architecture

### Content lives at the root

All MDX files and `docs.json` live in the **repo root** — there is no `docs/` subdirectory. `mint dev` must be run from the root.

### Content types

1. **Hand-written MDX** — implementation guides, quickstart, MCP docs (`index.mdx`, `implementations/`, `low-code-no-code/`, `websocket/`)
2. **Auto-generated API docs** — all endpoint pages under the API Reference tab, fetched live from the upstream OpenAPI spec at build time
3. **Auto-served AI files** — `llms.txt`, `llms-full.txt`, `skill.md`, and `sitemap.xml` are served by Mintlify directly. **Do not add local copies** — they override and go stale.

### docs.json — the central config

* Controls all navigation, theming, and site behavior
* 3 tabs: **Guides**, **Integrations**, **API Reference**
* API endpoints are referenced via the `openapi` key pointing to the live URL `https://api2.rhombussystems.com/api/openapi/public.json` — do not manually create endpoint pages
* Mintlify auto-generates all API Reference endpoint pages from the OpenAPI spec on every build

### Reusable snippets

Shared MDX content goes in `snippets/` — includes `changelog-author.jsx` React component.

### Scripts (`scripts/`)

* `check-content-quality.py` — Validate frontmatter, test-file residue, and `docs.json` integrity

### OpenAPI sync (Mintlify-native)

There is no local enrichment pipeline. `docs.json` references the live upstream spec; Mintlify fetches it on every build. Three rebuild triggers exist: (1) any push to `main` auto-deploys, (2) the API team's CD pipeline calls Mintlify's [Trigger Deployment](https://www.mintlify.com/docs/api/update/trigger) endpoint after backend releases (`POST https://api.mintlify.com/v1/project/update/{projectId}` with `Authorization: Bearer mint_…`), and (3) a Mintlify dashboard Workflow on a 6-hour cron (`0 */6 * * *` UTC) rebuilds the site autonomously to keep the live OpenAPI spec fresh — this is the floor that guarantees rebuilds at least every 24 hours and lives in the Mintlify dashboard (Workflows), not in `.github/workflows/`. Tag descriptions, `x-mint` group metadata, and `x-codeSamples` should live in the upstream spec, not in transform scripts here. The changelog (`changelog.mdx`) is hand-written.

## Localization (English + Spanish)

The site is bilingual. English is the default and lives at the repo root; Spanish translations live in a parallel `es/` directory with mirrored filenames and subfolders, per [Mintlify's i18n guide](https://www.mintlify.com/docs/guides/internationalization). Spanish coverage is full parity for all Guides + Integrations content (34 files); only the API Reference tab is English-only because the upstream OpenAPI spec is English. Each Spanish page opens with a `<Note>` callout disclosing the translation was machine-generated. **Parity rule**: when you edit or add an English page in the Spanish-covered set, update the matching `es/...` file too — Mintlify does not fall back from Spanish to English, so a missing Spanish page returns 404. `docs.json` carries `navigation.languages: [{language: "en", default: true, ...}, {language: "es", ...}]`; the Spanish nav deliberately omits the API Reference tab.

## Working Standards

* Always ask for clarification rather than making assumptions
* Search for existing content before adding new pages to avoid duplication
* Start by making the smallest reasonable changes
* Push back on ideas when it leads to better documentation — cite sources
* Never use `--no-verify` or skip pre-commit hooks

### MDX page requirements

Every `.mdx` file must have YAML frontmatter with at least `title` and `description`. Use second-person voice ("you"). Match style and formatting of existing pages.

### Mintlify components

* **Steps** — sequential procedures
* **Tabs** — platform-specific alternatives
* **CodeGroup** — same concept in multiple languages
* **Accordions** — progressive disclosure
* **RequestExample / ResponseExample** — API endpoint docs
* **ParamField / ResponseField / Expandable** — API parameter and response documentation
* Callouts: `<Note>`, `<Tip>`, `<Warning>`, `<Info>`, `<Check>`

### API-specific conventions

* All API requests use base URL `https://api2.rhombussystems.com` (US) or `https://api2.eu.rhombussystems.com` (EU)
* Rhombus runs two independent regions (US and EU); every public hostname has a `.eu.` variant, and orgs/API keys are region-bound. Canonical page: `api-regions.mdx`. Code samples default to US hosts and link to `/api-regions`.
* Authentication headers: `x-auth-scheme: api-token` + `x-auth-apikey: YOUR_API_KEY` (identical in both regions)
* Media endpoints use `https://media.rhombussystems.com` (US) or `https://media.eu.rhombussystems.com` (EU)
* Code examples should be complete, runnable, with realistic data
* Use relative paths for internal links, absolute URLs only for external

### Source of truth for verifying content

When checking that a guide's endpoints/fields/auth/behavior are correct, the **primary source of truth is the private enterprise git server `git2.rhombus.corp`** (org `rhombus`) — not the public `github.com/RhombusSystems` repos (those are the customer-facing SDK/example repos the guides link to). The `gh` CLI is authenticated to both hosts: query via `gh api --hostname git2.rhombus.corp repos/rhombus/<repo>/contents/<path>`. Key repos: `rhombus-cloud-webservice` (Main API server), `rhombus-cloud-websockets`, `rhombus-cloud-webhooks`, the alert/access-control consumers, `rhombus-node-private-mcp`, `rhombus-cloud-video`. Cross-check against the live OpenAPI spec (the public surface). The enterprise code is internal — verify against it but only publish public-API surface; never leak internal hostnames, service names, internal-only endpoints, or infra topology.
