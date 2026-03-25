> Last updated: 25th March 2026

# Personal Hub — Project Context & Architecture Reference

---

## 1. Who Is This For

**Alessandro Kuz** (also known as "Alex" or "Kuz") — Data Scientist, AI/ML
Engineer, and Builder. 3+ years of professional experience.
Based in Italy. Polyglot: Italian (native), English (fluent), Spanish, Russian,
Ukrainian, German (decreasing fluency).

The website serves two audiences simultaneously:

- **The world** — a central hub that communicates who Alex is, what he builds,
  what he thinks, and how to reach him. Functions as both a personal brand and
  a living CV.
- **Future Alex** — a codebase that teaches, not just works. Every
  architectural decision is documented so the project can be reproduced from
  scratch with minimal help.

---

## 2. Goals & Philosophy

### Primary Goals

1. Ship a clean, minimal personal website/portfolio.
2. Learn the full stack end-to-end: Django, HTMX, async Python, Docker,
   deployment, DNS.
3. Build a foundation that grows through three phases without requiring
   structural rewrites.

### Design Philosophy
>
> "Do more with less."

- **No JavaScript frameworks** (no React, Vue, Angular). The server assembles
  HTML; the browser displays it.
- **HTMX** for dynamic interactions — partial page swaps triggered by HTML
  attributes, not JS.
- **Minimal custom JS** — only for things HTMX cannot handle (theme toggle,
  scroll effects).
- **Bootstrap 5** as the CSS framework — responsive grid and components without
  fighting the tool.
- **Fast, accessible, indexable** — server-side rendering means no hydration,
  no flash of unstyled content, full SEO compatibility out of the box.

### Design Aesthetic

- Monochromatic with single accent: warm white / near-black + blue accent
  (`#c0392b` light, `#e74c3c` dark).
- Light and dark themes, toggled via a `data-theme` attribute on `<html>`.
- CSS custom properties (`--color-bg`, `--color-accent`, etc.) — never
  hardcoded colors.
- Fully responsive: desktop and mobile first-class citizens.

---

## 3. Tech Stack

| Layer | Technology | Why |
| --- | --- | --- |
| Language | Python 3.14 | Bleeding edge, intentional |
| Framework | Django 6.0 | Batteries-included, mature async support |
| Async server | Uvicorn (ASGI) | Non-blocking, ready for WebSockets in future |
| Templating | Django Templates | Server-side, no JS bundle, SEO-friendly |
| Interactivity | HTMX | HTML-attribute-driven partial updates |
| CSS | Bootstrap 5 + custom CSS | Responsive grid, no JS framework needed |
| Static files | WhiteNoise | Serves static files without a separate layer |
| Package manager | uv | Single tool: replaces pip + venv + pip-tools |
| Linter/formatter | Ruff | Fast, configured in pyproject.toml |
| Dev tools | django-debug-toolbar | Query inspection, template analysis |
| i18n | Django i18n + manual translations | EN/IT/ES/DE |
| Containerisation | Docker + Docker Compose | Dev/prod parity, easy migration |
| Reverse proxy | Caddy | Auto-HTTPS via Let's Encrypt, minimal config |
| DDoS / CDN | Cloudflare (free tier) | Absorbs attacks, hides origin IP |
| Database (dev) | SQLite | Zero config, sufficient for local work |
| Database (prod) | PostgreSQL | ACID, full-featured, Django's preferred |

---

## 4. Project Structure

```
personalhub/
│
├── config/                         # Project config
│   ├── settings/
│   │   ├── __init__.py             # Empty package marker
│   │   ├── base.py                 # Shared across all environments
│   │   ├── dev.py                  # Local Development settings
│   │   └── prod.py                 # Production settings
│   ├── urls.py                     # Root URL dispatcher
│   ├── asgi.py                     # ASGI entry point (uvicorn target)
│   └── wsgi.py                     # Kept for compatibility
│
├── apps/
│   ├── core/                       # Phase 1: home, about, work, contact
│   │   ├── __init__.py
│   │   ├── apps.py                 # AppConfig (name = "apps.core")
│   │   ├── urls.py
│   │   └── views.py                # Async views
│   ├── projects/                   # Phase 2: Project + Tag models, HTMX filtering
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── urls.py
│   │   └── views.py
│   ├── blog/                       # Phase 3: Post model, admin writing UI
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── urls.py
│   │   └── views.py
│   └── chat/                       # Phase 5: A ChatBot to interact about Alex
│       ├── __init__.py
│       ├── apps.py
│       ├── models.py
│       ├── urls.py
│       └── views.py
│
├── templates/
│   ├── base.html                   # Master layout: nav, footer, meta, theme
│   ├── components/                 # Reusable UI: _nav.html, _footer.html, _cta.html
│   └── partials/                   # HTMX fragments: _project_card.html, etc.
│
├── static/
│   ├── css/
│   │   ├── main.css                # Design tokens (CSS custom properties)
│   │   └── components/             # Per-component stylesheets
│   ├── js/
│   │   └── main.js                 # Theme toggle, micro-interactions only
│   └── img/
│
├── locale/                         # i18n translation files (.po / .mo)
│   ├── en/
│   ├── it/
│   ├── es/
│   └── de/
│
├── pyproject.toml                  # uv: deps + ruff config
├── uv.lock                         # Committed — guarantees reproducibility
├── .python-version                 # "3.14" — read by uv and mise
├── Dockerfile
├── docker-compose.yml              # Dev: volume mounts, Django dev server
├── docker-compose.prod.yml         # Prod: built image, gunicorn/uvicorn, postgres
├── .env.example                    # Committed template — never commit .env
├── .env                            # Gitignored
└── manage.py
```

---

## 5. Settings Architecture

Settings are split by environment. `DJANGO_SETTINGS_MODULE` controls which file
loads.

```
base.py         ← everything shared (apps, middleware, templates, i18n, email config)
  ├── dev.py    ← adds DEBUG, SQLite, debug-toolbar, console email backend
  └── prod.py   ← adds PostgreSQL, HTTPS headers, security settings
```

**How the environment variable flows:**

1. `.env` sets `DJANGO_SETTINGS_MODULE=config.settings.dev` (loaded by `python-dotenv`)
2. `manage.py` has `setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.dev')`
   as a fallback
3. `setdefault` means: "use this value *only if the variable isn't already set*"
4. In production, `.env` sets `config.settings.prod` — `manage.py` never
   overrides it

**Critical ordering in `MIDDLEWARE`:**

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",       # added
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.locale.LocaleMiddleware",        # added
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
    "django_htmx.middleware.HtmxMiddleware",            # added
]
```

---

## 6. i18n (Internationalisation)

Languages supported: English (default), Italian, Spanish, German.
All translations are done manually by Alex.

**How Django i18n works:**

- Templates use `{% trans "Hello" %}` or `{% blocktrans %}...{% endblocktrans %}`
- `makemessages` scans the codebase and generates `.po` files in `locale/`
- Alex fills in translations in the `.po` files
- `compilemessages` compiles `.po` → binary `.mo` files Django reads at runtime
- `LocaleMiddleware` detects the user's preferred language from: URL prefix → session →
  `Accept-Language` header → `LANGUAGE_CODE` default

**URL strategy:** Language prefix in URL (`/it/about/`, `/en/about/`) — clean, shareable,
SEO-friendly. Implemented via `i18n_patterns()` in `config/urls.py`.

**Key setting:**

```python
LANGUAGES = [
    ('en', 'English'),
    ('it', 'Italiano'),
    ('es', 'Español'),
    ('de', 'Deutsch'),
]
LOCALE_PATHS = [BASE_DIR / "locale"]
```

---

## 7. Async Architecture

Django is running under **ASGI via Uvicorn** from day one. This means:

- No refactoring needed when WebSockets are added in phase 3 (blog live preview)
- Higher concurrency under load (event loop vs. thread pool)
- Views are written as `async def` with `await` on all ORM calls

**The one gotcha — sync code in async context:**

```python
# Use native async ORM methods:
projects = await Project.objects.filter(featured=True).alist()

# Or wrap unavoidably sync third-party calls:
from asgiref.sync import sync_to_async
result = await sync_to_async(some_sync_library_call)(args)
```

**Dev server:** Always use `uvicorn` directly, never `manage.py runserver` — the
latter uses WSGI internally and won't test your async code paths.

```bash
uv run uvicorn config.asgi:application --reload
```

---

## 8. Multi-Phase Development

### Phase 0 — Personal Hub (Current)

**Goal:** Ship something real. Static-ish content, no models needed.

Content:

- **Landing page** — TLDR of each section, CTA to contact, CV download
  - Hero
  - About
  - Work
  - Projects
  - Process
  - **Contact section** — email + cal.com links

Architecture:

- `apps.core` only
- `TemplateView` or simple async function views
- No database queries (or minimal — contact form logs optionally)

### Phase 1 — Personal Hub (Current)

**Goal:** Expands on the landing page by adding extended versions of the
sections to dedicated pages.

New:

- Dedicated pages:
  - **About** — personal description, interests, personality
  - **Work/Career** — skills, achievements, interactive timeline (education + career),
    CV download
  - **Contact** — email form (HTMX submission) + cal.com link
  - **Error pages** — 400/500 pages with custom errors

### Phase 2 — Dynamic Projects

**Goal:** Manage and display projects via Django admin.

New:

- `apps.projects` with `Project` and `Tag` models
- `ListView` + `DetailView` (async)
- HTMX: filter projects by tag without full page reload
- Django admin for content management

### Phase 3 — Blog

**Goal:** Write and publish from a simple interface.

New:

- `apps.blog` with `Post` model (title, slug, body, status, published_at)
- Custom writing view with Markdown editor (EasyMDE via CDN)
- WebSocket-based live preview (this is where ASGI pays off visibly)
- Django admin + potentially a custom lightweight editor view

### Phase 4 — Polish and cleanup

**Goal:** Complete cleanup of code and content, small improvements and refactoring.

New:

- Improve content
- Improve HTML, CSS & JS
- Improve code structure if necessary

### Phase 5 — Chat

**Goal:** Interact with a simple ChatBot about Alex, his projects, career and
the website.

New:

- `apps.chat` with chatbot with RAG.
- Interaction on multiple languages.
- Custom greetings and interaction.
- Content ingestion with CV, Career data, projects and meta-knowledge of itself.

---

## 9. URL Architecture

```python
# config/urls.py
from django.conf import settings
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.i18n import i18n_patterns

urlpatterns = [
    path('admin/', admin.site.urls),
    path('i18n/', include('django.conf.urls.i18n')),  # language switcher endpoint
]

# All user-facing URLs get language prefix: /en/... /it/...
urlpatterns += i18n_patterns(
    path('', include('apps.core.urls')),
    path('projects/', include('apps.projects.urls')),
    # ...
    prefix_default_language=True,  # /about/ redirects to /en/about/
)
```

---

## 10. Hosting Strategy

Two approaches will be explored and compared:

### Option A — Home Server (Dell Optiplex) + Cloudflare Tunnel

`Internet → Cloudflare Edge → [outbound tunnel] → cloudflared → Docker → Django`

- Cost: ~€0/mo (electricity already paid)
- Home IP never exposed — Cloudflare Tunnel makes an outbound connection, no
  port forwarding
- DDoS protection: Cloudflare absorbs it before it reaches the server
- Risk: power cut or ISP outage = downtime
- Best for: learning, homelab, personal tolerance for occasional downtime

### Option B — VPS (~€5/mo) + Caddy

`Internet → Cloudflare (DNS + CDN) → VPS → Caddy → Docker → Django`

- Cost: fixed €4/mo
- 99.9% uptime SLA
- Caddy handles automatic TLS (Let's Encrypt) with zero config
- Best for: reliable uptime, professional presentation

**Migration between options is trivial** — everything is Dockerized.
Copy `docker-compose.prod.yml` + `.env` to new host, run `docker compose up`. Done.

Maybe **Hetzner VPS (CX22, ~€4/mo)** or **Linode**.

### Production Stack

docker-compose.prod.yml:

- Django + Uvicorn (ASGI)
- PostgreSQL
- Caddy (reverse proxy, auto-HTTPS)
- cloudflared (if home server option)

---

## 11. Key Commands Reference

```bash
# Install dependencies
uv sync

# Run dev server (ASGI)
uv run uvicorn config.asgi:application --reload

# Django management
uv run python manage.py check
uv run python manage.py migrate
uv run python manage.py makemigrations
uv run python manage.py createsuperuser

# i18n workflow
uv run python manage.py makemessages -l it    # extract strings to .po files
uv run python manage.py compilemessages       # compile .po → .mo

# Static files (prod only)
uv run python manage.py compress
uv run python manage.py collectstatic

# Linting
uv run ruff check .
uv run ruff format .
```

---

## 12. Environment Variables Reference

```bash
# .env.example

# Django
SECRET_KEY=
DJANGO_SETTINGS_MODULE=config.settings.dev  # replacable with prod
ALLOWED_HOSTS=localhost,127.0.0.1

# Email (dev uses console backend — these are ignored in dev)
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_HOST_USER=
EMAIL_HOST_PASSWORD=
DEFAULT_FROM_EMAIL=
CONTACT_EMAIL=

# PostgreSQL (prod only)
POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_HOST=db
POSTGRES_PORT=5432
```

---

## 13. Decisions Log

| Decision | Rationale |
| --- | --- |
| ASGI from day 1 | Zero refactor cost now; WebSockets needed in phase 3 |
| SQLite dev / Postgres prod | Simplicity in dev, correctness in prod |
| uv over pip+venv | Single tool, faster, lockfile for reproducibility |
| WhiteNoise over nginx for static | Simpler stack no value at this scale |
| Bootstrap 5 | Prebuilt components, no build step |
| i18n_patterns | Clean URLs redirect correctly, prefixed for others |
| Debug toolbar in Development | Easier development, security in production |
| Cloudflare Tunnel | No exposed home IP, no port forwarding, DDoS absorbed |
