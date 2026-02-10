# Haze Client Design System Specification
# ctxdc.com-Inspired Hybrid Approach Redesign

**Author**: Architect Agent
**Date**: 2026-02-10
**Status**: Ready for Implementation
**Approach**: Hybrid -- Preserve Functionality Layer, Rebuild Presentation Layer
**Timeline**: 20-30 hours across 3 weeks

---

## Table of Contents

1. [Design Tokens (CSS Custom Properties)](#1-design-tokens)
2. [Typography Scale](#2-typography-scale)
3. [Color System](#3-color-system)
4. [Spacing Scale](#4-spacing-scale)
5. [Component Specifications](#5-component-specifications)
6. [Grid System](#6-grid-system)
7. [Animation Guidelines](#7-animation-guidelines)
8. [Game of Life Background](#8-game-of-life-background)
9. [Migration Plan](#9-migration-plan)
10. [File Inventory & Mapping](#10-file-inventory--mapping)
11. [CSS Class Naming Conventions](#11-css-class-naming-conventions)

---

## 1. Design Tokens

All tokens are defined as CSS custom properties on `:root`. The Engineer should
replace the current `:root` block (index.html lines 42-53) with this complete set.

```css
:root {
  /* ============================================
     COLOR TOKENS
     ============================================ */

  /* Primary Accent -- ctxdc.com cyan/teal */
  --color-primary: #9BD4D7;
  --color-primary-rgb: 155, 212, 215;

  /* Accent variants (opacity-based hierarchy from ctxdc.com) */
  --color-primary-5:  rgba(155, 212, 215, 0.05);
  --color-primary-10: rgba(155, 212, 215, 0.10);
  --color-primary-20: rgba(155, 212, 215, 0.20);
  --color-primary-30: rgba(155, 212, 215, 0.30);
  --color-primary-40: rgba(155, 212, 215, 0.40);
  --color-primary-60: rgba(155, 212, 215, 0.60);
  --color-primary-80: rgba(155, 212, 215, 0.80);

  /* Background Layers (darkest to lightest) */
  --color-bg-base:    #050505;
  --color-bg-surface: #0A0A0A;
  --color-bg-raised:  #111111;
  --color-bg-overlay: #1A1A1A;

  /* Foreground / Text */
  --color-fg-primary:   #EDEDED;
  --color-fg-secondary: rgba(237, 237, 237, 0.60);
  --color-fg-tertiary:  rgba(237, 237, 237, 0.40);
  --color-fg-muted:     rgba(237, 237, 237, 0.20);

  /* Borders -- opacity-based per ctxdc.com pattern */
  --color-border:        rgba(237, 237, 237, 0.10);
  --color-border-hover:  rgba(155, 212, 215, 0.30);
  --color-border-active: rgba(155, 212, 215, 0.60);

  /* Semantic Colors */
  --color-success: #4ADE80;
  --color-warning: #FBBF24;
  --color-error:   #F87171;
  --color-info:    #9BD4D7;

  /* ============================================
     TYPOGRAPHY TOKENS
     ============================================ */

  /* Font Families */
  --font-display: 'tronica', 'JetBrains Mono', monospace;
  --font-body:    'Figtree', 'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono:    'JetBrains Mono', 'Fira Code', 'Consolas', monospace;

  /* Font Sizes -- modular scale (1.25 ratio) */
  --text-xs:   0.75rem;   /* 12px */
  --text-sm:   0.875rem;  /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-md:   1.125rem;  /* 18px */
  --text-lg:   1.25rem;   /* 20px */
  --text-xl:   1.5rem;    /* 24px */
  --text-2xl:  2rem;      /* 32px */
  --text-3xl:  2.5rem;    /* 40px */
  --text-4xl:  3rem;      /* 48px */
  --text-5xl:  3.75rem;   /* 60px */
  --text-6xl:  4.5rem;    /* 72px */

  /* Font Weights */
  --weight-normal:   400;
  --weight-medium:   500;
  --weight-semibold: 600;
  --weight-bold:     700;

  /* Line Heights */
  --leading-none:    1;
  --leading-tight:   1.1;
  --leading-snug:    1.25;
  --leading-normal:  1.5;
  --leading-relaxed: 1.625;
  --leading-loose:   1.8;

  /* Letter Spacing */
  --tracking-tighter: -0.05em;
  --tracking-tight:   -0.025em;
  --tracking-normal:   0;
  --tracking-wide:     0.05em;
  --tracking-wider:    0.1em;
  --tracking-widest:   0.2em;

  /* ============================================
     SPACING TOKENS
     ============================================ */

  /* Spacing Scale (4px base unit) */
  --space-0:    0;
  --space-px:   1px;
  --space-0.5:  0.125rem;  /* 2px */
  --space-1:    0.25rem;   /* 4px */
  --space-1.5:  0.375rem;  /* 6px */
  --space-2:    0.5rem;    /* 8px */
  --space-3:    0.75rem;   /* 12px */
  --space-4:    1rem;      /* 16px */
  --space-5:    1.25rem;   /* 20px */
  --space-6:    1.5rem;    /* 24px */
  --space-8:    2rem;      /* 32px */
  --space-10:   2.5rem;    /* 40px */
  --space-12:   3rem;      /* 48px */
  --space-16:   4rem;      /* 64px */
  --space-20:   5rem;      /* 80px */
  --space-24:   6rem;      /* 96px */
  --space-32:   8rem;      /* 128px */

  /* ============================================
     BORDER TOKENS
     ============================================ */
  --border-width:  1px;
  --border-style:  solid;
  --border-default: var(--border-width) var(--border-style) var(--color-border);

  /* Border Radius */
  --radius-none: 0;
  --radius-sm:   0.25rem;  /* 4px */
  --radius-md:   0.5rem;   /* 8px */
  --radius-lg:   0.75rem;  /* 12px */
  --radius-xl:   1rem;     /* 16px */
  --radius-full: 9999px;

  /* ============================================
     LAYOUT TOKENS
     ============================================ */
  --container-max:  1400px;
  --container-pad:  var(--space-10); /* 40px default, responsive override */
  --grid-columns:   12;
  --grid-gap:       var(--space-6);  /* 24px */

  /* ============================================
     TRANSITION TOKENS
     ============================================ */
  --ease-default:   cubic-bezier(0.4, 0, 0.2, 1);
  --ease-in:        cubic-bezier(0.4, 0, 1, 1);
  --ease-out:       cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out:    cubic-bezier(0.4, 0, 0.2, 1);
  --ease-spring:    cubic-bezier(0.25, 0.46, 0.45, 0.94);

  --duration-fast:    150ms;
  --duration-normal:  250ms;
  --duration-slow:    400ms;
  --duration-slower:  600ms;
  --duration-slowest: 800ms;

  /* ============================================
     Z-INDEX TOKENS
     ============================================ */
  --z-bg:       -2;
  --z-base:      0;
  --z-raised:    1;
  --z-overlay:   10;
  --z-nav:       100;
  --z-modal:     1000;
  --z-cursor:    9999;
}
```

### Dark Mode Note

The site is dark-mode only. No light mode variant is needed. All tokens are
designed for dark backgrounds.

---

## 2. Typography Scale

### 2.1 Font Loading

Replace the current Google Fonts `<link>` tag (index.html line 34) with:

```html
<!-- Primary Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Figtree:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```

For the display font `tronica`/`tronicaMono`, ctxdc.com uses a custom font. Since
this font is not freely available, we use `JetBrains Mono` as the display fallback,
which provides a similar angular, technical aesthetic. If the actual tronicaMono font
file becomes available, add it via `@font-face`:

```css
/* Optional: If tronicaMono font file is obtained */
@font-face {
  font-family: 'tronica';
  src: url('./fonts/tronicaMono.woff2') format('woff2');
  font-display: swap;
  font-weight: 400 700;
}
```

### 2.2 Typography Styles

Each text style maps to a CSS class. The Engineer should add these classes.

```css
/* ============================================
   TYPOGRAPHY CLASSES
   ============================================ */

/* Display -- Hero headlines, section titles */
.text-display-xl {
  font-family: var(--font-display);
  font-size: var(--text-6xl);       /* 72px */
  font-weight: var(--weight-bold);
  line-height: var(--leading-tight); /* 1.1 */
  letter-spacing: var(--tracking-tight);
  color: var(--color-primary);
}

.text-display-lg {
  font-family: var(--font-display);
  font-size: var(--text-4xl);       /* 48px */
  font-weight: var(--weight-bold);
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tight);
  color: var(--color-fg-primary);
}

.text-display-md {
  font-family: var(--font-display);
  font-size: var(--text-3xl);       /* 40px */
  font-weight: var(--weight-bold);
  line-height: var(--leading-snug);  /* 1.25 */
  letter-spacing: var(--tracking-normal);
  color: var(--color-fg-primary);
}

/* Heading -- Section sub-headers */
.text-heading-lg {
  font-family: var(--font-display);
  font-size: var(--text-xl);        /* 24px */
  font-weight: var(--weight-semibold);
  line-height: var(--leading-snug);
  letter-spacing: var(--tracking-wide);
  color: var(--color-fg-primary);
}

.text-heading-md {
  font-family: var(--font-display);
  font-size: var(--text-lg);        /* 20px */
  font-weight: var(--weight-semibold);
  line-height: var(--leading-snug);
  letter-spacing: var(--tracking-wide);
  color: var(--color-fg-primary);
}

.text-heading-sm {
  font-family: var(--font-display);
  font-size: var(--text-base);      /* 16px */
  font-weight: var(--weight-semibold);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-wider);
  text-transform: uppercase;
  color: var(--color-fg-primary);
}

/* Body -- Paragraph text, descriptions */
.text-body-lg {
  font-family: var(--font-body);
  font-size: var(--text-md);        /* 18px */
  font-weight: var(--weight-normal);
  line-height: var(--leading-relaxed);
  color: var(--color-fg-secondary);
}

.text-body-md {
  font-family: var(--font-body);
  font-size: var(--text-base);      /* 16px */
  font-weight: var(--weight-normal);
  line-height: var(--leading-relaxed);
  color: var(--color-fg-secondary);
}

.text-body-sm {
  font-family: var(--font-body);
  font-size: var(--text-sm);        /* 14px */
  font-weight: var(--weight-normal);
  line-height: var(--leading-normal);
  color: var(--color-fg-tertiary);
}

/* Mono -- Tags, labels, technical content */
.text-mono-lg {
  font-family: var(--font-mono);
  font-size: var(--text-sm);        /* 14px */
  font-weight: var(--weight-medium);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-wider);
  text-transform: uppercase;
  color: var(--color-primary);
}

.text-mono-md {
  font-family: var(--font-mono);
  font-size: var(--text-xs);        /* 12px */
  font-weight: var(--weight-medium);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-widest);
  text-transform: uppercase;
  color: var(--color-primary);
}

.text-mono-sm {
  font-family: var(--font-mono);
  font-size: 0.6875rem;             /* 11px */
  font-weight: var(--weight-normal);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-widest);
  text-transform: uppercase;
  color: var(--color-fg-tertiary);
}

/* Caption -- Small labels, metadata */
.text-caption {
  font-family: var(--font-body);
  font-size: var(--text-xs);        /* 12px */
  font-weight: var(--weight-normal);
  line-height: var(--leading-normal);
  color: var(--color-fg-muted);
}
```

### 2.3 Typography Mapping (Current to New)

| Current Usage                  | Current Style                    | New Class           |
|-------------------------------|----------------------------------|---------------------|
| Hero h1 ("Dominate...")       | 72px, DM Sans 700, #00ff88      | `.text-display-xl`  |
| Section titles                | 44px, DM Sans 700               | `.text-display-md`  |
| Split titles                  | 44px, DM Sans 700               | `.text-display-md`  |
| Section tags ("> NEW_IN_V4")  | 14px, JetBrains Mono            | `.text-mono-lg`     |
| Split tags                    | 14px, JetBrains Mono, #00ff88   | `.text-mono-lg`     |
| Body paragraphs               | 17px, DM Sans                   | `.text-body-lg`     |
| Nav links                     | 14px, JetBrains Mono 500        | `.text-mono-lg`     |
| Button text                   | 15px, JetBrains Mono 700        | `.text-mono-lg`     |
| Logo text                     | 24px, JetBrains Mono 700        | `.text-heading-lg`  |
| Stats values                  | 48px, JetBrains Mono 700        | `.text-display-lg`  |
| Card descriptions             | 14px, DM Sans                   | `.text-body-sm`     |
| Version info                  | 14px, JetBrains Mono            | `.text-mono-md`     |

---

## 3. Color System

### 3.1 Palette Summary

**Primary Accent**: `#9BD4D7` (ctxdc.com cyan/teal)
Replaces: `#00ff88` (current bright green)

**Background**: `#050505` (near-black)
Replaces: `#0a0e1a` (current dark blue)

**Foreground**: `#EDEDED` (warm white)
Replaces: `#e2e8f0` (current cool white)

### 3.2 Color Mapping (Current to New)

| Current Token      | Current Value              | New Token               | New Value                       |
|--------------------|---------------------------|-------------------------|---------------------------------|
| `--bg-dark`        | `#0a0e1a`                 | `--color-bg-base`       | `#050505`                       |
| `--bg-light`       | `#1a1f2e`                 | `--color-bg-raised`     | `#111111`                       |
| `--bg-card`        | `#151a28`                 | `--color-bg-surface`    | `#0A0A0A`                       |
| `--border`         | `#2d3748`                 | `--color-border`        | `rgba(237,237,237, 0.10)`       |
| `--accent`         | `#00ff88`                 | `--color-primary`       | `#9BD4D7`                       |
| `--accent-dim`     | `rgba(0,255,136, 0.1)`    | `--color-primary-10`    | `rgba(155,212,215, 0.10)`       |
| `--text`           | `#e2e8f0`                 | `--color-fg-primary`    | `#EDEDED`                       |
| `--text-dim`       | `#94a3b8`                 | `--color-fg-secondary`  | `rgba(237,237,237, 0.60)`       |
| `--yellow`         | `#ffd700`                 | `--color-warning`       | `#FBBF24`                       |
| `--blue`           | `#00bfff`                 | `--color-info`          | `#9BD4D7` (unified with primary)|

### 3.3 Opacity Hierarchy (ctxdc.com Pattern)

ctxdc.com uses foreground color at varying opacity levels instead of separate
gray values. This creates a more cohesive, layered visual hierarchy:

```
Foreground at 100% -- Primary text, headings
Foreground at  60% -- Body text, descriptions
Foreground at  40% -- Tertiary text, placeholders
Foreground at  20% -- Muted text, disabled states
Foreground at  10% -- Borders, dividers
Foreground at   5% -- Subtle backgrounds, hover states
```

### 3.4 Glow Effects

Where the current site uses `box-shadow: 0 0 Xpx #00ff88`, replace with:

```css
/* Subtle glow (borders, small elements) */
box-shadow: 0 0 8px rgba(155, 212, 215, 0.15);

/* Medium glow (cards on hover, active states) */
box-shadow: 0 0 16px rgba(155, 212, 215, 0.20),
            0 0 32px rgba(155, 212, 215, 0.10);

/* Strong glow (hero elements, CTAs) */
box-shadow: 0 0 24px rgba(155, 212, 215, 0.25),
            0 0 48px rgba(155, 212, 215, 0.15),
            0 0 96px rgba(155, 212, 215, 0.05);
```

### 3.5 Theme Color Meta Tag

Replace `<meta name="theme-color" content="#00ff88">` with:

```html
<meta name="theme-color" content="#9BD4D7">
```

---

## 4. Spacing Scale

### 4.1 Context-Specific Spacing

ctxdc.com uses generous padding with clear spacing tiers. Here is the mapping
from ctxdc.com's Tailwind classes to our custom properties:

```
ctxdc.com class  |  Value   |  Our Token     |  Usage
-----------------+----------+----------------+---------------------------
px-4             |  16px    |  --space-4     |  Card horizontal padding
px-6             |  24px    |  --space-6     |  Section horizontal padding
py-2             |   8px    |  --space-2     |  Button vertical padding
py-3             |  12px    |  --space-3     |  Nav item vertical padding
gap-1            |   4px    |  --space-1     |  Tight inline spacing
gap-4            |  16px    |  --space-4     |  Card grid gaps
gap-6            |  24px    |  --space-6     |  Section grid gaps
gap-8            |  32px    |  --space-8     |  Large layout gaps
```

### 4.2 Section Padding Pattern

```css
/* Section padding (ctxdc.com pattern) */
.section {
  padding: var(--space-24) var(--space-10);  /* 96px 40px */
}

/* Mobile override */
@media (max-width: 768px) {
  .section {
    padding: var(--space-16) var(--space-4); /* 64px 16px */
  }
}

/* Phone override */
@media (max-width: 480px) {
  .section {
    padding: var(--space-12) var(--space-4); /* 48px 16px */
  }
}
```

### 4.3 Spacing Mapping (Current to New)

| Current Usage               | Current Value | New Token    | New Value |
|----------------------------|---------------|--------------|-----------|
| Nav padding                | `20px 0`      | `--space-5`  | `20px`    |
| Container padding          | `0 40px`      | `--space-10` | `40px`    |
| Hero padding               | `120px 40px 80px` | `--space-32` top, `--space-10` sides | Responsive |
| Section padding            | `100px 40px`  | `--space-24` top/bottom, `--space-10` sides | `96px 40px` |
| Card padding               | `24-32px`     | `--space-6` to `--space-8` | `24-32px` |
| Button padding             | `18px 48px`   | `--space-3` vertical, `--space-8` horizontal | `12px 32px` |
| Grid gaps                  | `20-40px`     | `--space-4` to `--space-6` | `16-24px` |
| Component margin-bottom    | `24-60px`     | `--space-6` to `--space-16` | Variable |

---

## 5. Component Specifications

### 5.1 Buttons

ctxdc.com uses minimal, border-based buttons with subtle hover effects.
No rounded corners, no filled backgrounds for secondary buttons.

```css
/* ============================================
   BUTTON BASE
   ============================================ */
.btn {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-8);    /* 12px 32px */
  font-family: var(--font-mono);
  font-size: var(--text-sm);                  /* 14px */
  font-weight: var(--weight-medium);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
  text-decoration: none;
  border: var(--border-default);
  background: transparent;
  color: var(--color-fg-primary);
  cursor: pointer;
  transition: color var(--duration-normal) var(--ease-default),
              border-color var(--duration-normal) var(--ease-default),
              background-color var(--duration-normal) var(--ease-default);
}

/* Primary Button -- Filled */
.btn-primary {
  background: var(--color-primary);
  border-color: var(--color-primary);
  color: var(--color-bg-base);
}

.btn-primary:hover {
  background: var(--color-primary-80);
  border-color: var(--color-primary-80);
}

/* Secondary Button -- Outlined */
.btn-secondary {
  background: transparent;
  border-color: var(--color-border);
  color: var(--color-fg-secondary);
}

.btn-secondary:hover {
  border-color: var(--color-primary-40);
  color: var(--color-primary);
}

/* Ghost Button -- No border */
.btn-ghost {
  background: transparent;
  border-color: transparent;
  color: var(--color-fg-secondary);
}

.btn-ghost:hover {
  color: var(--color-primary);
  background: var(--color-primary-5);
}

/* Discord Button -- keep brand color */
.btn-discord {
  background: #5865F2;
  border-color: #5865F2;
  color: #FFFFFF;
}

.btn-discord:hover {
  background: #4752C4;
  border-color: #4752C4;
}

/* Button Sizes */
.btn-sm {
  padding: var(--space-2) var(--space-4);     /* 8px 16px */
  font-size: var(--text-xs);                   /* 12px */
}

.btn-lg {
  padding: var(--space-4) var(--space-12);    /* 16px 48px */
  font-size: var(--text-base);                 /* 16px */
}
```

**Key change from current**: Buttons lose `border-radius: 8px`. ctxdc.com uses
square or very slightly rounded buttons. We use `border-radius: 0` by default or
`--radius-sm` (4px) maximum.

### 5.2 Cards

ctxdc.com cards are border-divided, grid-aligned modules with no rounded corners
and opacity-based border hierarchies.

```css
/* ============================================
   CARD BASE
   ============================================ */
.card {
  background: var(--color-bg-surface);
  border: var(--border-default);
  padding: var(--space-6);                    /* 24px */
  transition: border-color var(--duration-normal) var(--ease-default);
}

.card:hover {
  border-color: var(--color-border-hover);    /* primary at 30% */
}

/* Card with subtle glow on hover */
.card-glow:hover {
  border-color: var(--color-border-active);
  box-shadow: 0 0 16px rgba(155, 212, 215, 0.10);
}

/* Featured / highlighted card */
.card-featured {
  border-color: var(--color-primary-40);
  background: linear-gradient(
    180deg,
    var(--color-primary-5) 0%,
    var(--color-bg-surface) 100%
  );
}

/* Card header with border separator */
.card-header {
  padding-bottom: var(--space-4);
  margin-bottom: var(--space-4);
  border-bottom: var(--border-default);
}

/* Card with inner grid dividers (ctxdc.com pattern) */
.card-divided > * + * {
  border-top: var(--border-default);
  padding-top: var(--space-4);
  margin-top: var(--space-4);
}
```

**Key change from current**: Cards lose `border-radius: 16px`. ctxdc.com uses
sharp edges (0px radius) or maximum `2px` radius. This is a fundamental aesthetic
shift from the current rounded, soft look to sharp, grid-based design.

### 5.3 Navigation

ctxdc.com navigation is minimal, border-bottom separated, with subtle text
color transitions.

```css
/* ============================================
   NAVIGATION
   ============================================ */
nav {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background: rgba(5, 5, 5, 0.90);
  border-bottom: var(--border-default);
  padding: var(--space-4) 0;                  /* 16px vertical */
  z-index: var(--z-nav);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

.nav-container {
  max-width: var(--container-max);
  margin: 0 auto;
  padding: 0 var(--space-10);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  font-family: var(--font-display);
  font-size: var(--text-lg);                  /* 20px -- slightly smaller */
  font-weight: var(--weight-bold);
  color: var(--color-primary);
  letter-spacing: var(--tracking-wider);
  display: flex;
  align-items: center;
  gap: var(--space-3);
}

.nav-links {
  display: flex;
  gap: var(--space-8);                        /* 32px -- tighter */
  list-style: none;
}

.nav-links a {
  font-family: var(--font-mono);
  font-size: var(--text-xs);                  /* 12px -- smaller, ctxdc.com style */
  font-weight: var(--weight-medium);
  letter-spacing: var(--tracking-wider);
  text-transform: uppercase;
  color: var(--color-fg-tertiary);
  text-decoration: none;
  transition: color var(--duration-normal) var(--ease-default);
  padding: var(--space-2) 0;
}

.nav-links a:hover {
  color: var(--color-primary);
}
```

**Key change**: Navigation gets `backdrop-filter: blur(8px)` (ctxdc.com has this).
The current site explicitly has "No blur" in a comment -- this changes that.

### 5.4 Section Headers

```css
/* ============================================
   SECTION HEADERS
   ============================================ */
.section-header {
  text-align: center;
  margin-bottom: var(--space-16);             /* 64px */
}

.section-tag {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  letter-spacing: var(--tracking-widest);
  text-transform: uppercase;
  color: var(--color-primary);
  display: block;
  margin-bottom: var(--space-4);
}

.section-title {
  font-family: var(--font-display);
  font-size: var(--text-3xl);                 /* 40px */
  font-weight: var(--weight-bold);
  line-height: var(--leading-tight);
  color: var(--color-fg-primary);
  margin-bottom: var(--space-4);
}

.section-subtitle {
  font-family: var(--font-body);
  font-size: var(--text-md);                  /* 18px */
  color: var(--color-fg-secondary);
  max-width: 600px;
  margin: 0 auto;
}
```

### 5.5 Border Grid Dividers (ctxdc.com Signature Pattern)

This is one of the most distinctive visual elements of ctxdc.com: content sections
separated by thin border lines, creating a grid-like layout feel.

```css
/* ============================================
   BORDER GRID PATTERN
   ============================================ */

/* Horizontal section divider */
.border-divide {
  border-top: var(--border-default);
}

/* Vertical column divider within a flex/grid row */
.border-divide-x > * + * {
  border-left: var(--border-default);
}

/* Grid with all internal borders (ctxdc.com "border grid" effect) */
.grid-bordered {
  border: var(--border-default);
}

.grid-bordered > * {
  border: var(--border-default);
  /* Collapse borders by using negative margin or by relying on
     the grid gap being 0 and borders overlapping. Simplest approach
     is to add border to each child and let them visually merge: */
}

/* Full-bleed section border pattern */
.section-bordered {
  border-top: var(--border-default);
  border-bottom: var(--border-default);
}
```

### 5.6 Trust Badges / Pill Components

```css
/* ============================================
   BADGES & PILLS
   ============================================ */
.badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-4);     /* 8px 16px */
  border: var(--border-default);
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
  color: var(--color-fg-secondary);
  transition: border-color var(--duration-normal) var(--ease-default),
              color var(--duration-normal) var(--ease-default);
}

.badge:hover {
  border-color: var(--color-border-hover);
  color: var(--color-primary);
}

/* Status dot inside badge */
.badge-dot {
  width: 6px;
  height: 6px;
  background: var(--color-primary);
  border-radius: var(--radius-full);
}
```

**Key change**: Badges lose `border-radius: 50px`. They become square/rectangular
to match ctxdc.com's grid aesthetic. If pills are still desired, use
`border-radius: var(--radius-sm)` (4px) maximum.

### 5.7 Stats / Data Display

```css
/* ============================================
   STATS DISPLAY
   ============================================ */
.stat-value {
  font-family: var(--font-display);
  font-size: var(--text-4xl);                 /* 48px */
  font-weight: var(--weight-bold);
  color: var(--color-primary);
  line-height: var(--leading-none);
}

.stat-label {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  letter-spacing: var(--tracking-widest);
  text-transform: uppercase;
  color: var(--color-fg-tertiary);
  margin-top: var(--space-2);
}

/* Stats row (key-value pair) */
.stat-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-3) 0;                  /* 12px */
  border-bottom: var(--border-default);
}

.stat-row:last-child {
  border-bottom: none;
}
```

### 5.8 Version Badge (Hero)

```css
/* ============================================
   VERSION BADGE
   ============================================ */
.version-badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  background: var(--color-primary-5);
  border: var(--border-default);
  border-color: var(--color-primary-20);
  padding: var(--space-2) var(--space-4);     /* 8px 16px */
  margin-bottom: var(--space-6);
}

.version-badge .dot {
  width: 6px;
  height: 6px;
  background: var(--color-primary);
  border-radius: var(--radius-full);
}

.version-badge span {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  color: var(--color-primary);
  font-weight: var(--weight-medium);
  letter-spacing: var(--tracking-wider);
}
```

---

## 6. Grid System

### 6.1 Base Grid

ctxdc.com uses a 12-column responsive grid. Since there is no build system
(no Tailwind), we define utility classes manually.

```css
/* ============================================
   12-COLUMN GRID SYSTEM
   ============================================ */
.grid {
  display: grid;
  gap: var(--grid-gap);
}

/* Column spans */
.grid-cols-1  { grid-template-columns: repeat(1, 1fr); }
.grid-cols-2  { grid-template-columns: repeat(2, 1fr); }
.grid-cols-3  { grid-template-columns: repeat(3, 1fr); }
.grid-cols-4  { grid-template-columns: repeat(4, 1fr); }
.grid-cols-6  { grid-template-columns: repeat(6, 1fr); }
.grid-cols-12 { grid-template-columns: repeat(12, 1fr); }

/* Span utilities (for items within a 12-col grid) */
.col-span-1  { grid-column: span 1; }
.col-span-2  { grid-column: span 2; }
.col-span-3  { grid-column: span 3; }
.col-span-4  { grid-column: span 4; }
.col-span-6  { grid-column: span 6; }
.col-span-8  { grid-column: span 8; }
.col-span-12 { grid-column: span 12; }

/* Gap variants */
.gap-1 { gap: var(--space-1); }
.gap-2 { gap: var(--space-2); }
.gap-4 { gap: var(--space-4); }
.gap-6 { gap: var(--space-6); }
.gap-8 { gap: var(--space-8); }
```

### 6.2 Container

```css
.container {
  max-width: var(--container-max);
  margin: 0 auto;
  padding: 0 var(--space-10);                /* 40px */
}
```

### 6.3 Breakpoints

```
Breakpoint     Width      Container Pad    Grid Cols Changes
-----------    --------   -------------    ---------------------
Phone (xs)     < 480px    16px             1-col, stacked everything
Phone (sm)     < 768px    20px             1-2 col, hamburger nav
Tablet (md)    < 968px    30px             2-col splits, 2-col features
Desktop (lg)   < 1280px   40px             Full grid, all features
Wide (xl)      > 1280px   40px             Full grid, max-width container
```

### 6.4 Responsive Grid Classes

```css
/* Tablet: 2 columns for feature grids */
@media (max-width: 968px) {
  .grid-cols-4 {
    grid-template-columns: repeat(2, 1fr);
  }
  .grid-cols-3 {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Phone: single column */
@media (max-width: 480px) {
  .grid-cols-4,
  .grid-cols-3,
  .grid-cols-2 {
    grid-template-columns: 1fr;
  }
}
```

### 6.5 Split Section Grid (Current Pattern, Redesigned)

The current site uses `grid-template-columns: 1fr 1fr` for split sections.
Redesign uses the same pattern but with border separation:

```css
.split-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  min-height: 70vh;
  align-items: stretch;                       /* Changed: stretch to fill */
  border-bottom: var(--border-default);       /* Added: border divider */
}

/* Vertical divider between split halves */
.split-content {
  padding: var(--space-20) var(--space-16);   /* 80px 64px */
  border-right: var(--border-default);        /* Added: vertical border */
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.split-visual {
  padding: var(--space-16);                   /* 64px */
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-bg-surface);
}

@media (max-width: 968px) {
  .split-section {
    grid-template-columns: 1fr;
  }
  .split-content {
    border-right: none;
    border-bottom: var(--border-default);
    padding: var(--space-12) var(--space-6);
  }
}
```

---

## 7. Animation Guidelines

### 7.1 Transition Defaults

ctxdc.com uses `transition-colors` (Tailwind class) which maps to only
transitioning color-related properties. This is more performant than
transitioning `all`.

```css
/* Default transition for interactive elements */
.transition-colors {
  transition-property: color, background-color, border-color, fill, stroke;
  transition-timing-function: var(--ease-default);
  transition-duration: var(--duration-normal);  /* 250ms */
}

/* For layout transitions (transforms, opacity) */
.transition-transform {
  transition-property: transform, opacity;
  transition-timing-function: var(--ease-default);
  transition-duration: var(--duration-normal);
}

/* For all properties (use sparingly) */
.transition-all {
  transition-property: all;
  transition-timing-function: var(--ease-default);
  transition-duration: var(--duration-normal);
}
```

### 7.2 Hover Effects (ctxdc.com Pattern)

ctxdc.com hover effects are intentionally subtle. No `transform: translateY()`
shifts, no scale changes. Only color transitions.

```css
/* Standard hover: border color change only */
.hover-border:hover {
  border-color: var(--color-border-hover);
}

/* Text hover: color shift to primary */
.hover-text:hover {
  color: var(--color-primary);
}

/* Card hover: border + subtle glow */
.hover-glow:hover {
  border-color: var(--color-border-hover);
  box-shadow: 0 0 16px rgba(155, 212, 215, 0.08);
}
```

**Key change**: Remove all `transform: translateY(-2px)` hover effects from
buttons. ctxdc.com never lifts elements on hover -- it only changes colors.

### 7.3 Scroll-Triggered Animations

The current site uses a `clip-path: inset()` reveal with a green scan line.
For the ctxdc.com aesthetic, replace with a simpler fade-up:

```css
/* ============================================
   SCROLL REVEAL (SIMPLIFIED)
   ============================================ */
.reveal-section {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity var(--duration-slowest) var(--ease-spring),
              transform var(--duration-slowest) var(--ease-spring);
}

.reveal-section.revealed {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger children */
.reveal-section.revealed > *:nth-child(1) { transition-delay: 0ms; }
.reveal-section.revealed > *:nth-child(2) { transition-delay: 100ms; }
.reveal-section.revealed > *:nth-child(3) { transition-delay: 200ms; }
.reveal-section.revealed > *:nth-child(4) { transition-delay: 300ms; }
```

**Key change**: Remove the scan-line edge effect (`::after` pseudo-element with
green line sweep). Remove `clip-path` animation. Replace with simple opacity +
translateY fade-in. The current glitch-scan aesthetic does not match ctxdc.com's
refined, minimal approach.

### 7.4 Glitch Effect Decision

The current site has elaborate glitch hover effects on buttons (`.btn-glitch`).
ctxdc.com has none of this. Two options:

**Option A (Recommended)**: Remove glitch entirely. Buttons get only
`transition-colors` on hover.

**Option B (Compromise)**: Keep glitch on the hero CTA button only, as a
signature element that differentiates Haze from ctxdc.com. Remove from all
other buttons.

### 7.5 Custom Cursor Decision

The crosshair cursor is unique to Haze and not present in ctxdc.com. However,
it strongly ties to the gaming identity. Recommendation: **Keep it**, but
update the accent color from `#00ff88` to `#9BD4D7` in all cursor CSS.

### 7.6 Preserved Animations (Functionality Layer)

These JavaScript animations must be preserved during migration:
- IntersectionObserver for scroll reveals (update CSS classes only)
- Stats counter animation (update colors only)
- Smooth scroll behavior (no changes needed)
- Hamburger menu toggle (no changes needed)
- Hero icosahedron cursor tilt (update colors only)

---

## 8. Game of Life Background

### 8.1 Overview

Replace the current GLSL shader grid background (Three.js `ShaderMaterial` with
`GRID_VERTEX_SHADER` / `GRID_FRAGMENT_SHADER`) with a Canvas 2D Game of Life
simulation. This matches ctxdc.com's background pattern.

### 8.2 Technical Specification

```
Engine:          Canvas 2D API (no WebGL required)
Grid Resolution: Window width / cellSize x Window height / cellSize
Cell Size:       8px (adjustable via constant)
Colors:
  - Alive cell:  rgba(155, 212, 215, 0.08)  -- primary at 8% opacity
  - Dead cell:   transparent
  - Grid lines:  rgba(237, 237, 237, 0.03)  -- fg at 3% (optional)
Frame Rate:      ~10 FPS (Game of Life updates) with 60fps canvas render
Canvas:          Fixed position, z-index -2, pointer-events none
Fallback:        CSS static grid pattern (existing pattern, updated colors)
```

### 8.3 Implementation Pseudocode

```javascript
(function() {
  'use strict';

  var CELL_SIZE = 8;
  var UPDATE_INTERVAL = 100;  // ms between GoL generations (10 FPS)
  var ALIVE_PROBABILITY = 0.08;  // initial density

  var canvas = document.getElementById('gol-bg-canvas');
  if (!canvas) return;

  var ctx = canvas.getContext('2d');
  var cols, rows;
  var grid, nextGrid;
  var lastUpdate = 0;

  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    cols = Math.ceil(canvas.width / CELL_SIZE);
    rows = Math.ceil(canvas.height / CELL_SIZE);
    initGrid();
  }

  function initGrid() {
    grid = new Uint8Array(cols * rows);
    nextGrid = new Uint8Array(cols * rows);
    for (var i = 0; i < grid.length; i++) {
      grid[i] = Math.random() < ALIVE_PROBABILITY ? 1 : 0;
    }
  }

  function countNeighbors(x, y) {
    var count = 0;
    for (var dy = -1; dy <= 1; dy++) {
      for (var dx = -1; dx <= 1; dx++) {
        if (dx === 0 && dy === 0) continue;
        var nx = (x + dx + cols) % cols;
        var ny = (y + dy + rows) % rows;
        count += grid[ny * cols + nx];
      }
    }
    return count;
  }

  function updateGeneration() {
    for (var y = 0; y < rows; y++) {
      for (var x = 0; x < cols; x++) {
        var idx = y * cols + x;
        var neighbors = countNeighbors(x, y);
        if (grid[idx]) {
          // Alive: survive with 2 or 3 neighbors
          nextGrid[idx] = (neighbors === 2 || neighbors === 3) ? 1 : 0;
        } else {
          // Dead: birth with exactly 3 neighbors
          nextGrid[idx] = (neighbors === 3) ? 1 : 0;
        }
      }
    }
    // Swap grids
    var temp = grid;
    grid = nextGrid;
    nextGrid = temp;
  }

  function render() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = 'rgba(155, 212, 215, 0.08)';

    for (var y = 0; y < rows; y++) {
      for (var x = 0; x < cols; x++) {
        if (grid[y * cols + x]) {
          ctx.fillRect(
            x * CELL_SIZE,
            y * CELL_SIZE,
            CELL_SIZE - 1,  // -1 creates grid gap effect
            CELL_SIZE - 1
          );
        }
      }
    }
  }

  function loop(timestamp) {
    requestAnimationFrame(loop);

    if (timestamp - lastUpdate >= UPDATE_INTERVAL) {
      updateGeneration();
      lastUpdate = timestamp;
    }

    render();
  }

  // --- Periodically inject new cells to prevent extinction ---
  setInterval(function() {
    var count = Math.floor(cols * rows * 0.002);
    for (var i = 0; i < count; i++) {
      var idx = Math.floor(Math.random() * grid.length);
      grid[idx] = 1;
    }
  }, 3000);

  // --- Init ---
  resize();
  window.addEventListener('resize', resize, { passive: true });
  requestAnimationFrame(loop);
})();
```

### 8.4 HTML Changes

Replace the current canvas element:
```html
<!-- OLD: Three.js background canvas -->
<canvas id="haze-bg-canvas" ...></canvas>

<!-- NEW: Game of Life canvas -->
<canvas id="gol-bg-canvas" style="
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -2;
    pointer-events: none;
"></canvas>
```

### 8.5 CSS Fallback (Mobile)

Update the existing CSS grid fallback colors:

```css
.background-grid {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image:
    linear-gradient(rgba(155, 212, 215, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(155, 212, 215, 0.03) 1px, transparent 1px);
  background-size: 8px 8px;                   /* Match GoL cell size */
  z-index: -2;
  pointer-events: none;
}
```

### 8.6 Performance Budget

```
Metric              Target        Measurement
------------------  -----------   --------------------------
Canvas draw calls   1 per frame   Single fillRect batch
Memory (grid)       ~200KB        Two Uint8Array buffers
CPU (update)        <2ms/frame    Measured via performance.now()
Frame rate           60fps render  requestAnimationFrame
GoL update rate      10 FPS       100ms interval
Mobile              DISABLED      CSS fallback only (< 1024px)
```

### 8.7 Hero Icosahedron Decision

The hero icosahedron (Three.js wireframe) can be:

**Option A**: Kept as-is but with color change to `#9BD4D7`
**Option B**: Replaced with a pure CSS/SVG geometric element
**Option C**: Removed entirely for cleaner ctxdc.com aesthetic

**Recommendation**: Option A. The icosahedron is a differentiator. Update its
color from `0x00ff88` to `0x9BD4D7` in the Three.js material:

```javascript
var icoMat = new THREE.MeshBasicMaterial({
  color: 0x9BD4D7,      // Changed from 0x00ff88
  wireframe: true,
  transparent: true,
  opacity: 0.20          // Slightly reduced from 0.25
});
```

Also update the particle color:
```javascript
var pMat = new THREE.PointsMaterial({
  color: 0x9BD4D7,      // Changed from 0x00ff88
  ...
});
```

---

## 9. Migration Plan

### Phase 1: Foundation (Week 1, ~8 hours)

**Goal**: Token system in place, no visual regression yet.

| Step | Task                                          | Hours | Dependencies |
|------|-----------------------------------------------|-------|-------------|
| 1.1  | Add design token `:root` block alongside existing tokens | 0.5 | None |
| 1.2  | Add Google Fonts link for Figtree             | 0.25  | None |
| 1.3  | Add typography utility classes                | 1.0   | 1.1 |
| 1.4  | Add grid utility classes                      | 1.0   | 1.1 |
| 1.5  | Add component base classes (btn, card, badge) | 2.0   | 1.1, 1.3 |
| 1.6  | Implement Game of Life canvas (JS)            | 2.5   | None [P] |
| 1.7  | Test token system without removing old styles | 0.75  | 1.1-1.5 |

`[P]` = Can be done in parallel with other tasks.

### Phase 2: Presentation Swap (Week 2, ~12 hours)

**Goal**: Visual transformation. Old styles replaced with new token-based styles.

| Step | Task                                           | Hours | Dependencies |
|------|------------------------------------------------|-------|-------------|
| 2.1  | Replace `:root` old tokens with new tokens     | 1.0   | Phase 1 |
| 2.2  | Update `body` base styles (font, bg, color)    | 0.5   | 2.1 |
| 2.3  | Restyle navigation (nav, logo, links)           | 1.5   | 2.1, 2.2 |
| 2.4  | Restyle hero section (badge, h1, tagline, CTA)  | 1.5   | 2.1, 2.2 |
| 2.5  | Restyle split sections (content + visual)       | 2.0   | 2.1 |
| 2.6  | Restyle comparison section (cards, grid)        | 1.5   | 2.1, 2.5 |
| 2.7  | Restyle feature grids (apex, anticheat, security) | 2.0 | 2.1, 2.5 |
| 2.8  | Restyle social proof (stats, badges)            | 1.0   | 2.1 |
| 2.9  | Restyle footer                                  | 0.5   | 2.1 |
| 2.10 | Swap GLSL shader for Game of Life canvas        | 0.5   | 1.6 |

### Phase 3: Polish & Responsive (Week 3, ~6 hours)

**Goal**: Responsive testing, animation refinement, final QA.

| Step | Task                                           | Hours | Dependencies |
|------|------------------------------------------------|-------|-------------|
| 3.1  | Update all animation CSS (scroll reveals, hover) | 1.5  | Phase 2 |
| 3.2  | Update crosshair cursor colors                  | 0.5   | Phase 2 |
| 3.3  | Update Three.js icosahedron + particle colors    | 0.5   | Phase 2 |
| 3.4  | Responsive testing: 480px                        | 0.5   | Phase 2 |
| 3.5  | Responsive testing: 768px                        | 0.5   | Phase 2 |
| 3.6  | Responsive testing: 968px                        | 0.5   | Phase 2 |
| 3.7  | Remove all old/dead CSS                          | 1.0   | 3.1-3.6 |
| 3.8  | Performance audit (Lighthouse, load time)        | 0.5   | 3.7 |
| 3.9  | Cross-browser testing (Chrome, Firefox, Edge)    | 0.5   | 3.7 |

### Total Estimated Hours: ~26 hours

---

## 10. File Inventory & Mapping

### Current State: Single File

```
index.html           -- 3,344 lines, all inline
  Lines 1-34:        <head> meta tags, font links
  Lines 35-2037:     <style> block (all CSS, ~2000 lines)
  Lines 2039-2632:   <body> HTML content
  Lines 2634-3063:   Three.js background system (inline JS)
  Lines 3074-3342:   Animation system (inline JS)

logo.png             -- Site logo
debug.html           -- Debug page (not part of migration)
```

### Target State: Still Single File (per constraints)

The fundamental constraint is: no build system, everything in `index.html`.
The migration happens entirely within the existing `<style>` block and the
existing `<script>` blocks.

### CSS Section Reorganization

The current ~2000 lines of CSS should be reorganized into these sections:

```
/* 1. RESET & BASE */           (~20 lines)
/* 2. DESIGN TOKENS */          (~120 lines -- the :root block)
/* 3. TYPOGRAPHY UTILITIES */   (~100 lines)
/* 4. LAYOUT UTILITIES */       (~60 lines -- grid, container)
/* 5. COMPONENT: Navigation */  (~60 lines)
/* 6. COMPONENT: Hero */        (~80 lines)
/* 7. COMPONENT: Buttons */     (~80 lines)
/* 8. COMPONENT: Cards */       (~60 lines)
/* 9. COMPONENT: Badges */      (~40 lines)
/* 10. COMPONENT: Split Section */ (~60 lines)
/* 11. COMPONENT: Stats */      (~40 lines)
/* 12. SECTION: Comparison */   (~60 lines)
/* 13. SECTION: Apex Mode */    (~60 lines)
/* 14. SECTION: Anti-Cheat */   (~60 lines)
/* 15. SECTION: Security */     (~60 lines)
/* 16. SECTION: Social Proof */ (~40 lines)
/* 17. SECTION: Download CTA */ (~40 lines)
/* 18. SECTION: Footer */       (~40 lines)
/* 19. ANIMATIONS */            (~100 lines)
/* 20. RESPONSIVE: 968px */     (~60 lines)
/* 21. RESPONSIVE: 768px */     (~80 lines)
/* 22. RESPONSIVE: 480px */     (~60 lines)
```

**Estimated total CSS**: ~1,300 lines (reduction from ~2,000 due to token reuse
and elimination of duplicated patterns).

---

## 11. CSS Class Naming Conventions

### Convention: BEM-lite with Utility Classes

We use a simplified BEM naming convention. No strict `__` or `--` separators
because the current codebase does not use them and a full BEM migration adds
unnecessary churn.

### Pattern

```
.component              -- Block (e.g., .card, .btn, .nav)
.component-variant      -- Modifier (e.g., .card-featured, .btn-primary)
.component-element      -- Child element (e.g., .card-header, .btn-icon)
.text-{scale}           -- Typography utility
.grid-cols-{n}          -- Grid utility
.col-span-{n}           -- Grid child utility
.gap-{n}                -- Gap utility
.space-{n}              -- Spacing utility (if needed)
```

### Naming Rules

1. All lowercase, hyphen-separated
2. No abbreviations except universally understood ones (`btn`, `nav`, `bg`, `fg`)
3. Component classes are semantic (describe what it IS)
4. Utility classes are functional (describe what it DOES)
5. State classes use `is-` or `has-` prefix (e.g., `is-active`, `has-glow`)
6. JavaScript hook classes use `js-` prefix (e.g., `js-counter`, `js-reveal`)

### Current Class to New Class Mapping (Key Components)

| Current Class              | New Class              | Notes                           |
|---------------------------|------------------------|---------------------------------|
| `.btn-primary`            | `.btn-primary`         | Same name, new styles           |
| `.btn-secondary`          | `.btn-secondary`       | Same name, new styles           |
| `.btn-glitch`             | REMOVED                | Or kept on hero CTA only        |
| `.comparison-card`        | `.card`                | Simplified                      |
| `.comparison-card.highlighted` | `.card-featured`  | New modifier                    |
| `.feature-box`            | `.card`                | Unified card component          |
| `.apex-feature-card`      | `.card`                | Unified card component          |
| `.security-feature-box`   | `.card`                | Unified card component          |
| `.anticheat-feature-card` | `.card`                | Unified card component          |
| `.trust-badge`            | `.badge`               | Unified badge component         |
| `.platform-badge`         | `.badge`               | Unified badge component         |
| `.version-badge`          | `.version-badge`       | Unique enough to keep           |
| `.section-tag`            | `.section-tag`         | Same name, new styles           |
| `.section-title`          | `.section-title`       | Same name, new styles           |
| `.split-section`          | `.split-section`       | Same name, new styles           |
| `.split-content`          | `.split-content`       | Same name, new styles           |
| `.split-visual`           | `.split-visual`        | Same name, new styles           |
| `.reveal-section`         | `.reveal-section`      | Same name, simplified animation |
| `.proof-number`           | `.stat-value`          | More semantic                   |
| `.proof-label`            | `.stat-label`          | More semantic                   |

---

## Appendix A: Color Comparison Visual

```
PROPERTY        CURRENT (Haze)        TARGET (ctxdc.com)
-----------     ------------------    ------------------
Primary         #00ff88 (green)       #9BD4D7 (cyan/teal)
Background      #0a0e1a (dark blue)   #050505 (near-black)
Surface         #151a28 (dark navy)   #0A0A0A (dark gray)
Raised          #1a1f2e (navy)        #111111 (charcoal)
Border          #2d3748 (blue-gray)   rgba(fg, 0.10)
Text Primary    #e2e8f0 (cool white)  #EDEDED (warm white)
Text Secondary  #94a3b8 (slate)       rgba(fg, 0.60)
```

## Appendix B: Key Aesthetic Differences

| Attribute          | Current Haze              | Target (ctxdc.com)         |
|-------------------|---------------------------|----------------------------|
| Corners           | Rounded (8-16px)          | Sharp (0-2px)              |
| Borders           | Subtle, colored (#2d3748) | Prominent, opacity-based   |
| Hover effects     | Transform + glitch        | Color transition only      |
| Spacing           | Medium density            | Generous, airy             |
| Typography mix    | 2 fonts                   | 3 fonts (display/body/mono)|
| Background        | GLSL shader grid          | Game of Life canvas        |
| Visual weight     | Heavy, bold accents       | Light, subtle hierarchy    |
| Button style      | Filled, rounded           | Outlined, square           |
| Section dividers  | Background color change   | Border lines               |

## Appendix C: Preserved Functionality Checklist

These JavaScript features must work identically after migration:

- [ ] Three.js hero icosahedron (color update only)
- [ ] Three.js particle system (color update only, or remove if GoL replaces)
- [ ] Download link tracking (href unchanged)
- [ ] Smooth scroll anchors
- [ ] IntersectionObserver scroll reveals
- [ ] Counter animation on social proof section
- [ ] Custom crosshair cursor (desktop only)
- [ ] Hamburger menu toggle (mobile)
- [ ] Hero visibility gating (IntersectionObserver)
- [ ] Device detection / graceful degradation
- [ ] FPS monitoring / auto-teardown

---

**End of Design System Specification**
**Next Step**: Engineer implements Phase 1 (Foundation) using this document as the single source of truth.
