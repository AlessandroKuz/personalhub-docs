# Personalhub — Developer Documentation

Personal portfolio and central hub for Alex — ML Engineer, Data Scientist, Builder.

Built with Django 6 + HTMX + Bootstrap 5, served async via Uvicorn/ASGI, containerised
with Docker, fronted by Cloudflare.

---

## Quick Start

```bash
git clone https://www.github.com/AlessandroKuz/personalhub
cd personalhub
cp .env.example .env        # fill in SECRET_KEY at minimum
uv sync
uv run uvicorn config.asgi:application --reload --port 8080
```

Visit `http://127.0.0.1:8080`.

For docs locally:

```bash
uv run mkdocs serve --dev-addr 127.0.0.1:8001
```

---

## Project at a Glance

| Property | Value |
| --- | --- |
| Python | 3.14 |
| Django | 6.0 |
| Async server | Uvicorn (ASGI) |
| CSS framework | Bootstrap 5 |
| Interactivity | HTMX |
| Package manager | uv |
| Database (dev) | SQLite |
| Database (prod) | PostgreSQL |
| Languages | EN (default), IT, (ES, DE - Soon) |

---

## What's Here

| Section | Contents |
| --- | --- |
| [Architecture](architecture/overview.md) | Project structure, settings split, URL tree, async rationale |
| [Apps](apps/core.md) | Per-app views, models, URL patterns |
| [Frontend](frontend/design-system.md) | Design system, templates, HTMX patterns, i18n |
| [Infrastructure](infrastructure/docker.md) | Docker setup, deployment, hosting options |
| [Decisions Log](decisions.md) | Why things were done the way they were |

---

## Development Phases

### Phase 1 — Personal Hub *(current)*

Static-ish content. No database models needed for core content.
Pages: Landing, About, Work/Career (with timeline + CV download), Contact.

### Phase 2 — Dynamic Projects

`apps.projects` with `Project` + `Tag` models. HTMX tag filtering.
Django admin for content management.

### Phase 3 — Blog

`apps.blog` with `Post` model. Markdown editor. WebSocket live preview.
Custom lightweight writing interface.
