> Last updated: 25th March 2026

# Templates

## Template Hierarchy

```
templates/
├── base.html                   # Master layout — extended by all pages
├── components/
│   ├── _nav.html               # Navigation bar (included in base.html)
│   ├── _footer.html            # Footer (included in base.html)
│   └── _cta.html               # "Contact me" call-to-action block
└── partials/
    ├── _contact_form.html      # HTMX fragment: contact form
    ├── _contact_success.html   # HTMX fragment: post-submit confirmation
    ├── _project_grid.html      # HTMX fragment: project grid (filtered)
    └── _project_card.html      # HTMX fragment: single project card
```

**Naming convention:**

- `_underscore.html` prefix → partial or component, not a standalone page
- `page_name.html` → full page template (always extends `base.html`)

---

## base.html

```html
{% load i18n static %}
<!DOCTYPE html>
<html lang="{{ LANGUAGE_CODE }}" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}Alex — ML Engineer{% endblock %}</title>

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

  {% include "components/_footer.html" %}

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
{% include "_nav.html" %}   <!-- include a partial -->
{% block content %}{% endblock %}  <!-- define/override a block -->
```
