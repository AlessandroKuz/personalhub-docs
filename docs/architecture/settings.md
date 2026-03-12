# Settings & Configuration

## The Settings Split

```
config/settings/
├── __init__.py     # empty — makes settings/ a Python package
├── base.py         # shared: apps, middleware, templates, i18n, email config
├── dev.py          # imports base.*, adds: DEBUG, SQLite, debug-toolbar
└── prod.py         # imports base.*, adds: PostgreSQL, HTTPS headers
```

### How the environment variable flows

1. `.env` sets `DJANGO_SETTINGS_MODULE=config.settings.dev`
2. `python-dotenv` loads `.env` at startup (`load_dotenv()` called in `base.py`)
3. `manage.py` has `os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.dev')`
4. `setdefault` means: *use this value only if the variable isn't already set*
5. In production `.env` sets `config.settings.prod` — `manage.py` never overrides it

---

## Middleware Order (load-bearing)

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",        # must be 2nd
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.locale.LocaleMiddleware",         # after Session, before Common
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
    "django_htmx.middleware.HtmxMiddleware",             # adds request.htmx
]
```

**Why `LocaleMiddleware` must be after `SessionMiddleware`:**
`LocaleMiddleware` reads the language preference stored in the session. If it runs before
`SessionMiddleware`, the session hasn't been loaded yet and it falls back to the
`Accept-Language` header every time, ignoring the user's saved language preference.

**Why `WhiteNoise` must be 2nd:**
It needs to intercept static file requests before any other middleware processes them.
Placing it after `SecurityMiddleware` (which should always be first) ensures security
headers are still applied to static file responses.

---

## Environment Variables

See `.env.example` in the project root for the complete list.

!!! warning
    Never commit `.env`. The `.env.example` file is committed as a template.
    The actual `.env` is in `.gitignore`.

Key variables:

| Variable | Used in | Purpose |
|---|---|---|
| `SECRET_KEY` | base | Django cryptographic signing |
| `DJANGO_SETTINGS_MODULE` | manage.py / asgi.py | Which settings file to load |
| `ALLOWED_HOSTS` | base | Comma-separated list of valid hostnames |
| `EMAIL_*` | base | SMTP configuration |
| `POSTGRES_*` | prod | Database connection |

---

## Security Settings (prod.py)

```python
SECURE_HSTS_SECONDS = 31536000          # 1 year HSTS
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_SSL_REDIRECT = True              # redirect HTTP → HTTPS
SESSION_COOKIE_SECURE = True            # cookies only over HTTPS
CSRF_COOKIE_SECURE = True
```

!!! note
    `SECURE_SSL_REDIRECT` can be disabled if Caddy/Cloudflare handles the redirect
    upstream. Doing it twice is harmless but redundant. If you use Cloudflare's
    "Always Use HTTPS" feature, you can leave this off on the Django side.
