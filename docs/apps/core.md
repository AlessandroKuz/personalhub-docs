# App: Core

The main app. Contains all personal content: landing page, about, work/career, contact.

## Responsibility

- Phase 1 of the project
- No database models (content is in templates and context dictionaries)
- All views are async function views

## URL Patterns

| URL | View | Name |
|---|---|---|
| `/` | `home` | `core:home` |
| `/about/` | `about` | `core:about` |
| `/work/` | `work` | `core:work` |
| `/contact/` | `contact` | `core:contact` |

---

## Pages

### Landing (`/`)
TLDR of each section — each block links to its dedicated page.
Prominent CTA: "Contact me" → `/contact/`.
Sections: hero, about snippet, work/skills snippet, projects preview, blog preview, contact.

### About (`/about/`)
Personal description, background, languages spoken, interests, personality.

### Work/Career (`/work/`)
- Skills and capabilities overview
- Interactive education + career timeline (CSS/HTML, no model needed in phase 1)
- CV download (PDF served from `static/img/` or `static/files/`)

### Contact (`/contact/`)
- Email contact form (HTMX-powered — no full page reload on submit)
- cal.com scheduling link
- Social links: GitHub, LinkedIn

---

## HTMX Pattern: Contact Form

The form submits via HTMX. The server returns an HTML fragment — either the form with
inline errors, or a success confirmation — which HTMX swaps in without a page reload.

```python
# apps/core/views.py
from django.shortcuts import render
from asgiref.sync import sync_to_async
from .forms import ContactForm

async def contact(request):
    if request.method == 'POST' and request.htmx:
        form = ContactForm(request.POST)
        if await sync_to_async(form.is_valid)():
            await send_contact_email(form.cleaned_data)
            return render(request, 'partials/_contact_success.html')
        return render(request, 'partials/_contact_form.html', {'form': form})
    return render(request, 'core/contact.html', {'form': ContactForm()})
```

```html
<!-- In core/contact.html -->
<div id="contact-form-wrapper">
  {% include "partials/_contact_form.html" %}
</div>
```

```html
<!-- partials/_contact_form.html -->
<form hx-post="{% url 'core:contact' %}"
      hx-target="#contact-form-wrapper"
      hx-swap="outerHTML">
  {% csrf_token %}
  {{ form.as_div }}
  <button type="submit">Send</button>
</form>
```
