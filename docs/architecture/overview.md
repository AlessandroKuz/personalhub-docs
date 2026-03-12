# Architecture Overview

## The Stack in Layers

```
┌─────────────────────────────────────────────────────┐
│  Django 6 (the brain)                               │
│  Routing · Views · ORM · Templates · Admin          │
├─────────────────────────────────────────────────────┤
│  HTMX (the nervous system)                          │
│  Partial page updates without a full SPA            │
├─────────────────────────────────────────────────────┤
│  Bootstrap 5 + minimal JS (the body)                │
│  Responsive layout · Theme toggle · Interactions    │
└─────────────────────────────────────────────────────┘
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
│   │   └── prod.py             # PostgreSQL, HTTPS headers, security settings
│   ├── urls.py                 # Root URL dispatcher
│   ├── asgi.py                 # ASGI entry point (uvicorn target)
│   └── wsgi.py
│
├── apps/
│   ├── core/                   # Phase 1: home, about, career, contact
│   ├── projects/               # Phase 2: Project + Tag models
│   └── blog/                   # Phase 3: Post model, writing interface
│
├── templates/
│   ├── base.html               # Master layout
│   ├── components/             # _nav.html, _footer.html, _cta.html
│   └── partials/               # HTMX fragments
│
├── static/
│   ├── css/
│   ├── js/
│   └── img/
│
├── locale/                     # i18n .po / .mo files
├── docs/                       # MkDocs source (you are here)
├── site/                       # MkDocs build output (gitignored)
│
├── manage.py                   # Django CLI entry point
├── mkdocs.yml                  # MkDocs configuration
├── pyproject.toml              # uv: dependencies + tool config (ruff, etc.)
├── uv.lock                     # Committed lockfile — guarantees reproducible installs
├── .python-version             # Pins Python 3.14 — read by uv and mise
├── Dockerfile                  # Production image definition
├── docker-compose.yml          # Dev environment
├── docker-compose.prod.yml     # Production environment
├── .env                        # Secret values — gitignored
└── .env.example                # Committed template for .env
```

---

## Request Lifecycle

A request to `/it/projects/` travels this path:

```
Browser
  → Cloudflare (CDN / DDoS layer)
    → Caddy (TLS termination, reverse proxy)
      → Uvicorn (ASGI server)
        → Django middleware stack (security, locale, CSRF, ...)
          → config/urls.py (root dispatcher)
            → i18n_patterns detects 'it' prefix, sets language
              → apps/projects/urls.py
                → ProjectListView (async)
                  → ORM → PostgreSQL
                    → Django Template → full HTML response
```

For an HTMX partial request (e.g. filtering projects by tag), the response is an
HTML fragment (`templates/partials/_project_grid.html`) rather than a full page.
HTMX swaps it into the DOM without a page reload.
