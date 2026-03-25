> Last updated: 25th March 2026

# Testing & TDD

## Philosophy: Why Tests First

TDD — Test-Driven Development — is not primarily a testing strategy. It's a **design strategy
that happens to produce tests**. The cycle is:

```
1. Write a failing test  (Red)
2. Write the minimum code to make it pass  (Green)
3. Refactor with confidence  (Refactor)
```

The discipline of writing the test *before* the implementation forces you to think about
the interface and expected behaviour of your code before you think about how to build it.
This has two concrete consequences:

- You only write code that is actually needed (tests define the requirements)
- Every piece of code is testable by definition, because you wrote the test first

In this project, TDD is non-negotiable. **No production code is written without a
failing test that justifies it.** If you find yourself writing a view or a model without
a test already failing, stop and write the test first.

---

## The Testing Pyramid

The testing pyramid is a mental model for how to proportion your test suite:

```
        /\
       /  \    E2E Tests (few, slow, expensive)
      /    \   "Does the whole thing work?"
     /──────\
    /        \  Integration Tests (some)
   /          \  "Do the parts work together?"
  /────────────\
 /              \  Unit Tests (many, fast, cheap)
/                \  "Does this function do what it says?"
──────────────────
```

For a Django portfolio site, the practical breakdown is:

**Unit tests** — individual view functions, form validation logic, model methods.
These are the majority of your test suite. Fast, isolated, no database.

**Integration tests** — views that hit the database (with a test database), URL
routing, template rendering. In Django, these are still called "unit tests" loosely,
but they exercise more moving parts.

**End-to-end tests** — full browser tests (Playwright, Selenium). Not implemented
in this project at this stage.

---

## Framework Choice

### The Alternatives

| Framework | Style | Async support | Django integration | Notes |
|---|---|---|---|---|
| `unittest` (stdlib) | Class-based, `self.assertEqual` | Manual, via `asyncio.run()` | Via `django.test.TestCase` | Built in, no install |
| **`pytest`** | Function-based, plain `assert` | Via `pytest-asyncio` | Via `pytest-django` | Industry standard |
| `nose2` | `unittest`-compatible runner | Minimal | Manual | Effectively unmaintained |
| `hypothesis` | Property-based testing | Compatible with pytest | Compatible | Complementary, not a replacement |

### Why `pytest` + `pytest-django` + `pytest-asyncio`

Django ships with a test runner based on Python's `unittest`. It works. But the Python
ecosystem has converged on `pytest` as the standard — used by Django itself, FastAPI,
Pydantic, SQLAlchemy, and virtually every major Python project.

The three deciding factors for this project:

**1. Plain `assert` with intelligent failure output**

```python
# unittest
self.assertEqual(response.status_code, 200)
self.assertIn('core/home.html', [t.name for t in response.templates])

# pytest
assert response.status_code == 200
assert 'core/home.html' in [t.name for t in response.templates]
```

When a pytest assertion fails, it introspects the values and shows you *exactly*
what was on each side. `assertEqual` just tells you they weren't equal.

**2. Fixtures — composable test setup**

`pytest` fixtures are functions that provide test dependencies via dependency injection.
`pytest-django` ships fixtures like `client`, `async_client`, `db`, `rf` (request
factory) out of the box. You declare what a test needs, pytest provides it:

```python
async def test_home(async_client):      # async_client injected by pytest-django
    response = await async_client.get('/')
    assert response.status_code == 200
```

No `setUp`/`tearDown` class methods, no inheritance chain.

**3. `pytest-asyncio` is essential for async views**

This project uses `async` views from day one (see [Async & ASGI](../architecture/async.md)).
Testing async views with `unittest` requires wrapping everything in `asyncio.run()` or
a custom `TestCase` subclass. With `pytest-asyncio` and `asyncio_mode = "auto"`:

```python
# This just works. No decorator, no ceremony.
async def test_home_view(async_client):
    response = await async_client.get('/')
    assert response.status_code == 200
```

### Installed packages

```toml
# pyproject.toml — [dependency-groups] dev section
[dependency-groups]
dev = [
    "pytest",
    "pytest-django",
    "pytest-asyncio",
    "pytest-cov",
    ...
]
```

```bash
uv add --dev pytest pytest-django pytest-asyncio pytest-cov
```

### `pyproject.toml` configuration

```toml
[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.dev"
asyncio_mode = "auto"
python_files = ["tests.py", "test_*.py"]
```

`asyncio_mode = "auto"` — every `async def test_*` function automatically runs
on the asyncio event loop. Without this you'd need `@pytest.mark.asyncio` on every
single async test. Given that all views are async, this eliminates a lot of noise.

`DJANGO_SETTINGS_MODULE` — tells pytest-django which settings file to use when
bootstrapping Django. Points to `dev` settings: SQLite (no Postgres needed in CI),
`DEBUG=True`, no HTTPS redirects.

---

## File Structure

Tests live **inside each app**, in a `tests/` sub-package. This is the Django convention
and it has a specific rationale: tests are part of the app, not a separate concern.

```
apps/
├── core/
│   ├── __init__.py
│   ├── apps.py
│   ├── views.py
│   ├── urls.py
│   └── tests/
│       ├── __init__.py       ← makes tests/ a Python package
│       ├── test_urls.py      ← URL resolution tests
│       └── test_views.py     ← view response tests
│
├── projects/
│   ├── models.py
│   └── tests/
│       ├── __init__.py
│       ├── test_models.py    ← model validation, methods
│       ├── test_urls.py
│       └── test_views.py
│
└── blog/
    └── tests/
        └── ...
```

### Why not a top-level `tests/` directory?

A top-level `tests/` directory is common in non-Django Python projects. For Django apps,
co-locating tests with the app they test has a concrete advantage: when you delete or
archive an app, its tests go with it. There's no orphaned `tests/projects/` directory
left behind pointing at code that no longer exists.

This also aligns with how Django's own test suite is structured and how most
well-maintained Django packages organise their tests.

### File naming convention

| File | What it tests |
|---|---|
| `test_urls.py` | URL reverse resolution, namespacing, i18n prefixes |
| `test_views.py` | HTTP responses: status codes, templates used, context |
| `test_models.py` | Model validation, custom methods, `__str__`, constraints |
| `test_forms.py` | Form validation logic, cleaned data, error messages |

---

## How pytest-django Works

### The `db` fixture and database access

By default, pytest-django tests have **no database access**. Attempting to hit the ORM
raises an error. You must declare that a test needs the database:

```python
@pytest.mark.django_db
def test_something_with_db():
    ...

# Or for async tests:
@pytest.mark.django_db
async def test_something_async(async_client):
    ...
```

For a whole file, add a module-level mark:

```python
pytestmark = pytest.mark.django_db
```

!!! note
    For Phase 1 (`apps.core`), all views are template-only with no ORM calls.
    Most tests don't need `@pytest.mark.django_db` at all — they just test HTTP
    responses. This means the test suite runs entirely in memory, with no database
    setup overhead.

### Key fixtures provided by pytest-django

| Fixture | Type | What it provides |
|---|---|---|
| `client` | sync | `django.test.Client` — makes sync HTTP requests |
| `async_client` | async | `django.test.AsyncClient` — makes async HTTP requests |
| `rf` | sync | `RequestFactory` — builds `HttpRequest` objects directly |
| `settings` | — | Modifies Django settings for a single test, restored after |
| `db` | — | Grants database access to a test |
| `django_db_setup` | — | Customise DB setup (rarely needed) |

### `conftest.py` — shared fixtures

`conftest.py` files are pytest's mechanism for sharing fixtures across multiple test
files. A `conftest.py` in an app's `tests/` directory makes its fixtures available
to all test files in that directory. A `conftest.py` at the project root makes
fixtures available project-wide.

```
personalhub/
├── conftest.py           ← project-wide fixtures (e.g. authenticated user)
└── apps/
    └── core/
        └── tests/
            └── conftest.py  ← core-specific fixtures (if needed)
```

---

## Test Patterns in This Project

### URL resolution tests

URL tests verify two things: that `reverse()` produces the right path for the default
language, and that `i18n_patterns` correctly prefixes it for non-default languages.

```python
# apps/core/tests/test_urls.py
import pytest
from django.conf import settings
from django.urls import reverse
from django.utils import translation


@pytest.mark.parametrize("url_name, expected_url", [
    ("core:home",    "/"),
    ("core:about",   "/about/"),
    ("core:work",    "/work/"),
    ("core:contact", "/contact/"),
])
def test_core_urls_resolve(url_name: str, expected_url: str) -> None:
    # Default language (EN) — no prefix because prefix_default_language=False
    assert reverse(url_name) == expected_url

    # All other languages — should have the language code prefix
    for language_code, _ in settings.LANGUAGES:
        if language_code == settings.LANGUAGE_CODE:
            continue
        with translation.override(language_code):
            assert reverse(url_name) == f"/{language_code}{expected_url}"
```

Why test URL resolution separately from views? Because a misconfigured `app_name`,
a missing `i18n_patterns` entry, or a wrong `prefix_default_language` value will
break URL resolution silently — your views might exist, but `{% url 'core:home' %}`
in templates will raise `NoReverseMatch` in production. The URL test catches this
before the view test even runs.

### View tests

View tests make HTTP requests and assert on the response. The `async_client` fixture
from `pytest-django` handles the async/await machinery.

```python
# apps/core/tests/test_views.py
import pytest
from django.test import AsyncClient
from django.conf import settings
from django.urls import reverse
from django.utils import translation


@pytest.mark.parametrize("url_name, expected_template", [
    ("core:home",    "core/home.html"),
    ("core:about",   "core/about.html"),
    ("core:work",    "core/work.html"),
    ("core:contact", "core/contact.html"),
])
async def test_core_views(
    async_client: AsyncClient,
    url_name: str,
    expected_template: str,
) -> None:
    # Default language
    await _assert_page(async_client, reverse(url_name), expected_template)

    # Non-default languages
    for language_code, _ in settings.LANGUAGES:
        if language_code == settings.LANGUAGE_CODE:
            continue
        with translation.override(language_code):
            url = reverse(url_name)
        await _assert_page(async_client, url, expected_template)


async def _assert_page(
    async_client: AsyncClient,
    url: str,
    expected_template: str,
) -> None:
    response = await async_client.get(url)
    assert response.status_code == 200
    assert expected_template in [t.name for t in response.templates]
```

!!! note "Why `_assert_page` is a helper, not a fixture"
    It's a plain async function extracted to avoid repeating the same two assertions.
    Fixtures are for *setup* (providing objects). Helpers are for *shared assertion
    logic*. Don't reach for a fixture when a plain function will do.

### Model tests (Phase 2+)

```python
# apps/projects/tests/test_models.py
import pytest
from apps.projects.models import Project, Tag


@pytest.mark.django_db
def test_project_str():
    tag = Tag.objects.create(name="PyTorch", slug="pytorch")
    project = Project.objects.create(
        title="Anomaly Detector",
        slug="anomaly-detector",
        description="...",
    )
    project.tags.add(tag)
    assert str(project) == "Anomaly Detector"


@pytest.mark.django_db
def test_featured_project_queryset():
    Project.objects.create(title="A", slug="a", description="", featured=True)
    Project.objects.create(title="B", slug="b", description="", featured=False)
    featured = Project.objects.filter(featured=True)
    assert featured.count() == 1
    assert featured.first().title == "A"
```

---

## Running Tests

### Basic run

```bash
uv run pytest
```

### Verbose (show test names)

```bash
uv run pytest -v
```

### Run a specific app

```bash
uv run pytest apps/core/
```

### Run a specific file

```bash
uv run pytest apps/core/tests/test_views.py
```

### Run a specific test

```bash
uv run pytest apps/core/tests/test_views.py::test_core_views
```

### With coverage report

```bash
uv run pytest --cov=apps --cov-report=term-missing
```

The `--cov-report=term-missing` flag shows which lines are not covered by tests,
printed inline in the terminal. This is the fastest feedback loop for identifying gaps.

---

## Current Test Inventory

### `apps/core` — 8 tests, all passing

| Test | File | What it checks |
|---|---|---|
| `test_core_urls_resolve[core:home-/]` | `test_urls.py` | `reverse('core:home')` → `/` |
| `test_core_urls_resolve[core:about-/about/]` | `test_urls.py` | `/about/` and `/it/about/` |
| `test_core_urls_resolve[core:work-/work/]` | `test_urls.py` | `/work/` and `/it/work/` |
| `test_core_urls_resolve[core:contact-/contact/]` | `test_urls.py` | `/contact/` and `/it/contact/` |
| `test_core_views[core:home-core/home.html]` | `test_views.py` | 200, correct template |
| `test_core_views[core:about-core/about.html]` | `test_views.py` | 200, correct template |
| `test_core_views[core:work-core/work.html]` | `test_views.py` | 200, correct template |
| `test_core_views[core:contact-core/contact.html]` | `test_views.py` | 200, correct template |

---

## What's Not Tested Yet (and Why)

| What | Why not yet |
|---|---|
| Contact form submission | Form logic not yet implemented (Phase 1 in progress) |
| HTMX partial responses | No HTMX views written yet |
| `apps.projects` models | Phase 2 — models not yet written |
| `apps.blog` models | Phase 3 |
| i18n translation strings | Content not yet written — no `.po` files to validate |
| Authentication | No auth-protected views in scope |

Tests will be written **before** each of these features is implemented, per TDD.
