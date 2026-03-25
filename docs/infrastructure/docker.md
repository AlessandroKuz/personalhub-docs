> Last updated: 25th March 2026

# Docker

## Philosophy

Two Compose files for full environment parity:

- `docker-compose.yml` — development: volume mounts, live reload, no PostgreSQL needed
- `docker-compose.prod.yml` — production: built image, Uvicorn workers, PostgreSQL, Caddy

The application code is identical in both — only the infrastructure layer differs.

---

## Dockerfile

```dockerfile
FROM python:3.14-slim-trixie

# Install uv from its official image (fast, no pip needed)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

# Install dependencies first — Docker caches this layer
# Only re-runs if pyproject.toml or uv.lock changes
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

# Copy application code
COPY . .

# Compile translation files
RUN uv run python manage.py compilemessages

# Collect static files into staticfiles/
RUN uv run python manage.py collectstatic --noinput

EXPOSE 8000
CMD ["uv", "run", "uvicorn", "config.asgi:application",
     "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

!!! note
    `--frozen` on `uv sync` means: fail if `uv.lock` is out of sync with `pyproject.toml`.
    This prevents silent dependency drift in production images.

---

## docker-compose.yml (development)

```yaml
services:
  web:
    build: .
    command: uv run uvicorn config.asgi:application --reload --host 0.0.0.0 --port 8000
    volumes:
      - .:/app           # mount source code — changes reflect without rebuild
    ports:
      - "8000:8000"
    env_file:
      - .env
```

No PostgreSQL in dev — SQLite file lives at `db.sqlite3` in the mounted volume.

---

## docker-compose.prod.yml (production)

```yaml
services:
  web:
    build: .
    restart: always
    env_file: .env
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:18-alpine
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      retries: 5

  caddy:
    image: caddy:2-alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config

volumes:
  postgres_data:
  caddy_data:
  caddy_config:
```

---

## Useful Commands

```bash
# Development
docker compose up                           # start dev stack
docker compose exec web uv run python manage.py migrate
docker compose exec web uv run python manage.py createsuperuser

# Production
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml exec web uv run python manage.py migrate
docker compose -f docker-compose.prod.yml logs -f web

# Rebuild after code changes (prod)
docker compose -f docker-compose.prod.yml build
docker compose -f docker-compose.prod.yml up -d
```
