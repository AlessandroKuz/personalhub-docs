> Last updated: 25th March 2026

# HTMX Patterns

## What HTMX Does

HTMX lets HTML elements make HTTP requests and swap parts of the DOM with the response.
**The server always returns HTML — never JSON.** No custom JavaScript needed for
the request/response cycle.

```html
<!-- This button makes a GET request and replaces #result with the response body -->
<button hx-get="/api/something/"
        hx-target="#result"
        hx-swap="innerHTML">
  Load
</button>
```

---

## Core Attributes

| Attribute | Purpose |
|---|---|
| `hx-get` / `hx-post` | HTTP method + URL to request |
| `hx-target` | CSS selector of the element to update |
| `hx-swap` | How to insert the response (see below) |
| `hx-trigger` | What triggers the request (default: `click` for buttons, `submit` for forms) |
| `hx-indicator` | Show a loading element while request is in flight |
| `hx-push-url` | Update the browser URL bar (makes state bookmarkable) |

### hx-swap values

| Value | Effect |
|---|---|
| `innerHTML` | Replace the target's content |
| `outerHTML` | Replace the target element itself |
| `afterend` | Insert after the target |
| `beforeend` | Append inside the target |

---

## Detecting HTMX Requests in Django

`django-htmx` adds `request.htmx` — truthy when the request came from HTMX.
This lets one URL serve both full-page loads and partial HTMX updates:

```python
async def project_list(request):
    projects = await Project.objects.all().alist()

    if request.htmx:
        # Return only the grid fragment
        return render(request, 'partials/_project_grid.html', {'projects': projects})

    # Return the full page (with nav, footer, etc.)
    return render(request, 'projects/list.html', {'projects': projects})
```

---

## Pattern: Contact Form

The form submits without a page reload. On success the form is replaced by a
confirmation message. On error the form re-renders with inline validation errors.

```html
<!-- core/contact.html -->
<div id="contact-wrapper">
  {% include "partials/_contact_form.html" %}
</div>
```

```html
<!-- partials/_contact_form.html -->
<form hx-post="{% url 'core:contact' %}"
      hx-target="#contact-wrapper"
      hx-swap="outerHTML">
  {% csrf_token %}
  {{ form.as_div }}
  <button type="submit">Send</button>
</form>
```

```html
<!-- partials/_contact_success.html -->
<div id="contact-wrapper">
  <p>Message sent. I'll get back to you soon.</p>
</div>
```

---

## Pattern: Tag Filtering

```html
<!-- Tag buttons -->
{% for tag in tags %}
<button hx-get="{% url 'projects:list' %}?tag={{ tag.slug }}"
        hx-target="#project-grid"
        hx-swap="innerHTML"
        hx-push-url="true">
  {{ tag.name }}
</button>
{% endfor %}

<!-- Target grid -->
<div id="project-grid">
  {% include "partials/_project_grid.html" %}
</div>
```

`hx-push-url="true"` updates the browser URL to `/projects/?tag=django` — the filtered
view becomes bookmarkable and shareable without any extra JavaScript.

---

## CSRF with HTMX

HTMX respects Django's CSRF protection automatically when using `{% csrf_token %}` inside
forms. For non-form HTMX requests (e.g. `hx-delete`), add the CSRF token to all requests
via a meta tag + one line in `main.js`:

```html
<!-- base.html -->
<meta name="csrf-token" content="{{ csrf_token }}">
```

```javascript
// main.js
document.body.addEventListener('htmx:configRequest', (event) => {
    event.detail.headers['X-CSRFToken'] =
        document.querySelector('meta[name="csrf-token"]').content;
});
```
