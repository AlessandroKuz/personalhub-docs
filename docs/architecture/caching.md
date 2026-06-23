> Last updated: June 2026

# Caching & Rate Limiting

## Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Caddy      │ ──▶ │   Uvicorn    │ ──▶ │    Redis     │
│ rate_limit   │     │ (Django)     │     │ cache + rate │
│ 100 r/m      │     │              │     │ limit state  │
└──────────────┘     │ Cache MIDDLE │     └──────────────┘
                     │ RateLimit    │
                     │ @cache_page  │
                     │ {% cache %}  │
                     └──────────────┘
```

Three layers of caching + rate limiting:

1. **Caddy edge** — IP-based rate limit (100 req/min, burst 15)
2. **Django middleware** — full-page cache + per-path rate limiting
3. **Template fragments** — nav, footer cached independently

---

## Redis Backend

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": os.environ.get("REDIS_URL", "redis://localhost:6379/0"),
    }
}

CACHE_MIDDLEWARE_ALIAS = "default"
CACHE_MIDDLEWARE_SECONDS = 3600  # 1 hour
CACHE_MIDDLEWARE_KEY_PREFIX = "ph"
```

Redis serves dual purpose:
- **Cache store** — page responses, template fragments
- **Rate-limit state** — atomic counters with TTL

---

## Cache Tiers

| Tier | Mechanism | TTL | Scope |
|---|---|---|---|
| Full-page | `UpdateCacheMiddleware` + `FetchFromCacheMiddleware` | 3600s | Anonymous GET responses for all pages |
| View-level | `@cache_page(3600)` on individual views | 3600s | Home, About, Work, Projects, Contact |
| Fragment | `{% cache 3600 "nav" LANGUAGE_CODE %}` | 3600s | Nav, Footer |

All three tiers share the same 1-hour TTL. Cache invalidation is manual
(`FLUSHALL` via `docker compose exec redis redis-cli FLUSHALL`) or automatic
on TTL expiry.

---

## Rate Limit Tiers

The `RateLimitMiddleware` (`apps/core/middleware/rate_limit.py`) uses a sliding
window via Redis atomic increment:

| Tier | Rate | Scope | Key |
|---|---|---|---|
| Global | 100/minute | Per IP across all paths | `rl:{ip}:global` |
| Admin | 30/minute | Per IP to /stratos/ paths | `rl:{ip}:admin` |
| Login | 5/minute | Per IP to /stratos/login/ | `rl:{ip}:login` |
| Health | 10/10s | Per IP to /health/ | `rl:{ip}:health` |

```python
RATE_LIMIT_TIERS = {
    "global": {"limit": 100, "window": 60},
    "admin":  {"limit": 30,  "window": 60},
    "login":  {"limit": 5,   "window": 60},
    "health": {"limit": 10,  "window": 10},
}
```

Administrators and staff are exempt from rate limiting.

---

## Caddy Edge Rate Limit

```caddy
rate_limit {
    zone dynamic {
        key {remote_host}
        events 100
        window 1m
        burst  15
    }
}
```

Caddy rejects excess requests before they reach Uvicorn. This protects against
IP flood attacks. The Django middleware handles per-path granularity.

---

## Cache Keys

| Pattern | Example |
|---|---|
| Page response | `ph:views.decorators.cache.cache_page:GET:en:/work/` |
| Nav fragment | `ph:template.cache.nav.en` |
| Footer fragment | `ph:template.cache.footer.en` |
| Rate limit | `rl:192.168.1.1:global` |

---

## Test Mode

In test mode, the cache backend is overridden to `DummyCache`:

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.dummy.DummyCache",
    }
}
```

No Redis dependency. Rate limit never triggers because `DummyCache.add()` always
returns True.

---

## Useful Commands

```bash
# Flush entire cache
docker compose exec redis redis-cli FLUSHALL

# Inspect cache keys
docker compose exec redis redis-cli KEYS '*'

# Check rate limit counters
docker compose exec redis redis-cli KEYS 'rl:*'

# Get TTL of a specific key
docker compose exec redis redis-cli TTL ph:template.cache.nav.en

# Monitor live Redis traffic
docker compose exec redis redis-cli MONITOR
```
