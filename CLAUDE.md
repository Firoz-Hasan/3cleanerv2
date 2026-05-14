# CLAUDE.md — 3 Cleaners Website

This file is read by Claude Code at the start of every session. It captures the decisions made during initial development so future work stays consistent.

---

## What this project is

A static, single-page marketing website for **3 Cleaners**, a professional cleaning service. Source of truth for design and content: `3cleaners-website.md` (the original brief the client provided).

**Stack:** Pure HTML + CSS + vanilla JS. Bootstrap 5.3 via CDN. No npm, no bundler, no framework. Everything lives in one file: `index.html`.

---

## File map

```
index.html   — the entire site
assets/      — drop static assets here (logo, images)
README.md    — user-facing setup & customisation guide
CLAUDE.md    — this file (AI context)
```

---

## Architecture decisions

### Single-file approach
All CSS is in a `<style>` block and all JS is in a `<script>` block inside `index.html`. This was a deliberate choice from the brief ("no npm, just open and view"). **Do not split into separate `.css` / `.js` files unless the user explicitly asks**, because it would require a local server or module bundler to avoid CORS issues when loading from the filesystem.

### CSS custom properties for theming
All brand colors are defined in `:root` at the top of `<style>`. Never hardcode hex values inline — always reference a `var(--...)` token. This makes a full rebrand a one-line-per-color change.

### No JavaScript framework
The only JS on the page handles:
1. Smooth scrolling for anchor links
2. Auto-closing the mobile hamburger menu after a link is tapped
3. IntersectionObserver-based active nav highlight

Keep additions lightweight. A jQuery dependency or a bundled component library would violate the "just open in browser" constraint.

### Form is intentionally unconnected
The contact `<form>` in `#contact` has no `action` attribute. Backend wiring is deferred to the client. When connecting it, prefer Formspree or Getform (no backend code needed). If a custom backend is used, add CSRF protection.

---

## Brand & design rules

Colors extracted programmatically from `assets/3Cleaners_logo.png` (Pillow quantize, top 10 dominant hues).

| Token | Hex | Source in logo | Rule |
|-------|-----|---------------|------|
| `--primary-blue` | `#016BBD` | Circle arc & "3" gradient | Primary CTA buttons, card borders, hero icon |
| `--dark-navy` | `#02204A` | "CLEANERS" wordmark | All headings, navbar brand, footer background |
| `--accent-green` | `#7CB750` | Leaves & bucket | Eco accents, check-circle icons, "Most Popular" badge |
| `--light-bg` | `#F0F7FF` | Derived: blue-tinted white | Services section bg (alternates with `#fff`) |
| `--text-dark` | `#02204A` | Same as dark-navy | Body copy (unified with brand) |
| `--text-muted` | `#5A6E8A` | Derived | Subtitles, descriptions, pricing feature lists |

Hardcoded hover colors that must be updated if green changes:
- `.btn-green:hover` → `#619040` (darker of `#7CB750`)
- `.service-card.green .icon` gradient endpoint → `#4A7030`

**Card alternation pattern:** Service cards alternate blue (`service-card`) / green (`service-card green`) top-border by position in the grid (1st=blue, 2nd=green, 3rd=blue, ...). Maintain this if adding new service cards.

**Section alternation pattern:** `#about` = white bg → `#services` = `--light-bg` → `#pricing` = white → `#contact` = dark gradient. Keep this rhythm when inserting new sections.

**Buttons:** All buttons use `border-radius: 50px` (pill shape). Never use square buttons.

---

## Section inventory

| Section | Anchor | Notes |
|---------|--------|-------|
| Navbar | — | Sticky-top; collapses to hamburger on mobile |
| Hero | `#home` | Full-width gradient; Bootstrap icon as decorative visual (replace with image later) |
| About | `#about` | Left: copy block. Right: 3 feature rows with colored icon boxes |
| Services | `#services` | 6 cards, 3-col desktop, 2-col tablet, 1-col mobile |
| Pricing | `#pricing` | 3 tiers; Standard (`featured`) is visually elevated with border + scale |
| Contact | `#contact` | Left: 4 info cards. Right: 5-field form |
| Footer | — | Social icons + copyright line |

---

## How to add new features

### New service card
1. Copy any `<div class="col-md-6 col-lg-4">` block in `#services`.
2. Assign `service-card` (blue) or `service-card green` to maintain the alternating pattern.
3. Pick an icon from Bootstrap Icons (`bi bi-*`).
4. Also add the new service as an `<option>` in the contact form's `<select>`.

### New pricing tier
1. Copy a `<div class="col-md-6 col-lg-4">` block in `#pricing`.
2. At most one card should have `class="featured"` — it should be the mid-tier.
3. Update the `col-lg-*` widths if the tier count changes (3 tiers = `col-lg-4`; 4 tiers = `col-lg-3`).

### New section
1. Add a `<section id="new-id" class="new-class">` between existing sections.
2. Respect the white / light-bg alternation pattern.
3. Add a matching `<li class="nav-item"><a class="nav-link" href="#new-id">Label</a></li>` in the navbar.
4. The IntersectionObserver in the JS block auto-picks up all `section[id]` elements — no extra JS needed.

### Logo
The real logo is already in place at `assets/3Cleaners_logo.png` (copied from `~/Downloads`). The navbar uses `<img src="assets/3Cleaners_logo.png" height="52">`. Do not revert to text-only brand.

### Adding a hero image
Replace `<i class="bi bi-house-heart-fill hero-visual"></i>` with an `<img>` tag. Make it `w-100 rounded-4` for a responsive, rounded look.

### Connecting the contact form
The `<form>` inside `.contact-form` needs an `action` attribute. Preferred:
- Formspree: `action="https://formspree.io/f/YOUR_ID" method="POST"`
- Getform: `action="https://getform.io/f/YOUR_ID" method="POST"`
- Custom backend: ensure CSRF token is included.

---

## Responsive rules

| Breakpoint | Key behaviours |
|-----------|---------------|
| < 768 px | 1-col layout; hero text left-aligned on lg, centred on mobile; `featured` pricing card loses `scale(1.04)` |
| 768–991 px | 2-col services & pricing |
| ≥ 992 px | 3-col services & pricing; sticky navbar at full width; `featured` card scaled up |

Never remove the `@media (max-width: 768px)` block — it resets the featured card scale which would otherwise cause horizontal scroll on narrow screens.

---

## What NOT to do

- Do not add a build step (webpack, Vite, Parcel) unless the user explicitly opts in — it breaks the "just open in browser" experience.
- Do not split the CSS into a separate file without also setting up a local server (file:// origin blocks ES module imports and some fetch calls).
- Do not hardcode hex color values outside of `:root` — always use `var(--token)`.
- Do not add a second `.featured` pricing card — only one should be highlighted at a time.
- Do not import jQuery — vanilla JS covers all current needs.
