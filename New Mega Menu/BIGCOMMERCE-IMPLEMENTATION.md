# Regal Fish Mega Menu — BigCommerce Implementation Guide

## Overview

This guide covers implementing the custom mega menu on the Regal Fish BigCommerce store. The implementation replaces the default Cornerstone flat-dropdown navigation with a three-region mega dropdown (left rail + card panels + promo rail) and a fully self-contained mobile menu.

**Source file:** `mega-menu.html`
**Reference file:** `original-header.html` (original Cornerstone nav — do not edit)

---

## What Changed vs Original (`index.html`)

| Area | Original (`index.html`) | New (`mega-menu.html`) |
|------|------------------------|----------------------|
| **Nav items** | Shop A-Z, Best Sellers, Fish, Smoked, Shellfish, Prepared, Recipes, Sign in, Club Regal | Shop Fish (mega), Best Sellers, Recipes, Sign in, Club Regal |
| **Dropdown style** | Flat single-column text links (Cornerstone default) | Three-region layout: left rail + card grid + promo rail |
| **Mobile menu** | Theme's `navPages-container` toggled open/closed | Fully self-contained `<nav class="mmenu">` accordion |
| **Hamburger icon** | Theme default (3 dark bars) | Custom gold circle (41.19px, white bars) |
| **Search bar** | Fixed 160px width | Fills available space (`1fr` in grid) |
| **Fonts** | Outfit only | Outfit (body) + Gloock (headings, left rail) |
| **Images** | None in nav | 30 product/category images |
| **Breakpoints** | Theme defaults (801px / 1200px / 1440px) | 2 breakpoints: desktop (>1024px), mobile (≤1024px) |

---

## Step 1 — Upload Images

Upload all 30 images from `assets/images/` to BigCommerce:

**Path:** `/assets/images/` (relative to theme root)

### Category Card Images (27 files)

| Panel | Files |
|-------|-------|
| Fish (9) | `white-fish.jpg`, `oily-fish.jpg`, `flatfish.jpg`, `whole-fish.jpg`, `fresh-fish.jpg`, `frozen-fish.jpg`, `fish-portions.jpg`, `fish-fillets.jpg`, `seasonal-picks.jpg` |
| Shellfish (5) | `prawns.jpg`, `crab.jpg`, `lobster.jpg`, `scallops.jpg`, `mussels.jpg` |
| Smoked (3) | `smoked-salmon.jpg`, `smoked-fish.jpg`, `cured-fish.jpg` |
| Prepared (5) | `no-prep-needed.jpg`, `chef-ready.jpg`, `fish-cakes.jpg`, `fish-fingers.jpg`, `battered-breaded.jpg` |
| Pantry (5) | `tinned-fish.jpg`, `fish-sauce-butter.jpg`, `pantry.jpg`, `accessories.jpg`, `pet-treats.jpg` |

### Featured Promo Card Images (3 files)

| Card | File |
|------|------|
| Best Sellers | `feature-best-sellers.jpg` |
| New Arrivals | `feature-new-in.jpg` |
| Club Regal | `feature-club-regal.jpg` |

---

## Step 2 — Add Fonts

In the theme's `templates/layout/base.html` (or equivalent), add to `<head>` before the theme stylesheet:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600&display=swap" rel="stylesheet">
<link href="https://fonts.bunny.net/css?family=gloock:400" rel="stylesheet">
```

---

## Step 3 — Add CSS Overrides

Add the contents of `<style id="megaMenuOverrides">` (lines 46–566 of `mega-menu.html`) to the theme.

**Recommended location:** Create a new file `assets/css/mega-menu.css` and reference it after `theme.css` in the base template, OR inject via Stencil's `{{> components/common/styles}}` block.

### Breakpoint Map

| Breakpoint | Purpose |
|-----------|---------|
| **Base (no MQ)** | Desktop defaults: logo left-aligned, nav centered with margin, `.mmenu` hidden, mega dropdown base styles, left rail, card grid, promo rail |
| `min-width: 980px` | Nav item font-size and padding (consolidates theme's 1200px + 1440px rules) |
| `min-width: 1025px` | Full desktop grid: `logo navigation search usernav` with search filling available space. `padding: 2rem 1.5rem` |
| `min-width: 801px` and `max-width: 1024px` | Two-row mobile grid forced (prevents theme switching to single-row) |
| `max-width: 1025px` (hide at exactly 1025px) | Hamburger suppressed at 1025px to align with 1024px mobile switch |
| `max-width: 1024px` | Full mobile: hamburger styled as gold circle, theme nav hidden, `.mmenu` active, pill-tab layout for dropdown, accordion groups |
| `max-width: 480px` (nested) | Single-column cards on very small screens |

### Key CSS Behaviors

- **`.navPages-item--mega { position: static; }`** — anchors the dropdown to the header, not the nav item
- **`.header__dropdown::before`** — invisible 10px hover bridge between trigger and dropdown
- **`.navPages-container` + `.navPages-container.is-open { display: none !important; }`** — hides theme's mobile nav so our `.mmenu` replaces it
- **Hamburger** — 41.19px gold circle (`#b8a77b`) with white bars, centered via absolute positioning + transform
- **Card hover** — `scaleX(0 → 1)` underline bar animation at `0.35s ease-out`, matching theme's `.navPages-action:hover:after`
- **Image zoom** — `scale(1.08)` on card hover, `scale(1.06)` on promo card hover
- **Promo card sizing** — Container: `width: 30%; min-width: 220px`. Image: `flex: 0 0 35%; height: 100%; object-fit: cover` (scales with dropdown, images fill full card height)

---

## Step 4 — Replace Header HTML

Replace the entire `<header>` section in `templates/layout/base.html` with the header from `mega-menu.html` (lines 568–920).

### Header Grid Structure

```
<header class="header">
  <div class="header--inner">
    <a class="mobileMenu-toggle" data-mobile-menu-toggle="menu">
    <nav class="navUser">Account + Cart</nav>
    <div class="header-logo">Logo</div>
    <div class="navPages-container" id="menu" data-menu>
      <nav class="navPages">
        <ul class="navPages-list navPages-list--mega">
          <li class="navPages-item navPages-item--mega">
            <a class="navPages-action" href="/brands/">Shop Fish</a>
            <div class="header__dropdown" id="navPages-Mega">...</div>
          </li>
          <li><a href="/best-sellers/">Best Sellers</a></li>
          <li><a href="/recipes/">Recipes</a></li>
          <li class="nav-acccount hide-desktop">Sign in / Register</li>
          <li><a href="/club-regal">Club Regal</a></li>
        </ul>
      </nav>
    </div>
    <div class="header-search">Search form</div>
  </div>

  <!-- Mobile menu (self-contained) -->
  <nav class="mmenu" id="mmenu" aria-hidden="true">...</nav>
</header>
```

### Mega Dropdown Structure

```
.header__dropdown#navPages-Mega
  .header__dropdown__container
    .header__dropdown__all-fish
      |
      |-- LEFT RAIL (.header__dropdown__all-fish__types) [240px]
      |     5 buttons: Fish / Shellfish / Smoked & Cured / Prepared & Easy / Pantry & More
      |     Each has: data-target + data-handle attributes, Gloock font, gold underline hover
      |
      |-- MIDDLE PANEL (.header__dropdown__all-fish__links) [flex: 1]
      |     <h4 id="megaTitle">Fish</h4> — dynamically updates to match selected rail category
      |     5 panels (only one .is-active at a time):
      |       data-panel="fish"      → 9 cards
      |       data-panel="shellfish" → 5 cards
      |       data-panel="smoked"    → 3 cards
      |       data-panel="prepared"  → 5 cards
      |       data-panel="pantry"    → 5 cards
      |     Each card: 2-col grid (text + image), max-width 220px, hover animations
      |
      |-- RIGHT RAIL (.header__dropdown__all-fish__links-hot-this-week-container) [30% width, min 220px]
            <h4>Featured</h4>
            3 promo cards: Best Sellers / New Arrivals / Club Regal
            Images fill full height of card (flex: 0 0 35%, object-fit: cover)
```

### Mobile Menu Structure

```
<nav class="mmenu" id="mmenu" aria-hidden="true">
  <div class="mmenu__inner">
    <button class="mmenu__top mmenu__top--shop">Shop Fish ▾</button>
    <div class="mmenu__shop" hidden>
      <button class="mmenu__group">Fish ▾</button>
        <div class="mmenu__cards" hidden>9 cards</div>
      <button class="mmenu__group">Shellfish ▾</button>
        <div class="mmenu__cards" hidden>5 cards</div>
      <button class="mmenu__group">Smoked & Cured ▾</button>
        <div class="mmenu__cards" hidden>3 cards</div>
      <button class="mmenu__group">Prepared & Easy ▾</button>
        <div class="mmenu__cards" hidden>5 cards</div>
      <button class="mmenu__group">Pantry & More ▾</button>
        <div class="mmenu__cards" hidden>5 cards</div>
    </div>
    <a class="mmenu__top" href="/best-sellers/">Best Sellers</a>
    <a class="mmenu__top" href="/recipes/">Recipes</a>
    <a class="mmenu__top" href="/login.php">Sign in</a>
    <a class="mmenu__top" href="/login.php?action=create_account">Register</a>
    <a class="mmenu__top mmenu__top--club" href="/club-regal">Club Regal</a>
  </div>
</nav>
```

---

## Step 5 — Add JavaScript

Add both JS blocks from `mega-menu.html` (lines 926–1038) before the closing `</body>` tag.

### Script 1: Mega Dropdown Controller (~65 lines)

**Purpose:** Controls hover open/close and left-rail tab switching on desktop.

Key behaviors:
- **Hover intent:** `mouseenter` opens panel, `mouseleave` schedules close with 250ms delay (prevents flicker across gap)
- **Left rail click:** If type already active AND has `data-handle`, navigates to URL. Otherwise swaps active panel and updates `<h4 id="megaTitle">` text to match selected category
- **Mobile tap:** Clicking `.navPages-action` toggles `.is-open` (prevents default link)
- **Resize:** Forces close when viewport exceeds 1024px

Title lookup map: `{ fish: 'Fish', shellfish: 'Shellfish', smoked: 'Smoked & Cured', prepared: 'Prepared & Easy', pantry: 'Pantry & More' }`

### Script 2: Mobile Menu Controller (~48 lines)

**Purpose:** Controls the self-contained `.mmenu` accordion.

Key behaviors:
- **Hamburger toggle:** Click toggles `.is-open` on both button and menu
- **Click outside:** Closes menu when clicking outside
- **Resize:** Force-closes when viewport exceeds 1024px
- **Shop Fish accordion:** Toggles `.mmenu__shop` panel visibility + chevron rotation
- **Group accordions:** Each `.mmenu__group` toggles its adjacent `.mmenu__cards` panel

---

## Step 6 — Preserve BigCommerce Data Attributes

These attributes must be preserved for BigCommerce functionality:

| Attribute | Element | Purpose |
|-----------|---------|---------|
| `data-mobile-menu-toggle="menu"` | Hamburger `<a>` | Links to `#menu` container |
| `data-menu` | `.navPages-container` | Menu container identifier |
| `data-collapsible="navPages-Mega"` | Chevron `<i>` | Links dropdown to trigger |
| `data-cart-preview` | Cart `<a>` | BigCommerce cart preview |
| `data-dropdown="cart-preview-dropdown"` | Cart `<a>` | Dropdown target |
| `data-options="align:right"` | Cart `<a>` | Dropdown alignment |
| `data-search-quick` | Search `<input>` | Quick search binding |
| `data-quick-search-form` | Search `<form>` | Quick search form |
| `data-bind="html: results"` | `.quickSearchResults` | Knockout.js binding |
| `data-content-region="header_bottom"` | Empty `<div>` | BigCommerce widget region |
| `data-stencil-stylesheet` | Theme CSS `<link>` | Stencil attribute |

---

## Step 7 — Category URLs

Ensure these category URLs exist in BigCommerce admin:

| Rail Item | URL | Panel Cards |
|-----------|-----|-------------|
| Fish | `/fish/` | White Fish, Oily Fish, Flatfish, Whole Fish, Fresh Fish, Frozen Fish, Fish Portions, Fish Fillets, Seasonal Picks |
| Shellfish | `/shellfish/` | Prawns, Crab, Lobster, Scallops, Mussels |
| Smoked & Cured | `/smoked-fish/` | Smoked Salmon, Smoked Haddock, Cured Fish |
| Prepared & Easy | `/prepared-fish/` | No Prep Needed, Chef Ready, Fish Cakes, Fish Fingers, Battered & Breaded |
| Pantry & More | `/pantry/` | Tinned Fish, Sauces & Butters, Pantry Essentials, Accessories, Pet Treats |

**Special pages:**
| Page | URL |
|------|-----|
| Best Sellers | `/best-sellers/` |
| New Arrivals | `/new/` |
| Club Regal | `/club-regal/` |
| Recipes | `/recipes/` |

---

## Design Tokens

| Token | Value | Usage |
|-------|-------|-------|
| Navy | `#0d3a5b` | Text, headings |
| Hover navy | `#01427f` | Link hover |
| Gold accent | `#b8a77b` | Hamburger circle, active states, subtitle badges, underline hovers |
| Card borders | `#eee7d6` | Card/rail borders, dividers |
| Warm tint | `#f6f1e4` | Card hover background |
| Rail tint | `#faf7ef` | Shop Fish mobile accordion background |
| Rail divider | `#f0ebe0` | Left rail row borders |
| Font: body | Outfit (400/500/600) | All body text, buttons, cards |
| Font: headings | Gloock (400) | Left rail labels, dropdown titles, promo titles, mmenu group buttons |
| Hover animation | `0.35s ease-out` | Underline bar, image zoom, promo tint |
| Card border radius | `3px` | All cards, promo cards |
| Rail border radius | `0` | Left rail rows (no rounding) |
| Hamburger size | `41.19px` diameter | Gold circle, matches basket.svg |

---

## Notes

- The desktop mega dropdown and mobile mmenu are **completely independent implementations** — the JS controllers do not cross-reference each other
- Every `<img>` in the dropdown and mmenu has `onerror="this.remove()"` for graceful degradation
- The `group-*.jpg` images (5 files) exist in `assets/images/` but are **not currently used** — they were intended for left-rail group header images
- The theme's `theme.css` must load **before** the mega menu overrides for correct cascade
- The original `original-header.html` is a pristine reference — all development goes into `mega-menu.html`
- `index.html` is the showcase page with the new header and download buttons
