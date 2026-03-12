# Design System

## Principles

- Monochromatic with a single red accent — intentional, not unfinished
- CSS custom properties for every colour — never hardcoded values
- Two themes (light/dark) via `data-theme` attribute on `<html>`
- Bootstrap 5 for layout and components — no PostCSS/build step needed

---

## Colour Tokens

```css
/* static/css/main.css */

:root {
    --color-bg:       #f8f7f4;   /* warm white — easier on eyes than pure #fff */
    --color-surface:  #efefec;   /* cards, subtle section backgrounds */
    --color-text:     #1a1a1a;
    --color-muted:    #6b6b6b;
    --color-accent:   #c0392b;   /* red — works on light backgrounds */
    --color-border:   #e0ddd8;
}

[data-theme="dark"] {
    --color-bg:       #141414;
    --color-surface:  #1e1e1e;
    --color-text:     #e8e6e1;
    --color-muted:    #888;
    --color-accent:   #e74c3c;   /* slightly lighter red for dark backgrounds */
    --color-border:   #2a2a2a;
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

---

## Typography

System font stack — no external font load, no flash of unstyled text, instant render:

```css
body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
                 "Helvetica Neue", Arial, sans-serif;
}
```

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
