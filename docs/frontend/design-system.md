> Last updated: 25th March 2026

# Design System

## Principles

- Monochromatic with a single blue accent — intentionally minimalist
- CSS custom properties for every colour — never hardcoded values
- Two themes (light/dark) via `data-theme` attribute on `<html>`
- Bootstrap 5 for layout and components — no PostCSS/build step needed

---

## Colour Tokens

```css
/* static/css/main.css */

:root {
  --accent

  --color-bg
  --color-bg2
  --color-surface
  --color-text
  --color-muted
  --color-border
}
```

Every component uses these variables. Switching themes is a single attribute change on `<html>`.

---

## Theme Toggle

```javascript
// static/js/main.js
const toggle = document.getElementById('theme-toggle');
const html = document.documentElement;

// Restore saved preference
html.dataset.theme = localStorage.getItem('theme') || 'light';

toggle.addEventListener('click', () => {
    const next = html.dataset.theme === 'dark' ? 'light' : 'dark';
    html.dataset.theme = next;
    localStorage.setItem('theme', next);
});
```

Via a simple javascript change, the theme is updated dynamically.

```javascript
(function () {
  var s = localStorage.getItem("theme") || "auto";
  var t = s === "auto"
    ? (window.matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light")
    : s;
  document.documentElement.setAttribute("data-theme", t);
  document.documentElement.setAttribute("data-bs-theme", t);
})();
```

Theme initialization in `base.html` guarantees no theme flash.

---

## Typography

System font stack — custom fonts for tailored experience:

```scss
$font-family-sans-serif: ...;

$font-family-monospace: ...;
```

Defined inside the scss, recalled in css, ensuring consistency.

---

## Responsive Strategy

Bootstrap 5's grid. Mobile-first — styles apply upward from the breakpoint:

| Breakpoint | Class prefix | Applies from |
|---|---|---|
| Mobile | `col-` | 0px |
| Tablet | `col-md-` | 768px |
| Desktop | `col-lg-` | 992px |
| Wide | `col-xl-` | 1200px |

```html
<!-- 1 col on mobile, 2 on tablet, 3 on desktop -->
<div class="col-12 col-md-6 col-lg-4">...</div>
```

---

## Custom CSS for design tuning

Extra CSS files per page are used to improve on the base design.
