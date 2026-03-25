> Last updated: 25th March 2026

# Async & ASGI

## Why ASGI from Day 1

| | WSGI | ASGI |
|---|---|---|
| Concurrency model | Thread per request | Event loop (single thread, many requests) |
| WebSocket support | No | Yes |
| Django support | Full, mature | Full, mature (Django 4.1+) |
| Cost to adopt now | 0 | ~0 |
| Cost to adopt later | — | High (every view, every ORM call) |

Starting async costs ~0 extra effort on a new project. Phase 3 (blog live preview)
requires WebSockets, which are ASGI-only. Starting async avoids a future structural rewrite.

---

## Writing Async Views

```python
# Simple async view — no DB
async def home(request):
    return render(request, 'core/home.html')

# Async view with database access
async def project_list(request):
    projects = await Project.objects.filter(featured=True).alist()
    return render(request, 'projects/list.html', {'projects': projects})
```

### Async ORM Methods

Django 4.1+ provides async equivalents for all queryset operations:

| Sync | Async |
|---|---|
| `.get(...)` | `.aget(...)` |
| `.first()` | `.afirst()` |
| `list(queryset)` | `await queryset.alist()` |
| `.create(...)` | `.acreate(...)` |
| `instance.save()` | `await instance.asave()` |
| `instance.delete()` | `await instance.adelete()` |

---

## Handling Sync Code in Async Context

When a third-party library has no async support:

```python
from asgiref.sync import sync_to_async

# Wrap any blocking call
result = await sync_to_async(some_sync_library_call)(args)
```

`asgiref` ships with Django — no extra install needed.

!!! warning
    Calling a sync function directly inside an async view blocks the entire event loop
    for all concurrent requests. Django will raise `SynchronousOnlyOperation` in strict
    mode to catch this.

---

## Dev Server

Always use Uvicorn in development to exercise the actual async code path:

```bash
uv run uvicorn config.asgi:application --reload --port 8080
```

`manage.py runserver` wraps async views via `asgiref`'s sync bridge — it works, but
it doesn't give you true async behaviour or WebSocket support during development.

---

## Where ASGI Visibly Pays Off

| Feature | Why ASGI matters |
|---|---|
| Phase 1-2 pages | Doesn't matter — but doesn't hurt |
| Contact form (HTMX) | Still just HTTP — no real difference |
| Phase 3 blog live preview | **WebSocket** — impossible under WSGI |
| Phase 5 chat | Streaming tokens + WebSocket — WSGI blocks the worker for the full inference duration; ASGI awaits non-blocking and streams tokens back as they're generated |
| High concurrency (traffic spike) | Event loop handles more concurrent requests than thread pool |
