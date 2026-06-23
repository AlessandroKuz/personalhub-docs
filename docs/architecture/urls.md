> Last updated: June 2026

# URL Structure

## The URL Tree

Django's URL routing is a tree. `config/urls.py` is the root; each app's `urls.py`
is a branch. A request walks the tree until a pattern matches.

```
config/urls.py  (root)
├── robots.txt               → robots_txt view
├── sitemap.xml              → sitemap view
├── health/                  → returns "ok" (load-balancer check)
├── stratos/                 ← Django admin (renamed from admin/)
├── i18n/                    ← language switcher POST endpoint
├── [i18n_patterns]
│   ├── /  →                 apps.core.urls
│   ├── projects/  →         apps.projects.urls   # commented until Phase 2
│   ├── blog/  →             apps.blog.urls       # commented until Phase 3
│   └── .../  →              # other apps
└── __debug__/               ← debug toolbar (dev only, import inside if DEBUG)
```

---

## Root URL Configuration

```python
# config/urls.py
from django.conf import settings
from django.conf.urls.i18n import i18n_patterns
from django.contrib import admin
from django.urls import path, include

from apps.core.views import robots_txt
from apps.core.sitemaps import StaticSitemap

sitemaps = {"static": StaticSitemap}

handler400 = "apps.core.views.error_400"
handler403 = "apps.core.views.error_403"
handler404 = "apps.core.views.error_404"
handler429 = "apps.core.views.error_429"
handler500 = "apps.core.views.error_500"

urlpatterns = [
    path("robots.txt", robots_txt, name="robots_txt"),
    path("sitemap.xml", sitemap, {"sitemaps": sitemaps}, name="sitemap"),
    path("health/", lambda _: HttpResponse(b"ok"), name="health"),
    path("stratos/", admin.site.urls),              # renamed from admin/
    path("i18n/", include("django.conf.urls.i18n")),
]

urlpatterns += i18n_patterns(
    path("", include("apps.core.urls")),
    # path('projects/', include('apps.projects.urls')),  # Phase 2
    # path('blog/', include('apps.blog.urls')),          # Phase 3
    prefix_default_language=True,
)

if settings.DEBUG:
    from debug_toolbar.toolbar import debug_toolbar_urls
    urlpatterns += debug_toolbar_urls()
    # Import is INSIDE the if block — critical.
    # A top-level import crashes prod because debug_toolbar is not installed there.

    # Error page previews
    urlpatterns += [
        path("__errors/400/", lambda r: render(r, "400.html", status=400)),
        path("__errors/403/", lambda r: render(r, "403.html", status=403)),
        path("__errors/404/", lambda r: render(r, "404.html", status=404)),
        path("__errors/410/", lambda r: render(r, "410.html", status=410)),
        path("__errors/500/", lambda r: render(r, "500.html", status=500)),
    ]
```

### prefix_default_language=True

| URL | Language | With flag | Without flag |
|---|---|---|---|
| `/about/` | English | ✅ works | Redirects |
| `/en/about/` | English | ✅ also works | ✅ works |
| `/it/about/` | Italian | ✅ works | ✅ works |

With `prefix_default_language=True`, every language gets the prefix for consistency.
Non-prefixed links get redirected.

---

## App-level URL Patterns

Each app defines its own `urls.py`. The `app_name` establishes a namespace.

```python
# apps/core/urls.py
from django.urls import path
from . import views

app_name = 'core'

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('work/', views.work, name='work'),
    path('contact/', views.contact, name='contact'),
    ...
]
```

### URL Namespacing in Templates

The namespace prevents collisions if two apps both define `name='index'`:

```html
<!-- Without namespace — ambiguous if multiple apps define 'home' -->
<a href="{% url 'home' %}">Home</a>

<!-- With namespace — unambiguous -->
<a href="{% url 'core:home' %}">Home</a>
```

Always use namespaced URLs in templates.

---

## Error Handlers

| Handler | View | Status |
|---|---|---|
| `handler400` | `apps.core.views.error_400` | Bad request |
| `handler403` | `apps.core.views.error_403` | Permission denied |
| `handler404` | `apps.core.views.error_404` | Not found |
| `handler429` | `apps.core.views.error_429` | Rate limited |
| `handler500` | `apps.core.views.error_500` | Server error |

Each renders a branded error page extending `4xx_base.html`.
