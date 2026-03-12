# Components

## Philosophy

Every visible UI element that appears in more than one place is a **component** — a
self-contained, reusable template fragment. Nothing is duplicated. If a design change
is needed, it happens in one file and propagates everywhere automatically.

This is Django's `{% include %}` tag put to disciplined use.

```
templates/
├── base.html               # master layout — orchestrates everything below
├── components/             # structural UI — present on most/all pages
│   ├── _nav.html
│   ├── _footer.html
│   ├── _lang_switcher.html
│   ├── _theme_toggle.html
│   └── _cta.html
└── partials/               # content fragments — often HTMX targets
    ├── _project_card.html
    ├── _project_grid.html
    ├── _blog_card.html
    ├── _contact_form.html
    └── _contact_success.html
```

**`components/`** — structural chrome that wraps every page. Included statically in
`base.html`. Never an HTMX target.

**`partials/`** — content fragments that can be rendered standalone. These are what
HTMX requests return. A `GET /projects/?tag=django` response is just
`_project_grid.html`, not a full page.

---

## Component Inventory

### `_nav.html`
The navigation bar. Included once in `base.html`. Contains:
- Site logo / name (links to `core:home`)
- Navigation links (About, Work, Projects, Blog)
- Language switcher (`_lang_switcher.html`)
- Theme toggle (`_theme_toggle.html`)

Keeping nav isolated means restyling or adding a link touches exactly one file.

### `_footer.html`
Site footer. Included once in `base.html`. Contains:
- Social links (GitHub, LinkedIn)
- Copyright line
- Secondary navigation if needed

### `_lang_switcher.html`
The EN/IT language selector. A small `<form>` that POSTs to Django's built-in
`set_language` endpoint. Isolated so it can be placed in the nav, footer, or
anywhere else without duplication.

### `_theme_toggle.html`
A single button that flips `data-theme` on `<html>`. Isolated for the same reason
as the language switcher — placement flexibility, zero duplication.

### `_cta.html`
The "Contact me" call-to-action block. Appears at the bottom of the landing page
and at the end of the About and Work pages. One file, three placements:

```html
{% include "components/_cta.html" %}
```

If the copy, styling, or destination URL ever changes, it changes once.

---

## Content Partials

### `_project_card.html`
Renders a single `Project` instance — title, description snippet, tag chips,
GitHub/live links. Used by `_project_grid.html`.

```html
<!-- Expects a `project` variable in context -->
<div class="card">
  <h3>{{ project.title }}</h3>
  <p>{{ project.description }}</p>
  {% for tag in project.tags.all %}
    <span class="badge">{{ tag.name }}</span>
  {% endfor %}
</div>
```

### `_project_grid.html`
Loops over a `projects` queryset and renders a `_project_card.html` for each.
This is the **HTMX target** for tag filtering — the entire grid is swapped in one
operation without touching the nav, filters, or page heading.

```html
{% for project in projects %}
  {% include "partials/_project_card.html" %}
{% empty %}
  <p>No projects found.</p>
{% endfor %}
```

### `_blog_card.html`
The blog post preview — title, published date, opening paragraph or excerpt, and
a "Read more" link. The equivalent of `_project_card.html` for posts. Each entry
on the `/blog/` listing page is one of these.

### `_contact_form.html` / `_contact_success.html`
The two states of the contact form interaction. The form renders by default;
on successful submission HTMX swaps it for the success confirmation. Both are
partials so they can be returned directly from a view without a full page render.

---

## The `{% include %}` + Context Pattern

Django's `{% include %}` passes the current template context down by default.
For cases where you need to pass specific data, use the `with` keyword:

```html
<!-- Pass a specific object -->
{% include "partials/_project_card.html" with project=featured_project %}

<!-- Pass data and isolate context (only the passed vars are available) -->
{% include "partials/_project_card.html" with project=p only %}
```

The `only` keyword is useful for partials that should be fully self-contained —
it prevents accidental dependence on whatever happens to be in the parent context.

---

## Why This Matters Long-Term

Without this discipline, a typical growth path looks like:

```
Week 1:  Copy-paste the nav into home.html, about.html, work.html
Week 8:  Add a new nav link — edit 6 files
Week 12: Redesign the project card — hunt down every place it's used
Week 16: The footer on the contact page is slightly different and nobody knows why
```

With components:

```
Week 1:  Write _nav.html once, include it in base.html
Week 8:  Add a new nav link — edit 1 file
Week 12: Redesign the project card — edit 1 file
Week 16: The footer is identical everywhere because there is only one footer
```

The upfront cost is naming discipline and one extra `{% include %}` tag.
The long-term payoff is a codebase where changes are local and auditable.
