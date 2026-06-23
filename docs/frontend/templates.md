> Last updated: June 2026

# Templates

## Template Hierarchy

```
templates/
├── base.html                      # Master layout — extended by all pages
├── 4xx_base.html                  # Base layout for error pages
├── 400.html                       # Bad request
├── 403.html                       # Permission denied
├── 403_csrf.html                  # CSRF failure
├── 404.html                       # Not found
├── 410.html                       # Gone
├── 500.html                       # Server error
├── coming_soon.html               # Placeholder for future pages
├── robots.txt                     # Robots exclusion file
├── components/
│   ├── _nav.html                  # Navigation bar (included in base.html)
│   ├── _footer.html               # Footer (included in base.html)
│   ├── _alerts.html               # Global alert banners
│   ├── _toasts.html               # Bootstrap toast notifications
│   └── _shortcutsModal.html       # Vim keyboard shortcuts modal
└── partials/
    ├── _contact_form.html      # HTMX fragment: contact form
    ├── _contact_success.html   # HTMX fragment: post-submit confirmation
    ├── _project_grid.html      # HTMX fragment: project grid (filtered)
    └── _project_card.html      # HTMX fragment: single project card
```

**Naming convention:**

- `_underscore.html` prefix → partial or component, not a standalone page
- `page_name.html` → full page template (always extends `base.html`)
- Error pages extend `4xx_base.html`

---

## base.html

```html
{% load i18n static %}
<!DOCTYPE html>
<html lang="{{ LANGUAGE_CODE }}" data-bs-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}Alex — AI/ML Engineer{% endblock %}</title>

  <!-- Bootstrap 5 (CDN — no build step) -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3/dist/css/bootstrap.min.css"
        rel="stylesheet">

  <!-- Design tokens + overrides -->
  <link rel="stylesheet" href="{% static 'css/main.css' %}">

  {% block extra_css %}{% endblock %}
</head>
<body>
  {% include "components/_nav.html" %}

  <main>
    {% block content %}{% endblock %}
  </main>

  {% include "components/_toasts.html" %}
  {% include "components/_footer.html" %}
  {% include "components/_shortcutsModal.html" %}

  <!-- HTMX -->
  <script src="https://unpkg.com/htmx.org@2" defer></script>
  <!-- Theme toggle + micro-interactions -->
  <script src="{% static 'js/main.js' %}" defer></script>

  {% block extra_js %}{% endblock %}
</body>
</html>
```

---

## Template Blocks

| Block | Purpose |
|---|---|
| `title` | Page `<title>` tag |
| `content` | Main page body |
| `extra_css` | Page-specific stylesheets |
| `extra_js` | Page-specific scripts |

---

## Template Tags Reference

```html
{% load i18n %}             <!-- enable translation tags -->
{% load static %}           <!-- enable static file URL tags -->

{% trans "Hello" %}         <!-- translate a short string -->
{% url 'core:home' %}       <!-- reverse a namespaced URL -->
{% static 'css/main.css' %} <!-- resolve a static file URL -->
{% include "_nav.html" %}   <!-- include a component -->
{% block content %}{% endblock %}  <!-- define/override a block -->
```
