> Last updated: June 2026

# Design System

**Alessandro Kuz personal brand.** Data Scientist & AI/ML Engineer. Minimal,
editorial, precision-first. Sharp corners, near-neutral dual-mode palette, single
blue accent, JetBrains Mono as the technical voice for all metadata — DM Sans for
all human-readable prose.

---

## 1. Principles

- **Monochromatic + single accent** — no palette sprawl. One blue. Everything else
  is a shade of near-black or near-white.
- **Warm off-tones** — never pure `#000` or `#fff`. Warm-shifted backgrounds, cool
  dark mode. Prevents sterile/clinical feel.
- **CSS custom properties everywhere** — no hardcoded hex outside `:root`. Theme
  switching is one `data-bs-theme` attribute change.
- **JetBrains Mono = technical voice** — every label, tag, nav link, button,
  eyebrow, stat label uses the mono font. DM Sans is reserved for prose: headings,
  descriptions, body copy.
- **2px radius** — sharp and editorial. Intentionally anti-rounded. Bootstrap's
  default (6px) overridden at SCSS compile time.
- **Hover = darken, not color** — interactive surfaces darken on hover; color only
  appears on active/focus states. Single-accent discipline.

---

## 2. Color Tokens

All tokens defined in `static/css/main.css` (or `uploads/main.css` in the design-system repo) under `:root` and `[data-bs-theme="dark"]`.

```css
:root {
  /* Backgrounds */
  --color-bg:      #f8f7f4;   /* warm off-white — page background */
  --color-bg2:     #ececec;   /* alternate section background */
  --color-surface: #efefec;   /* cards, hero card, code blocks */

  /* Text */
  --color-text:    #1a1a1a;   /* primary text — warm, not pure black */
  --color-muted:   #6b6b6b;   /* secondary text, labels, metadata */

  /* Structure */
  --color-border:  #e0ddd8;   /* all dividers, borders, grid gaps */

  /* Accent */
  --accent:        var(--bs-primary);  /* resolves to #275DAD */

  /* Functional */
  --green:         #4ade80;   /* status "available" dot only */
}

[data-bs-theme="dark"] {
  --color-bg:      #141414;
  --color-bg2:     #1a1a1a;
  --color-surface: #1e1e1e;
  --color-text:    #e8e6e1;
  --color-muted:   #888888;
  --color-border:  #2a2a2a;
}
```

**Bootstrap primary** (`--bs-primary`) is set in `static/scss/custom.scss` (or `uploads/custom.scss` in the design-system repo):

```scss
$primary: #275DAD;
```

**Accent hover** — defined as `--accent-hover: color-mix(in srgb, var(--accent) 85%, black)` — no hardcoded hex. Applied to button hover states, icon hovers, and any accent-surface interaction. Tracks `--accent` automatically — if `--accent` ever changes, hover follows.

**Extra Bootstrap colors** merged into the theme map:

```scss
$custom-colors: (
  "alt-light": #f0f4f8,
  "alt-dark":  #1a2433,
);
```

### Usage rules

| Rule | Detail |
|---|---|
| Never hardcode hex | Always use `--color-*` or `var(--accent)` |
| No pure black | `#000000` never appears anywhere |
| No pure white | `#ffffff` never appears anywhere |
| Accent = signalling color | `--accent` for interactive states, emphasis, hairlines; `--accent-hover` for hover on accent surfaces |
| Accent hover | `--accent-hover: color-mix(in srgb, var(--accent) 85%, black)` — auto-tracks `--accent`, no hardcoded hex |
| Green = status only | `--green` reserved for the "available" pulse dot |

---

## 3. Typography

### Fonts

Loaded via Google Fonts in `base.html`:

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600;700&family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;1,9..40,300&display=swap" rel="stylesheet">
```

Injected into Bootstrap's compile-time variables via `static/scss/custom.scss`:

```scss
$font-family-sans-serif:
  "DM Sans", system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;

$font-family-monospace:
  "JetBrains Mono", "Fira Code", "Cascadia Code", "SF Mono", "Roboto Mono",
  ui-monospace, monospace;
```

Referenced in CSS as `var(--font-sans)` and `var(--font-mono)` (aliases to
Bootstrap's compiled vars).

### Scale

| Role | Font | Weight | Size | Tracking | Transform |
|---|---|---|---|---|---|
| Hero name | DM Sans | 700 | `clamp(3.2rem, 7vw, 6rem)` | `-0.02em` | — |
| Section title | DM Sans | 400 | `clamp(2rem, 4vw, 3rem)` | `-0.02em` | — |
| Hero description | DM Sans | 300 | `0.95rem` | normal | — |
| Body prose | DM Sans | 400 | `0.95rem` | normal | — |
| About facts value | DM Sans | 400 | `0.95rem` | normal | — |
| Stat value | DM Sans | 600 | `1.5rem` | 0 | — |
| Eyebrow / section label | JetBrains Mono | 400 | `0.65rem` | `0.20em` | uppercase |
| Nav links | JetBrains Mono | 400 | `small` | `0.10em` | uppercase |
| Buttons | JetBrains Mono | 400 | `0.72rem` | `0.12em` | uppercase |
| Tags / badges | JetBrains Mono | 400 | `0.58–0.62rem` | `0.10em` | uppercase |
| Stat label | JetBrains Mono | 400 | `0.65rem` | `0.12em` | uppercase |
| Status row | JetBrains Mono | 400 | `0.70rem` | `0.10em` | uppercase |
| Process step label | JetBrains Mono | 400 | `0.60rem` | `0.15em` | uppercase |
| Skill icon label | JetBrains Mono | 400 | `0.72rem` | `0.10em` | — |
| Contact meta | JetBrains Mono | 400 | `0.65rem` | `0.10em` | — |
| Social links | JetBrains Mono | 400 | `0.70rem` | `0.10em` | uppercase |
| Marquee items | JetBrains Mono | 400 | `0.65rem` | `0.20em` | uppercase |

### Rules

- `font-style: italic` + `color: var(--accent)` marks emphasis in section titles
  (`.section-title em`)
- `strong` in body copy: `color: var(--color-text)`, `font-weight: 400` — weight
  stays light, color lifts it
- Line heights: prose `1.85–1.9`, mono labels `1.0–1.2`, body descriptions `1.75`

---

## 4. Spacing & Layout

### Design tokens

```css
--navbar-height: 45px;
--footer-height: 118px;
```

### Section rhythm

```css
.content-section {
  padding: clamp(64px, 10vh, 120px) 0;
}
```

Full-viewport sections (hero, about, work, projects, process, contact):

```css
min-height: calc(100vh - var(--navbar-height));
```

### Grid patterns

| Pattern | CSS |
|---|---|
| Hero 2-col | `grid-template-columns: 1fr 420px; gap: 80px` |
| About 2-col | `grid-template-columns: 1fr 1fr; gap: 80px` |
| Skills 3-col | `grid-template-columns: repeat(3, 1fr); gap: 1px; background: var(--color-border)` |
| Projects 2-col | `grid-template-columns: 1fr 1fr; gap: 1px; background: var(--color-border)` |
| Featured card | `grid-column: span 2; grid-template-columns: 1fr 1fr; gap: 48px` |
| Process 4-col | `grid-template-columns: repeat(4, 1fr); border: 1px solid var(--color-border)` |

The 1px grid gap technique: set `background` on the grid container to
`--color-border`, set `background` on each cell to `--color-bg` or `--color-bg2`.
The container color bleeds through as the border.

### Responsive breakpoints (Bootstrap 5)

| Breakpoint | Width | Behavior |
|---|---|---|
| Mobile | < 576px | Single column, centered CTAs |
| `sm` | 576px+ | Toast max-width starts growing |
| `md` | 768px+ | Footer switches to 2-col |
| `lg` | 992px+ | Nav expands, grids go multi-col |
| `xl` | 1200px+ | Toast max-width maxes |

Mobile overrides for all major grids at `max-width: 991.98px`: collapse to 1 column.
Process: 4→2 at `991.98px`, 2→1 at `575.98px`.

---

## 5. Shape

```scss
$border-radius:    0.125rem;  /* 2px — base */
$border-radius-sm: 0.125rem;  /* 2px — small */
$border-radius-lg: 0.25rem;   /* 4px — cards */
$border-radius-xl: 0.375rem;  /* 6px — modals */
```

Set in `static/scss/custom.scss` (or `uploads/custom.scss` in the design-system repo) before Bootstrap import — propagates to all
Bootstrap components automatically. Sharp, editorial; not rounded.

---

## 6. Motion

```css
--ease: cubic-bezier(0.16, 1, 0.3, 1);  /* spring-like, fast settle */
```

| Element | Duration | Easing | Notes |
|---|---|---|---|
| Standard transitions (links, buttons) | `0.25s` | `var(--ease)` | Color, transform, border |
| Navbar blur/bg | `0.3s` | `ease` | On `.scrolled` class |
| Scroll reveal | `0.65s` | `ease` | opacity + translateY |
| Reveal stagger delays | `0.1–0.4s` | — | `.reveal-delay-1` through `-4` |
| Cursor ring lerp | ~8 frames | JS RAF | factor `0.12` per frame |
| Cursor size | `0.15–0.2s` | `var(--ease)` | Expand on hover |
| Marquee | `30s` | `linear` | Pauses on hover |
| Status dot pulse | `2.4s` | `ease-in-out` | opacity 1→0.4→1, infinite |
| Hero elements | `0.8–0.9s` | `var(--ease)` | Staggered `animation: fadeUp` |

### Keyframes

```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(24px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.4; }
}
@keyframes marquee {
  from { transform: translateX(0); }
  to   { transform: translateX(var(--marquee-offset)); }
}
@keyframes blink {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0; }
}
```

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  .reveal { opacity: 1; transform: none; transition: none; }
  .hero-*, .status-dot { animation: none; }
}
```

---

## 7. Logo & Brand Mark

Defined in `assets/branding/`:

| File | Use |
|---|---|
| `kuz-wordmark-color.svg` | Full "KUZ" logotype with accent hairline |
| `kuz-wordmark-mono.svg` | Full "KUZ" logotype with white/black hairline |
| `kuz-avatar-color.svg` | "K" square mark with accent hairline |
| `kuz-avatar-mono.svg` | "K" square mark with neutral hairline |
| `kuz-brand.html` | Full brand identity reference document |

**Wordmark anatomy:**
- Font: DM Sans 500 (geometric sans-serif — clean, versatile)
- Tracking: `0.28–0.32em`
- Hairline rule below letters: `1px`, accent color (`#275dad`) in color variant
- Dark bg (`#141414`), off-white text (`#e8e6e1`) canonical form

**Usage rules:**
- Min width: 120px for wordmark, 16px for K avatar
- Accent on hairline only — never on the letterforms
- No shadows, glows, or outlines on any letterform
- Equal clear space = cap-height of "K" on all sides

**Site favicon:** `static/img/favicon.svg` — K avatar mark

**Profile image:** `static/img/profile-400.webp` (400px), `profile-1000.webp` (1000px).
Square crop, `object-fit: cover`, `aspect-ratio: 1/1`.

---

## 8. Component Library

### 8.1 Skip Link

Accessibility. Fixed top, hidden off-screen, slides in on `:focus`.

```css
.skip-link {
  position: fixed; top: 0; left: 0; right: 0; z-index: 10000;
  transform: translateY(-100%);
  transition: transform 0.25s var(--ease);
  background: var(--accent); color: var(--color-bg);
  font-family: var(--font-mono); font-size: 0.75rem;
  letter-spacing: 0.12em; text-transform: uppercase;
}
.skip-link:focus { transform: translateY(0); }
```

---

### 8.2 Custom Cursor

Desktop only (`hover: hover` + `pointer: fine`). Hidden on touch.

- `#cursor-dot`: 6px circle, accent color, snaps to mouse instantly via `mousemove`
- `#cursor-ring`: 40px circle, 1.5px accent border, lags via RAF lerp (factor `0.12`)
- Hover state: dot expands to 10px, ring to 60px + border-color → `--color-text`
- Keyboard nav mode (`html[data-kb-nav]`): both hidden, `cursor: auto !important`

All `cursor: none` applied globally on desktop — including `*::before`, `*::after`.

---

### 8.3 Sticky Navbar (`#main-nav`, `_nav.html`)

- Height: `--navbar-height: 45px`
- Bootstrap `sticky-top`
- Desktop: 3-column grid — brand (left) | nav links (absolute center) | controls (right)
- Mobile: brand + controls (always visible) | hamburger | collapsible links

**Scroll state** (`.scrolled` class added at 60px via JS):

```css
#main-nav.scrolled {
  background-color: rgba(var(--bs-body-bg-rgb), 0.5) !important;
  backdrop-filter: blur(15px);
  border-bottom-color: var(--color-border);
  box-shadow: var(--bs-box-shadow);
}
```

**Nav link hover:** underline bar grows from center (`left: 50%; width: 0 → 100%`).
Active link color: `var(--accent)`.

**Brand:** monospace, 1.4rem, tracking `0.05em → 0.12em` on hover.

**Controls:** language switcher dropdown + theme toggle (37px circle). Both have
`translateY(-1px)` + shadow on hover, `translateY(0)` on active.

---

### 8.4 Section Label

Reusable pattern: mono uppercase text with leading hairline.

```css
.section-label {
  font-family: var(--font-mono);
  font-size: 0.65rem; letter-spacing: 0.2em; text-transform: uppercase;
  color: var(--accent); margin-bottom: 1rem;
  display: flex; align-items: center; gap: 0.75rem;
}
.section-label::before {
  content: ''; width: 24px; height: 1px;
  background: var(--accent); flex-shrink: 0;
}
```

Same pattern in hero as `.hero-eyebrow` (32px line, 0.18em tracking).

---

### 8.5 Hero Section (`#hero`)

2-column grid layout: left = text stack, right = stat card.

**Left column:**
1. `.hero-eyebrow` — mono label, accent, 0.18em tracking, leading line
2. `.hero-name` — `clamp(3.2rem, 7vw, 6rem)`, weight 700, tracking `-0.02em`
3. `.hero-desc` — 0.95rem, muted, line-height 1.85; `strong` → text color, weight 400
4. `.hero-cta` — flex row, gap 1rem, wraps on mobile

**Right column — `.hero-card`:**
- Surface background, 1px border, 4px radius, 2rem padding
- `.status-row` — mono uppercase, green pulse dot (7px, box-shadow glow)
- `.stat-list` — each item: mono label (small) + large value; accent sub-word in mono 0.9rem
- `.divider-h` — 1px border-color line
- `.tag-cloud` — flex wrap, 6px gap; tags: mono 0.62rem, 1px border, 2px radius

**Mobile:** single column, card moves below text.

---

### 8.6 Marquee Ticker

Between hero and about sections. Horizontal scrolling strip.

- Border top + bottom (`--color-border`)
- Edge fade via `mask-image: linear-gradient(to right, transparent 0%, black 8%, black 92%, transparent 100%)`
- 30s linear loop, `animation-play-state: paused` on hover
- Items: mono 0.65rem, 0.2em tracking, uppercase, muted; separator dots in accent

---

### 8.7 About Section (`#about`)

2-column grid (1fr 1fr, 80px gap). Alternates section background (`--color-bg2`).

**Left:** prose paragraphs, DM Sans 300, 0.95rem, line-height 1.9.

**Right:** `<dl>` facts list:
- `dt`: mono 0.65rem, accent color, 0.12em tracking, uppercase
- `dd`: 0.95rem, muted, border-bottom `--color-border`, 1.25rem padding

---

### 8.8 Work/Skills Section (`#work`)

3-column border-grid. Grid container bg = `--color-border`, cells bg = `--color-bg`.

**`.skill-col`:**
- `.skill-icon` — mono 0.72rem, accent (used as a text label, not SVG icon)
- `.skill-col-title` — 1.35rem, weight 300
- `p` — 0.85rem, muted, line-height 1.7
- `.skill-tags` — flex wrap, margin-top auto (pushes to bottom)
- `.skill-tag` — mono 0.6rem, border, 2px radius; turns accent color on parent hover

Hover: background → `--color-bg2`. If `a.skill-col`: `.skill-view-link` fades in
(opacity 0→1, absolute top-right).

---

### 8.9 Projects Section (`#projects`)

2-column border-grid. Section bg = `--color-bg2`.

**`.project-card`:**
- 48px 44px padding
- Hover: bg → `--color-surface`, bottom accent bar `width: 0 → 100%` (0.4s)
- `.project-link` — absolute top-right, fades in on hover
- `.project-num` — mono 0.65rem, muted, with trailing `flex: 1; height: 1px; background: border` line
- `.project-title` — 1.5rem, weight 300
- `.project-desc` — 0.85rem, muted; `strong` → text color weight 400
- `.project-meta` — flex wrap, margin-top auto; `.project-tag` mono 0.58rem

**`.project-featured`:**
- `grid-column: span 2` (full width)
- Internal 2-col grid: content left, visual right
- `.project-visual` — 16:9 aspect ratio, surface bg, border; contains `.terminal-preview`
- `.terminal-preview` — mono 0.65rem, line-height 1.9; `.t-cursor` blinks; `.t-line-accent` in accent color
- `.featured-badge` — inline, mono 0.6rem, accent bg, white text, 2px radius

Mobile: featured collapses to 1-col, visual hidden.

---

### 8.10 Process Section (`#process`)

4-column bordered grid. Each `.process-step` has `border-right: 1px solid --color-border`.

**Decorative step number:** `::before` pseudo, `content: attr(data-step)`, 4rem,
weight 300, color = `--color-border` (blends into background, felt not read).

**Content:** `.process-step-label` (mono accent), `.process-step-title` (1.1rem
weight 400), `p` (0.8rem muted).

---

### 8.11 Contact Section (`#contact`)

Section bg = `--color-bg2`. `min-height: calc(100vh - var(--footer-height))`.

**`.contact-links`:** flex column. Each `.contact-link`:
- `gap: 1rem → 1.4rem` on hover
- `.contact-link-icon` — mono 0.75rem, accent
- `.contact-link-label` — flex 1
- `.contact-link-meta` — mono 0.65rem, opacity 0→1 on hover
- `.contact-link-arrow` — `translateX(0 → 4px)` on hover

**`.contact-divider`** — 1px border line.

**`.social-links`:** flex row, 2rem gap. Each `.social-link`: mono 0.7rem, muted,
Bootstrap Icon at 1.2rem. Color → `--color-text` on hover.

---

### 8.12 Scrollspy Sidebar (`#scrollspy-nav`)

Fixed right, `top: 50%`, `transform: translateY(-50%)`, z-index 1000.

Each `.nav-link` contains:
- `.dot` — 8px circle, `--color-muted`, opacity 0.3
- `.dot-label` — absolute right-of-dot, mono 0.68rem, slides in on hover

States:
- **Default:** 8px dot, opacity 0.3
- **Hover:** opacity 0.6, `scale(1.3)`, label slides in
- **Focus:** opacity 1, 2px accent outline on dot, label visible
- **Active:** 12px dot, `background: var(--accent)`, opacity 1

No focus outline on the `nav-link` itself — ring is on the `.dot` only.

---

### 8.13 Buttons

All buttons in `main`:

```css
main .btn {
  font-family: var(--font-mono);
  font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase;
  padding: 14px 28px; border-radius: 2px;
  transition: all 0.25s var(--ease);
}
```

**`.btn-ghost`:** transparent bg, muted color, `--color-border` border. Hover:
border → muted, color → text.

**Navbar/control buttons:** `translateY(-1px)` + `var(--shadow-hover)` on hover,
`translateY(0)` + no shadow on active.

---

### 8.14 Toast Notifications

Auto-show on load via `bootstrap.Toast(el).show()`. Responsive max-width (420→560px).

Three semantic variants:

```css
.toast-soft-success  { /* BS success-subtle bg/border/text colors */ }
.toast-soft-warning  { /* BS warning-subtle bg/border/text colors */ }
.toast-soft-danger   { /* BS danger-subtle bg/border/text colors */ }
```

---

### 8.15 Error / Coming Soon Pages

`.custom-page-section`: `min-height: calc(100vh - var(--navbar-height) - 120px)`.

`.error-page-code`: `clamp(5rem, 18vw, 10rem)`, line-height 1, tracking `-0.04em`.

`.error-hint-div`: surface bg, border, 2px radius. `.error-hint-p`: 0.75rem, muted.

---

### 8.16 Vim Shortcuts Modal (`_shortcutsModal.html`)

Triggered by `?` key. `.vim-modal__section-label`: 0.65rem, weight 700, 0.1em
tracking, uppercase, muted. `kbd` badges use accent background.

---

### 8.17 Footer (`_footer.html`)

`border-top`, `py-3`. 2-col on `md+`. Left: mono copyright + accent brand name.
Right: social icons (Bootstrap Icons, `fs-5`), easter egg text, back-to-top button.

`#back-top`: 24px circle, `translateY(-1px)` + shadow on hover.

---

## 9. Keyboard Navigation Mode

`html[data-kb-nav]` attribute suppresses all hover effects site-wide:

- Cursor elements hidden, `cursor: auto` restored
- Nav link underlines don't grow
- Skill/project cards don't darken
- Contact links don't shift
- Scrollspy dot labels don't appear

Allows full keyboard navigation without hover animations firing spuriously.

Vim shortcuts:
- `h / a / w / p / c` — navigate to Home / About / Work / Projects / Contact
- `zz` — scroll to center, `zs` — scroll to top
- `gb / gn / gf` — open GitHub / go to next section / go to first section
- `?` — open shortcuts modal

---

## 10. Scroll Reveal

`.reveal` class: opacity 0, translateY(28px). Added to elements before scroll.
`IntersectionObserver` adds `.visible` when element enters viewport.

```css
.reveal { opacity: 0; transform: translateY(28px); transition: opacity 0.65s ease, transform 0.65s ease; }
.reveal.visible { opacity: 1; transform: translateY(0); }
.reveal-delay-1 { transition-delay: 0.1s; }
.reveal-delay-2 { transition-delay: 0.2s; }
.reveal-delay-3 { transition-delay: 0.3s; }
.reveal-delay-4 { transition-delay: 0.4s; }
```

---

## 11. Theme System

Initialization in `base.html` (inline `<script>` before CSS) prevents theme flash:

```js
var stored = localStorage.getItem("theme");
// resolves "auto" → system preference
document.documentElement.setAttribute("data-bs-theme", resolvedTheme);
```

Toggle via `#theme-toggle` button updates `data-bs-theme` on `<html>` and persists
to `localStorage`. Icon (`#theme-icon`) swapped by `static/js/theme.js`.

Supported values: `"light"` | `"dark"`.

---

## 12. CSS File Map

| File | Scope |
|---|---|
| `static/scss/custom.scss` (or `uploads/custom.scss` in the design-system repo) | Bootstrap overrides: `$primary`, `$font-family-*`, `$border-radius-*`, custom colors |
| `static/css/main.css` (or `uploads/main.css` in the design-system repo) | Design tokens, base styles, navbar, cursor, scroll reveal, scrollspy, toasts, error pages, keyboard nav mode |
| `static/css/home.css` (or `uploads/home.css` in the design-system repo) | All home-page sections: hero, marquee, about, work/skills, projects, process, contact |

Page-specific CSS loaded via `{% block extra_css %}` in templates.

---

## 13. Asset Map

| Asset | Path | Notes |
|---|---|---|
| Profile photo (400px) | `static/img/profile-400.webp` | Square, object-fit cover |
| Profile photo (1000px) | `static/img/profile-1000.webp` | Retina / hero use |
| Profile wide | `static/img/profile-wide.png` | Landscape crop |
| Favicon SVG | `static/img/favicon.svg` | K avatar mark |
| Favicon ICO | `static/img/favicon.ico` | Fallback |
| OG card | `static/img/og-card.png` | 1200×630px |
| Wordmark (color) | `assets/branding/kuz-wordmark-color.svg` | Accent hairline |
| Wordmark (mono) | `assets/branding/kuz-wordmark-mono.svg` | Neutral hairline |
| Avatar (color) | `assets/branding/kuz-avatar-color.svg` | Accent hairline |
| Avatar (mono) | `assets/branding/kuz-avatar-mono.svg` | Neutral hairline |
| Brand reference | `assets/branding/kuz-brand.html` | Full identity doc |
| Screenshots | `assets/screenshot-dark.png`, `screenshot-light.png` | Site reference |

---

## 14. Don't Rules

| Don't | Why |
|---|---|
| Use `#000000` or `#ffffff` | Warm off-tones are the brand |
| Use `border-radius` > 4px on cards | Sharp corners = editorial identity |
| Use any color other than `--accent` for interactive highlights | Single-accent discipline |
| Use DM Sans for labels, tags, nav, or metadata | Mono is the technical voice |
| Use JetBrains Mono for body prose or headings | Sans is the human voice |
| Hardcode hex values outside `:root` | Break theme switching |
| Skip `--accent-hover` on accent surfaces | `color-mix(in srgb, var(--accent) 85%, black)` keeps hover in sync with `--accent` |
| Skip `prefers-reduced-motion` | Accessibility requirement |
| Apply hover effects in keyboard nav mode | `html[data-kb-nav]` must suppress all |
