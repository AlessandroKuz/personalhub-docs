> Last updated: 25th March 2026

# App: Blog

Phase 3. Writing, publishing, and managing blog posts with a lightweight editor.

!!! info
    Example code - NOT YET IMPLEMENTED

## Responsibility

- Activated in Phase 3
- `Post` model managed through Django admin and a custom editor view
- WebSocket-powered live Markdown preview (this is where ASGI visibly pays off)

---

## Models

```python
# apps/blog/models.py

class Post(models.Model):
    class Status(models.TextChoices):
        DRAFT = 'draft', 'Draft'
        PUBLISHED = 'published', 'Published'

    title = models.CharField(max_length=300)
    slug = models.SlugField(unique=True)
    body = models.TextField()               # Markdown source
    status = models.CharField(
        max_length=10,
        choices=Status,
        default=Status.DRAFT
    )
    published_at = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-published_at']

    def __str__(self):
        return self.title
```

---

## URL Patterns

| URL | View | Name |
|---|---|---|
| `/blog/` | `PostListView` | `blog:list` |
| `/blog/<slug>/` | `PostDetailView` | `blog:detail` |
| `/blog/editor/` | `EditorView` | `blog:editor` (staff only) |

---

## Writing Interface

The editor view uses **EasyMDE** — a browser-based Markdown editor loaded via CDN.
No build step, no npm. The editor sends Markdown source to a WebSocket endpoint;
the server renders it to HTML and pushes it back to the preview pane in real time.

```html
<!-- Load EasyMDE via CDN — no build step -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/easymde/dist/easymde.min.css">
<script src="https://cdn.jsdelivr.net/npm/easymde/dist/easymde.min.js"></script>
```

---

## ASGI Payoff: WebSocket Live Preview

This is the concrete moment where running ASGI from day 1 pays off.

A WebSocket keeps a persistent, bidirectional connection open between the editor
and the server. Every keystroke sends the Markdown source; the server renders it to
HTML and pushes it back to the preview pane. Under WSGI, this is impossible — HTTP
is request/response only, with no persistent connection.

```python
# Requires Django Channels or a custom ASGI consumer
# Planned for Phase 3 implementation
```

---

## Publishing Flow

1. Create a `Post` in Django admin or the custom editor view (status: `DRAFT`)
2. Write in Markdown with live preview
3. Set `status = PUBLISHED` and `published_at = now()`
4. Post appears on `/blog/` immediately
