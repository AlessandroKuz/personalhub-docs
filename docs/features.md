# Features

A complete reference of every feature and UI element on the site.
Organised by location — global elements first, then per-page.

---

## Global — Present on Every Page

### Navigation Bar
- Site name / logo — links to home (`core:home`)
- Navigation links: About, Work, Projects, Blog
- Language switcher (EN / IT — see [i18n](frontend/i18n.md))
- Theme toggle (light / dark)
- Fully responsive — collapses to hamburger menu on mobile (Bootstrap 5 navbar)

### Footer
- Social links: GitHub, LinkedIn, YouTube
- Contact shortcut: email link
- Copyright line with current year
- Secondary navigation: Privacy, Back to top

### Theme Toggle
- Switches between light and dark mode
- Preference persisted in `localStorage` — survives page reloads and browser restarts
- Implemented via `data-theme` attribute on `<html>` + CSS custom properties
- No flash of wrong theme on load — preference applied before first paint

### Language Switcher
- EN / IT toggle (expandable to ES, DE)
- Posts to Django's `set_language` endpoint
- Redirects back to the same page in the new language
- Language preference stored in session

### Responsive Layout
- Mobile-first using Bootstrap 5 grid
- All pages fully functional on mobile, tablet, and desktop
- Touch-friendly tap targets throughout

---

## Landing Page (`/`)

A TLDR of the entire site — each section is a preview that links to its full page.

### Hero Section
- Name, title ("ML Engineer · Data Scientist · Builder")
- One-line personal tagline
- Two CTAs: "View my work" → `/work/`, "Contact me" → `/contact/`

### About Snippet
- 2–3 sentence personal summary
- Link to full About page

### Skills Snapshot
- Compact overview of core competencies
- Link to full Work page

### Featured Projects Preview
- 2–3 project cards (featured=True projects from Phase 2 onward)
- Link to full Projects page

### Blog Preview
- 2–3 latest post cards (Phase 3 onward)
- Link to full Blog page

### Contact CTA
- Prominent section at the bottom: "Let's work together"
- Links to Contact page

---

## About Page (`/about/`)

Personal description page.

### Bio
- Full personal description: background, personality, approach to work
- Professional identity as ML Engineer, Data Scientist, Builder

### Interests
- Personal and professional interests
- Languages spoken (IT, EN, ES, DE, RU, UA) with proficiency indication

### Contact CTA
- Reusable `_cta.html` component at the bottom of the page

---

## Work / Career Page (`/work/`)

Professional overview — functions as an interactive CV.

### Skills & Capabilities
- Categorised skill overview: ML/AI, backend development, tooling, etc.
- Technologies, frameworks, and tools

### Interactive Timeline
- Combined education and career timeline
- Rendered in pure CSS/HTML — no model or database needed in Phase 1
- Each entry: date range, role/degree, institution, brief description
- Responsive layout: vertical on mobile, horizontal or alternating on desktop

### CV Download
- Prominent download button
- PDF served from `static/files/cv.pdf`
- Opens in new tab or triggers download

### Contact CTA
- Reusable `_cta.html` component at the bottom of the page

---

## Projects Page (`/projects/`) — Phase 2

Dynamic project portfolio managed via Django admin.

### Project Grid
- Cards rendered from `Project` model instances
- Each card: title, description snippet, tag chips, GitHub link, live link
- Empty state: friendly message when no projects exist

### Tag Filtering
- Filter buttons — one per `Tag` instance
- HTMX-powered: clicking a tag fetches filtered results without a page reload
- URL updated via `hx-push-url` — filtered state is bookmarkable
- "All" button resets the filter

### Project Detail Page (`/projects/<slug>/`)
- Full project write-up (`body` field — supports rich text or Markdown)
- Tag list
- GitHub and live site links
- Back to projects link

---

## Blog Page (`/blog/`) — Phase 3

Writing and reflections, published via a lightweight editor.

### Post List
- Post cards sorted by `published_at` descending
- Each card: title, date, excerpt, "Read more" link
- Only `status=PUBLISHED` posts shown to the public

### Post Detail Page (`/blog/<slug>/`)
- Full post body rendered from Markdown source
- Published date
- Back to blog link

### Writing Interface (`/blog/editor/`) — staff only
- EasyMDE Markdown editor (loaded via CDN — no build step)
- WebSocket-powered live preview (server renders Markdown → HTML in real time)
- Save as draft / publish controls

---

## Contact Page (`/contact/`)

### Contact Form
- Fields: name, email, message
- HTMX-powered submission — no full page reload
- Server-side validation with inline error messages
- On success: form replaced by confirmation message
- Django sends email to `CONTACT_EMAIL` via configured SMTP backend
- CSRF protected

### Scheduling Link
- Prominent link / embedded widget to cal.com booking page
- Allows visitors to book a call directly

### Social Links
- GitHub profile link
- LinkedIn profile link
- Email address (mailto link)
