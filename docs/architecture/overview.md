> Last updated: June 2026

# Architecture Overview

## The Stack in Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│  Caddy (TLS termination, reverse proxy, rate limiting)              │
├─────────────────────────────────────────────────────────────────────┤
│  Uvicorn (ASGI server, async Python event loop)                     │
├─────────────────────────────────────────────────────────────────────┤
│  Django 6 (the brain)                                               │
│  Routing · Views · ORM · Templates · Admin · Cache                  │
├─────────────────────────────────────────────────────────────────────┤
│  Redis (shared cache + rate-limit state)                            │
├─────────────────────────────────────────────────────────────────────┤
│  PostgreSQL (persistent data)                                       │
├─────────────────────────────────────────────────────────────────────┤
│  HTMX (the nervous system)                                          │
│  Partial page updates without a full SPA                            │
├─────────────────────────────────────────────────────────────────────┤
│  Bootstrap 5 + minimal CSS & JS (the body)                          │
│  Responsive layout · Theme toggle · Interactions                     │
└─────────────────────────────────────────────────────────────────────┘
```

The guiding principle: **the server assembles HTML, the browser displays it.**
No JavaScript framework, no hydration, no client-side routing. HTMX adds interactivity
by making the server return HTML fragments instead of JSON.

---

## Project Structure

> "The tree below is sorted by logical grouping, not alphabetically — directories and files are ordered by purpose and dependency to make the architecture easier to read at a glance."

```
personalhub/
│
├── config/                     # Project config — NOT an app
│   ├── settings/
│   │   ├── __init__.py         # empty — makes settings/ a Python package
│   │   ├── base.py             # shared across all environments
│   │   ├── dev.py              # DEBUG, SQLite, console email, debug-toolbar
│   │   ├── staging.py          # staging environment (optional)
│   │   └── prod.py             # PostgreSQL, HTTPS headers, security settings
│   ├── urls.py                 # Root URL dispatcher
│   ├── asgi.py                 # ASGI entry point (uvicorn target)
│   └── wsgi.py
│
├── apps/
│   ├── core/                   # Phase 1: home, about, work, contact
│   │   └── middleware/
│   │       └── rate_limit.py   # 4-tier rate limiting per path
│   ├── projects/               # Phase 2: Project + Tag models
│   └── blog/                   # Phase 3: Post model, writing interface
│
├── templates/
│   ├── base.html               # Master layout
│   ├── components/             # _nav.html, _footer.html, _alerts.html, _toasts.html
│   ├── 4xx_base.html           # Shared error page layout
│   ├── 429.html                # Rate-limited response
│   └── ...                     # 400.html, 403.html, 404.html, 500.html
│
├── static/
│   ├── scss/                   # Bootstrap overrides (custom.scss)
│   ├── css/
│   ├── js/
│   └── img/
│
├── locale/                     # i18n .po / .mo files
├── docs/                       # MkDocs source (you are here)
├── site/                       # MkDocs build output (gitignored)
├── scripts/
│   └── dev.sh                  # Launches Django + MkDocs simultaneously
│
├── manage.py                   # Django CLI entry point
├── pyproject.toml              # uv: dependencies + tool config (ruff, etc.)
├── uv.lock                     # Committed lockfile — guarantees reproducible installs
├── .python-version             # Pins Python 3.14 — read by uv and mise
├── Dockerfile                  # Multi-stage production image
├── docker-compose.yml          # Full production stack (web + db + redis + caddy)
├── .env                        # Secret values — gitignored
└── .env.example                # Committed template for .env
```

---

## Request Lifecycle

A request to `/it/work/` travels this path:

```text
Browser
  → Cloudflare (CDN / DDoS layer)
    → Caddy (TLS termination, rate-limit check, reverse proxy)
      → Uvicorn (ASGI server)
        → Django middleware stack (security, locale, CSRF, ...)
          → config/urls.py (root dispatcher)
            → i18n_patterns detects 'it' prefix, sets language
              → apps/core/urls.py
                → work view (async)
                  → ORM → PostgreSQL
                    → Django Template → full HTML response
```

If a cached response exists and is fresh, `FetchFromCacheMiddleware` returns it before
any downstream middleware or view processes it — the request never reaches the view.

If the rate limit is exceeded, `RateLimitMiddleware` short-circuits with a 429 response.
For an HTMX partial request (e.g. filtering projects by tag), the response is an
HTML fragment (`templates/partials/_project_grid.html`) rather than a full page.
HTMX swaps it into the DOM without a page reload.
