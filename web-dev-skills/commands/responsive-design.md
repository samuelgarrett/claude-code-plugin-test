# Responsive Design Expert

**Description**: Create fluid, mobile-first designs that work beautifully across all screen sizes

## Core Principles

Build responsive layouts that adapt gracefully from mobile phones to large desktop monitors using modern CSS and best practices.

## Mobile-First Approach

```css
/* ✅ Mobile-first: Start with mobile, enhance for larger screens */
.container {
  padding: 1rem;
  width: 100%;
}

@media (min-width: 768px) {
  .container {
    padding: 2rem;
    max-width: 720px;
  }
}

@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 960px;
  }
}

/* ❌ Desktop-first: Harder to maintain */
.container {
  padding: 3rem;
  max-width: 960px;
}

@media (max-width: 1024px) {
  .container {
    padding: 2rem;
    max-width: 720px;
  }
}
```

## Breakpoints

```css
/* Standard breakpoints */
/* Mobile: < 640px (default) */
/* Tablet: 640px - 1024px */
/* Desktop: > 1024px */

:root {
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
}

/* Tailwind-style breakpoints */
@media (min-width: 640px) { /* sm */ }
@media (min-width: 768px) { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
```

## Fluid Layouts

### CSS Grid
```css
/* Responsive grid without media queries */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

/* 12-column grid system */
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1rem;
}

.col-span-6 {
  grid-column: span 6; /* 50% width */
}

@media (max-width: 768px) {
  .col-span-6 {
    grid-column: span 12; /* Full width on mobile */
  }
}
```

### Flexbox
```css
/* Responsive flex layout */
.flex-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.flex-item {
  flex: 1 1 300px; /* grow, shrink, basis */
}

/* Center content */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## Responsive Typography

```css
/* Fluid typography using clamp */
h1 {
  font-size: clamp(2rem, 5vw, 4rem);
  /* min: 2rem, preferred: 5vw, max: 4rem */
}

body {
  font-size: clamp(1rem, 2.5vw, 1.25rem);
}

/* Using CSS custom properties */
:root {
  --font-size-base: 16px;
  --font-size-lg: 18px;
}

@media (min-width: 768px) {
  :root {
    --font-size-base: 18px;
    --font-size-lg: 20px;
  }
}

body {
  font-size: var(--font-size-base);
}
```

## Responsive Images

```html
<!-- Modern responsive images -->
<picture>
  <source
    media="(min-width: 1024px)"
    srcset="hero-desktop.webp 1x, hero-desktop@2x.webp 2x"
  >
  <source
    media="(min-width: 768px)"
    srcset="hero-tablet.webp 1x, hero-tablet@2x.webp 2x"
  >
  <img
    src="hero-mobile.webp"
    srcset="hero-mobile@2x.webp 2x"
    alt="Hero image"
    loading="lazy"
  >
</picture>

<!-- srcset with sizes -->
<img
  src="photo.jpg"
  srcset="photo-400w.jpg 400w,
          photo-800w.jpg 800w,
          photo-1200w.jpg 1200w"
  sizes="(max-width: 600px) 100vw,
         (max-width: 1200px) 50vw,
         33vw"
  alt="Photo"
>
```

## Container Queries (Modern)

```css
/* Responsive based on container size, not viewport */
.card-container {
  container-type: inline-size;
  container-name: card;
}

.card-title {
  font-size: 1.5rem;
}

@container card (min-width: 400px) {
  .card-title {
    font-size: 2rem;
  }

  .card-content {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

## Responsive Navigation

```css
/* Mobile hamburger menu */
.nav-toggle {
  display: block;
}

.nav-menu {
  display: none;
  flex-direction: column;
}

.nav-menu.active {
  display: flex;
}

@media (min-width: 768px) {
  .nav-toggle {
    display: none;
  }

  .nav-menu {
    display: flex;
    flex-direction: row;
  }
}
```

## Testing Responsive Design

### Viewport Meta Tag
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Device Testing Checklist
- [ ] iPhone SE (375px)
- [ ] iPhone 12/13 (390px)
- [ ] iPad (768px)
- [ ] Desktop (1024px+)
- [ ] Large desktop (1920px+)
- [ ] Test landscape orientations
- [ ] Test with browser zoom (125%, 150%)

## When to Use This Skill

- Designing responsive layouts
- Converting desktop designs to mobile
- Optimizing for different screen sizes
- Improving mobile user experience

---

**Remember**: Design for mobile first, then progressively enhance for larger screens!
