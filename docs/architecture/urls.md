# URL Structure

## The URL Tree

Django's URL routing is a tree. `config/urls.py` is the root; each app's `urls.py`
is a branch. A request walks the tree until a pattern matches.

```
config/urls.py  (root)
├── admin/
├── i18n/                    ← language switcher POST endpoint
├── [i18n_patterns]
│   ├── /  →                 apps.core.urls
│   ├── projects/  →         apps.projects.urls
│   └── blog/  →             apps.blog.urls
└── __debug__/               ← debug toolbar (dev only, import inside if DEBUG)
```

---

## Root URL Configuration

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

urlpatterns += i18n_patterns(
    path('', include('apps.core.urls')),
    path('projects/', include('apps.projects.urls')),
    path('blog/', include('apps.blog.urls')),
    prefix_default_language=False,
)

if settings.DEBUG:
    from debug_toolbar.toolbar import debug_toolbar_urls
    urlpatterns += debug_toolbar_urls()
    # Import is INSIDE the if block — critical.
    # A top-level import crashes prod because debug_toolbar is not installed there.
```

### prefix_default_language=False

| URL | Language | With flag | Without flag |
|---|---|---|---|
| `/about/` | English | ✅ works | ❌ 404 |
| `/en/about/` | English | ✅ also works | ✅ works |
| `/it/about/` | Italian | ✅ works | ✅ works |

With `prefix_default_language=False`, English (the default) gets clean URLs.
Non-default languages always have a prefix.

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
