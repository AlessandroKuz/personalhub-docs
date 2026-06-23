> Last updated: June 2026

# Docker

## Philosophy

Single `docker-compose.yml` for production. No separate dev compose — you run the app
locally with `uv run uvicorn` (no Docker needed for dev). The production compose file
includes everything: Django, PostgreSQL, Redis, Caddy.

The application code is identical in dev and prod — only the infrastructure layer differs.

---

## Dockerfile

Multi-stage build: a builder stage installs dependencies via uv, a runtime stage copies
only what's needed to run.

```dockerfile
# STAGE 1 — BUILDER
FROM ghcr.io/astral-sh/uv:python3.14-trixie-slim AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
  PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  gcc g++ libc6-dev \
  && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project

COPY . .
RUN uv sync --frozen --no-dev


# STAGE 2 — RUNTIME
FROM python:3.14-slim-trixie AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \
  PYTHONUNBUFFERED=1 \
  PATH="/app/.venv/bin:$PATH"

WORKDIR /app

RUN addgroup --system appgroup \
  && adduser --system --ingroup appgroup appuser

COPY --from=builder --chown=appuser:appgroup /app /app
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

RUN mkdir -p /srv/personalhub/staticfiles \
  && chown -R appuser:appgroup /srv/personalhub

USER appuser
EXPOSE 8000
ENTRYPOINT ["/entrypoint.sh"]
```

The entrypoint runs migrations, collectstatic, and compress before starting Uvicorn.

---

## docker-compose.yml (production)

```yaml
services:
  web:
    image: personalhub-web:latest
    build:
      context: .
      target: runtime
    restart: unless-stopped
    env_file: .env
    environment:
      - DJANGO_SETTINGS_MODULE=config.settings.prod
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - static_files:/srv/personalhub/staticfiles
    healthcheck:
      test: ["CMD", "python", "-c", "import http.client; c = http.client.HTTPConnection('localhost', 8000, timeout=5); c.request('GET', '/health/'); r = c.getresponse(); exit(0 if r.status == 200 else 1)"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 90s
    networks:
      - internal

  db:
    image: postgres:18-alpine
    restart: unless-stopped
    env_file: .env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - internal

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 10s
    networks:
      - internal

  caddy:
    image: caddy:2-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
      - ./certs:/etc/caddy/certs:ro
      - static_files:/srv/personalhub/staticfiles:ro
    depends_on:
      web:
        condition: service_healthy
    networks:
      - internal

volumes:
  postgres_data:
  redis_data:
  caddy_data:
  caddy_config:
  static_files:

networks:
  internal:
    driver: bridge
```

---

## Useful Commands

```bash
# Build and start
docker compose build
docker compose up -d
docker compose logs -f web

# Management
docker compose exec web python manage.py migrate
docker compose exec web python manage.py createsuperuser

# Flush Redis cache
docker compose exec redis redis-cli FLUSHALL

# Rebuild after code changes (prod)
docker compose build
docker compose up -d
```

!!! note
    The compose file targets the `runtime` stage via `target: runtime`.
    To rebuild after code changes, run `docker compose build` then `docker compose up -d`.
