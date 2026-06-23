> Last updated: June 2026

# Deployment

## First Deploy Checklist

- [x] `.env` created with all production values set
- [x] `DJANGO_SETTINGS_MODULE=config.settings.prod`
- [x] `DEBUG=False` (enforced by prod.py, but confirm)
- [x] `SECRET_KEY` is a long random string — never the dev key
- [x] `ALLOWED_HOSTS` includes your domain name
- [x] All `POSTGRES_*` variables set
- [ ] `docker compose up -d` succeeds
- [ ] `docker compose exec web python manage.py migrate`
- [ ] `docker compose exec web python manage.py createsuperuser`
- [ ] HTTPS working (Caddy auto-provision cert on first request)
- [ ] `/stratos/` accessible and login works

---

## Generating a Secure SECRET_KEY

```bash
uv run python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Never reuse the dev `SECRET_KEY` in production. Never commit it.

---

## Updating the Site

```bash
git pull
docker compose build web
docker compose up -d

# If there are new migrations:
docker compose exec web python manage.py migrate
```

The `up -d` command replaces running containers with newly built ones with zero downtime
(for the brief container restart period).

---

## Updating the Docs

MkDocs source lives in the `docs/` submodule. Push to the personalhub-docs repo;
Cloudflare Pages auto-deploys from the main branch.

```bash
uv run mkdocs build          # builds docs/site/
```

---

## Backing Up the Database

```bash
# Create a dump
docker compose exec db pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} > backup.sql

# Restore
docker compose exec -T db psql -U ${POSTGRES_USER} ${POSTGRES_DB} < backup.sql
```

Schedule this with a cron job or a simple shell script on the host. Store backups off-host
(e.g. Backblaze B2, which has a generous free tier).

---

## Environment-Specific Behaviour Summary

| Setting | dev.py | prod.py |
|---|---|---|
| `DEBUG` | `True` | `False` |
| Database | SQLite | PostgreSQL |
| Email backend | Console (prints to terminal) | SMTP (actually sends) |
| Static files | Served by Django dev server | Served by WhiteNoise |
| HTTPS redirect | Off | On |
| Debug toolbar | Enabled | Not installed |
