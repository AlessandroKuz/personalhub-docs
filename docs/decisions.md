> Last updated: 25th March 2026

# Decisions Log

A running record of architectural and technical decisions, with rationale.
Updated as the project evolves.

---

## ASGI from day 1

**Decision:** Run under Uvicorn/ASGI from the start, write async views throughout.

**Rationale:** The cost of writing async views now is ~0 (same syntax, add `async`/`await`).
The cost of refactoring sync → async later is non-trivial: every view, every ORM
call, every third-party integration needs auditing. Phase 3 (blog) requires
WebSockets, which are ASGI-only. Starting async avoids a future structural
rewrite.

**Consequence:** Always run the dev server as
`uvicorn config.asgi:application --reload`, never `manage.py runserver`
(which uses WSGI internally and won't test async code paths).

---

## SQLite dev / PostgreSQL prod

**Decision:** SQLite in development, PostgreSQL in production.

**Rationale:** SQLite requires zero configuration and is sufficient for local development.
PostgreSQL is Django's best-supported production database — full ACID
compliance, better concurrency, JSON fields, and no gotchas with Django's ORM.
The settings split (`dev.py` / `prod.py`) makes this a single config line difference.

**Watch out for:** SQLite-specific behaviours that silently differ from Postgres — e.g.
case sensitivity in `LIKE` queries, enforcement of foreign key constraints (disabled by
default in SQLite). Django's ORM mostly abstracts these, but raw queries can bite you.

---

## uv over pip + venv + pip-tools

**Decision:** Use `uv` as the single tool for Python version management, virtual
environments, and dependency resolution.

**Rationale:** Replaces three separate tools with one. The `uv.lock` file is committed,
guaranteeing reproducible installs across machines and CI. The `pyproject.toml` is the
single source of truth for both runtime and dev dependencies.

---

## WhiteNoise over a separate nginx for static files

**Decision:** Use WhiteNoise middleware to serve static files directly from Django/Uvicorn.

**Rationale:** At the scale of a personal site, a separate nginx container adds operational
complexity with no meaningful performance benefit. WhiteNoise compresses and caches static
files correctly. If traffic ever justifies it, Cloudflare's CDN handles edge caching anyway.

---

## Bootstrap 5

**Decision:** Use Bootstrap 5 as the CSS framework.

**Rationale:** Bootstrap provides prebuilt, accessible components (navbar, cards, grid,
modals) without a build step. Bootstrap 5 dropped jQuery, so there's no JS dependency conflict with HTMX.

---

## i18n_patterns

**Decision:** Language prefix in URL (`/it/about/`), when no prefix, the website handles redirection.

**Rationale:** Clean URLs for sharing. Automatic redirection. Italian visitors get `/it/about/`,
which is shareable and SEO-distinct, English visitors get `/en/about/` and so on. The
`LocaleMiddleware` handles detection and redirection.

---

## Debug toolbar import inside `if DEBUG` block

**Decision:** The `debug_toolbar` import in `config/urls.py` is placed *inside* the
`if settings.DEBUG:` block, not at the top of the file.

**Rationale:** `debug_toolbar` is in dev dependencies only — not installed in production.
A top-level import would cause an `ImportError` on startup in production even if the URL
registration was conditional. The import inside the `if` block means the module is only
loaded when it's actually installed.

---

## Cloudflare Tunnel for home server hosting

**Decision:** Use Cloudflare Tunnel (`cloudflared`) when self-hosting on the home server.

**Rationale:** The tunnel daemon makes an outbound connection to Cloudflare's edge — no
port forwarding required, home IP never appears in DNS. Cloudflare absorbs DDoS before
it reaches the origin. Free tier is sufficient. Switching to a VPS later requires only
copying `docker-compose.prod.yml` and `.env` to the new host.

---

## MkDocs Material for project documentation

**Decision:** Use MkDocs with the Material theme for all project documentation.

**Rationale:** Markdown source, no Node/React build step, outputs pure static HTML.
Material theme is the de facto standard for Python project docs (used by FastAPI,
Pydantic, etc.). Hosted as a static site on `docs.alessandrokuz.com` served by Caddy,
or via Cloudflare Pages as a free static host.

---

## Two hosting approaches (home server + VPS)

**Decision:** Implement both hosting approaches to gain hands-on experience with each.

**Rationale:** The home Optiplex + Cloudflare Tunnel approach costs nothing and teaches
the full self-hosting stack. The Hetzner VPS approach gives datacenter reliability.
Because everything is Dockerized, the application layer is identical — only the
`cloudflared` service and Caddy config differ. Migrating between them is a copy-paste
operation.
