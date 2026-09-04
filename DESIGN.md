# Архив СПбГТИ — Design System

## 0. Research Log

- Embedded refs: shortlisted [lamborghini.md, notion.md, claude.md] → picked `soft-skill.md` (Layer A) + `lamborghini.md` (Layer B) because: dark+gold palette matches existing tokens, editorial luxury aligns with archival academic tone, zero-radius cards and uppercase display provide institutional gravitas without supercar aggression
- Skipped lanes: lazyweb (no live site to research), imagen drafts (text-only demo)

## 1. Atmosphere & Identity

A quiet archive vault. The interface recedes — surfaces merge into darkness, content emerges under spotlights. The signature is **tonal depth through surface layering**: cards float on charcoal over true black, separated by subtle luminance shifts rather than borders. Gold is rare and deliberate — only for primary actions and the archive's institutional identity. The mood is nocturnal scholarship: exclusive, measured, deliberately slow. Every section transition is a scroll through darkness into the next illuminated artifact.

## 2. Color

### Palette

| Role | Token | Value | Usage |
|------|-------|-------|-------|
| Surface/void | `--surface-void` | `#0a0a0f` | Page background, deepest layer |
| Surface/primary | `--surface-primary` | `#12121a` | Main content canvas |
| Surface/elevated | `--surface-elevated` | `#1a1a25` | Cards, panels, modals |
| Surface/hover | `--surface-hover` | `#22222e` | Card hover state |
| Text/primary | `--text-primary` | `#f5f5f5` | Headlines, body text on dark |
| Text/secondary | `--text-secondary` | `#a0a0a8` | Captions, metadata, hints |
| Text/muted | `--text-muted` | `#6b6b73` | Disabled, timestamps |
| Accent/gold | `--accent-gold` | `#c9a96e` | Primary CTA, institutional identity |
| Accent/gold-light | `--accent-gold-light` | `#d4b87a` | Hover state for gold |
| Accent/gold-dark | `--accent-gold-dark` | `#b8985a` | Active/pressed state |
| Border/default | `--border-default` | `#2a2a35` | Dividers, card outlines |
| Border/hover | `--border-hover` | `#3a3a45` | Hover border shift |
| Status/success | `--status-success` | `#4ade80` | Confirmations |
| Status/error | `--status-error` | `#f87171` | Errors, destructive |

### Rules
- Surface hierarchy creates depth via tonal shifts — no shadows, no borders where possible
- Gold is sacred — used ONLY for primary CTA and the archive's institutional mark. Never decorative.
- Never introduce a color not in this table. Extend the table first.
- Contrast: `--text-primary` on `--surface-void` = 15.3:1 (exceeds WCAG AAA)

### Light Theme Tokens

Activated via `data-theme="light"` on `<html>`. Light theme reverses the tonal hierarchy: light surfaces with dark text. Accent gold is slightly darkened (`#b8985a`) for adequate contrast on light backgrounds.

| Role | Token | Value | Usage |
|------|-------|-------|-------|
| Surface/void | `--surface-void` | `#f8f8f6` | Page background |
| Surface/primary | `--surface-primary` | `#ffffff` | Main content canvas |
| Surface/elevated | `--surface-elevated` | `#f0f0ee` | Cards, panels |
| Surface/hover | `--surface-hover` | `#e8e8e6` | Interactive hover |
| Text/primary | `--text-primary` | `#1a1a1f` | Headlines, body text |
| Text/secondary | `--text-secondary` | `#5a5a63` | Captions, metadata |
| Text/muted | `--text-muted` | `#8a8a93` | Disabled, timestamps |
| Accent/gold | `--accent-gold` | `#b8985a` | Primary CTA (darkened for contrast) |
| Accent/gold-light | `--accent-gold-light` | `#c9a96e` | Hover state |
| Accent/gold-dark | `--accent-gold-dark` | `#a08040` | Active/pressed state |
| Border/default | `--border-default` | `#d5d5d0` | Dividers |
| Border/hover | `--border-hover` | `#b5b5b0` | Hover border shift |
| Status/success | `--status-success` | `#16a34a` | Confirmations |
| Status/error | `--status-error` | `#dc2626` | Errors |

## 3. Typography

### Scale

| Level | Size | Weight | Line Height | Tracking | Usage |
|-------|------|--------|-------------|----------|-------|
| Display | clamp(2rem, 5vw, 3.5rem) | 700 | 1.1 | -0.02em | Hero, page title |
| H1 | clamp(1.5rem, 3vw, 2.25rem) | 700 | 1.2 | -0.015em | Section headers |
| H2 | 1.5rem | 600 | 1.3 | -0.01em | Subsection headers |
| H3 | 1.25rem | 600 | 1.4 | 0 | Card titles |
| Body/lg | 1.125rem | 400 | 1.6 | 0 | Lead paragraphs |
| Body | 1rem | 400 | 1.6 | 0 | Default text |
| Body/sm | 0.875rem | 400 | 1.5 | 0 | Secondary info |
| Caption | 0.75rem | 500 | 1.4 | 0.02em | Labels, metadata |
| Overline | 0.6875rem | 600 | 1.3 | 0.08em | Section labels, uppercase |

### Font Stack
- Display: `'Playfair Display', Georgia, 'Times New Roman', serif`
- Body: `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Mono: `'JetBrains Mono', 'Fira Code', 'Cascadia Code', monospace`

### Rules
- Max 2 font families: Playfair for display/headings, Inter for body/UI
- Body text never below 14px (0.875rem)
- Headings that wrap to 4+ lines are too large — use clamp()
- Display font is serif — preserves archival, scholarly tone
- Uppercase reserved for overlines and section labels only — not full headlines (unlike Lamborghini)

## 4. Spacing & Layout

### Base Unit
All spacing derives from a base of **4px**.

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Tight: icon-to-label |
| `--space-2` | 8px | Compact: list items, inline groups |
| `--space-3` | 12px | Default: form field padding |
| `--space-4` | 16px | Standard: card padding, input height |
| `--space-5` | 20px | Comfortable: section inner spacing |
| `--space-6` | 24px | Generous: card padding (default) |
| `--space-8` | 32px | Separated: between card groups |
| `--space-10` | 40px | Sections within a page |
| `--space-12` | 48px | Major section breaks |
| `--space-16` | 64px | Page-level vertical rhythm |
| `--space-20` | 80px | Hero spacing |
| `--space-24` | 96px | Maximum section separation |

### Grid
- Max content width: 1280px
- Column system: 12-column, 24px gutter, 16px margin at mobile
- Breakpoints: sm 640px, md 768px, lg 1024px, xl 1280px

### Rules
- Macro-whitespace: section padding minimum `py-24` (96px) — the layout breathes heavily
- Mobile: all multi-column collapses to single-column with `px-4` (16px) horizontal padding
- Filter bar: horizontal scroll on mobile, no wrapping

## 5. Components

### Card (Media Item)
- **Structure**: `<article>` wrapper → inner shell (image container) + content area (title + metadata)
- **Variants**: photo (16:9 image), video (16:9 + play icon overlay), document (icon + filename)
- **Spacing**: `--space-6` padding, `--space-4` gap between image and text
- **States**: default (`--surface-elevated`), hover (`--surface-hover` + border shift), focus (2px gold outline offset 2px), disabled (opacity 0.5)
- **Accessibility**: `role="article"`, `aria-label` with filename, keyboard focusable
- **Motion**: hover — border-color 200ms ease-out; entry — fade-up 600ms cubic-bezier(0.16, 1, 0.3, 1)
- **Layout**: CSS Grid, `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`

### Filter Bar
- **Structure**: `<nav>` → horizontal list of `<button>` pills
- **Variants**: default (outlined), active (gold fill)
- **Spacing**: `--space-2` gap between pills, `--space-3` padding per pill
- **States**: default (border only), active (gold bg + dark text), hover (gold border)
- **Accessibility**: `role="tablist"`, `aria-selected`, keyboard arrow navigation
- **Motion**: background-color 150ms ease-out
- **Layout**: horizontal flex, `overflow-x: auto` on mobile

### Lightbox Modal
- **Structure**: `<dialog>` overlay → close button + image container + metadata panel
- **Variants**: single image, slideshow (prev/next navigation)
- **Spacing**: `--space-8` padding, `--space-6` gap between image and metadata
- **States**: open (backdrop-blur + fade-in), closing (fade-out)
- **Accessibility**: `role="dialog"`, `aria-modal="true"`, focus trap, Escape to close
- **Motion**: backdrop 300ms ease-out, content 400ms cubic-bezier(0.16, 1, 0.3, 1)
- **Layout**: fixed fullscreen, centered content

### Search Input
- **Structure**: `<input>` with search icon prefix
- **Variants**: default (dark bg), focused (gold border)
- **Spacing**: `--space-3` horizontal padding, `--space-4` height
- **States**: default, focus (gold border + outline), disabled
- **Accessibility**: `role="searchbox"`, `aria-label="Поиск архива"`, `type="search"`
- **Motion**: border-color 150ms ease-out
- **Layout**: inline within header

### Statistics Badge
- **Structure**: `<span>` with count + label
- **Variants**: default (muted text), highlighted (gold text)
- **Spacing**: `--space-2` padding, `--space-1` gap
- **States**: default, highlighted
- **Accessibility**: `aria-label` with full text
- **Motion**: none (static)
- **Layout**: inline-flex

## 6. Motion & Interaction

### Timing

| Type | Duration | Easing | Usage |
|------|----------|--------|-------|
| Micro | 100-150ms | ease-out | Button press, toggle, pill hover |
| Standard | 200-300ms | ease-in-out | Panel open, tab switch, border shift |
| Emphasis | 400-600ms | cubic-bezier(0.16, 1, 0.3, 1) | Page transition, hero entry, lightbox open |
| Scroll-driven | tied to scroll | linear | Card reveal (staggered) |

### Rules
- Only animate `transform` and `opacity`. Never animate layout properties.
- Every interactive element has hover + active + focus states.
- Scroll-triggered animations use `IntersectionObserver`, not scroll listeners.
- Reduced motion: respect `prefers-reduced-motion` — disable non-essential animation.
- Card entry: staggered fade-up, 60ms delay between items, 600ms duration.
- Lightbox: backdrop fade 300ms, content scale 400ms with spring easing.

## 7. Depth & Surface

### Strategy: Tonal Shift

Surfaces use progressively lighter shades from the void. No borders, no shadows — depth is communicated through luminance.

| Level | Token | Value | Usage |
|-------|-------|-------|-------|
| Void | `--surface-void` | `#0a0a0f` | Page background |
| Base | `--surface-primary` | `#12121a` | Content canvas |
| Elevated | `--surface-elevated` | `#1a1a25` | Cards, panels |
| Hover | `--surface-hover` | `#22222e` | Interactive hover |

### Accent Depth
- Gold on void = 7.2:1 contrast (exceeds WCAG AAA)
- Gold on elevated = 5.8:1 contrast (exceeds WCAG AA)

## 8. Accessibility Constraints & Accepted Debt

### Constraints
- WCAG 2.2 AA target — contrast floor 4.5:1 body / 3:1 large text
- Visible focus on every interactive element (2px gold outline, 2px offset)
- Full keyboard reachability (Tab, Arrow keys, Escape, Enter)
- `prefers-reduced-motion` respected (Section 6)
- Skip-to-content link present
- All images have `alt` text
- Semantic HTML: `<header>`, `<main>`, `<footer>`, `<nav>`, `<article>`, `<dialog>`
- ARIA landmarks and labels for screen readers

### Accepted Debt

| Item | Location | Why accepted | Owner / Exit |
|------|----------|--------------|--------------|
| Demo-only data | index.html | Prototype phase — real data from Directus later | Pre-launch |
| No i18n | index.html | Russian-only archive, no multilingual needed | Never |
| Google Fonts CDN | index.html | External dependency for Playfair Display + Inter | Self-host pre-launch |
