> Last updated: 25th March 2026

# App: Projects

Phase 2. Manages and displays portfolio projects dynamically via Django models and admin.

!!! info
    Example code - NOT YET IMPLEMENTED

## Responsibility

- Activated in Phase 2
- `Project` and `Tag` models managed through Django admin
- HTMX-powered tag filtering without full page reloads

---

## Models

```python
# apps/projects/models.py

class Tag(models.Model):
    name = models.CharField(max_length=50)
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.name


class Project(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    description = models.TextField()        # short — used in cards
    body = models.TextField()               # long write-up — used in detail view
    tags = models.ManyToManyField(Tag, blank=True)
    github_url = models.URLField(blank=True)
    live_url = models.URLField(blank=True)
    featured = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return self.title
```

---

## URL Patterns

| URL | View | Name |
|---|---|---|
| `/projects/` | `ProjectListView` | `projects:list` |
| `/projects/<slug>/` | `ProjectDetailView` | `projects:detail` |

---

## HTMX Pattern: Tag Filtering

Clicking a tag makes a GET request with a query parameter. The server detects it's an
HTMX request and returns only the project grid fragment, not the full page.

```html
<!-- Tag filter buttons -->
<button hx-get="{% url 'projects:list' %}?tag={{ tag.slug }}"
        hx-target="#project-grid"
        hx-swap="innerHTML"
        hx-push-url="true">
  {{ tag.name }}
</button>

<!-- Target container -->
<div id="project-grid">
  {% include "partials/_project_grid.html" %}
</div>
```

```python
# apps/projects/views.py
async def project_list(request):
    tag_slug = request.GET.get('tag')
    projects = Project.objects.all()
    if tag_slug:
        projects = projects.filter(tags__slug=tag_slug)
    projects = await projects.alist()

    if request.htmx:
        return render(request, 'partials/_project_grid.html', {'projects': projects})
    return render(request, 'projects/list.html', {'projects': projects})
```

`hx-push-url="true"` updates the browser's URL to `/projects/?tag=django`, making the
filtered state bookmarkable and shareable.
