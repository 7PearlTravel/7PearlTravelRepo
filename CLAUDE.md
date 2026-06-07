# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

7 Pearl Travel Service is a static single-page website for a travel agency specializing in domestic and international tour packages. The entire application lives in one file: `index.html` (1657 lines), with all HTML, CSS, and JavaScript embedded inline. There is no build system, package manager, or server-side code.

## Running & Development

No build step required — open `index.html` directly in a browser, or serve it with any static HTTP server:

```powershell
# Python
python -m http.server 8000

# Node.js
npx http-server
```

No automated tests exist. Manual testing covers: carousel navigation, modal dialogs (booking form, tour details), tab switching, mobile menu toggle, scroll-to-top button, and WhatsApp subscription.

## Architecture

The single `index.html` is structured in three contiguous blocks:

### CSS (lines 11–363)
- CSS custom properties in `:root`: `--orange (#E8500A)`, `--navy (#0D1B3E)`, etc.
- Responsive design via `clamp()` for fluid sizing; media queries at ~768px and ~960px

### HTML body (lines 365–866)
- Floating WhatsApp button, scroll-to-top button, toast notification (`id="toast"`)
- **Mobile menu**: `id="mMenu"` — slide-in drawer toggled by `mOpen()`/`mClose()`
- **Modals**: `id="modal"` (booking form), `id="tourModal"` (dynamic tour details, `id="tourContent"`)
- **Nav**: Sticky header with logo, desktop nav links with hover dropdowns, hamburger
- **Hero**: `<section id="home">` — full-screen carousel with slides and dot indicators
- **Sections**: Search bar, trust strip, tour grid with tab panels (`id="panel-dom"` / `id="panel-int"`), testimonials carousel, contact form, footer (`<footer>` at line 830)

### JavaScript (lines 867–1655)

| Section | Lines | Description |
|---|---|---|
| `tourData` object | 869–1494 | 16 tour packages (ladakh, goa, kerala, rajasthan, andaman, varanasi, manali, himachal, maldives, dubai, bali, paris, switzerland, kenya, singapore, thailand) |
| Hero carousel | 1497–1511 | `goSlide()`, `nextSlide()`, `prevSlide()`, `resetSlideTimer()` — auto-advances every 5 s |
| Tab switching | 1513–1520 | `switchTab(tab)` — manages `.active` class on tab buttons and panels |
| Testimonials | 1522–1544 | `goTest(n)` — responsive (1/2/3 cards), auto-advances every 4.5 s |
| Modals & UI | 1546–1592 | `openModal`/`closeModal`, `mOpen`/`mClose`, `toast(msg)`, `submitBook()`, `submitContact()`, `waSubscribe()` |
| Scroll effects | 1594–1605 | Scroll-to-top button (threshold 400 px), IntersectionObserver for `.reveal` fade-in |
| Tour modal | 1607–1642 | `openTourModal(tourId)` / `closeTourModal()` — dynamically builds modal HTML from `tourData` |
| Image fallback | 1644–1654 | Sets `opacity:0` on broken `.dest-card img` elements |

## Key Patterns

**Adding a new tour**: Add an entry to `tourData` (line 869). `openTourModal()` generates all modal HTML automatically — no HTML changes needed. Each tour entry requires: `name`, `category`, `subtitle`, `image`, `price`, `duration`, `highlights[]` (each with `icon` and `text`), `itinerary[]` (each with `day`, `title`, `desc`), `inclusions[]`, `exclusions[]`.

**Modal system**: Two modals coexist. The booking modal (`id="modal"`) is static HTML reused for all inquiries. The tour-details modal (`id="tourModal"`) is fully dynamic, rebuilt on each `openTourModal()` call. Both lock body scroll when open.

**Forms send via WhatsApp**: `submitBook()` and `submitContact()` both construct a WhatsApp deep-link (`wa.me/918866030780`) with a pre-filled message and open it in a new tab — there is no backend or email submission.

**Contact info**: Hardcoded in multiple places — search for `8866030780` / `918866030780` to find all occurrences when updating.

**Google Review link**: The "Write a Google Review" button (line ~771, inside the `.gcta` testimonials CTA) points to `https://g.page/r/CTIJp3ZszjqoEAE/review`. Update this URL if the Google Business listing changes.
