# CSS Analysis Report: main.css

**Scope:** Root `main.css` (primary stylesheet; all HTML pages link to it).  
**Constraint:** Analysis and recommendations only — no code changes applied.

---

## 1. Current Responsiveness Issues

### 1.1 Fixed widths that block fluid layout

- **Hero**
  - `.hero-image img`: `width: 360px; max-width: 360px` — fixed size; only overridden in a 768px media query to `280px` / `85vw`. Between ~360px and 768px viewport the image does not scale down.
  - `.hero-content`: `max-width: 520px` (fine). `.hero-content p`: `max-width: 460px` (fine).
  - `.hero-section::after` (watermark): `background-size: 780px auto` — fixed; at 768px it’s set to `520px auto`. No scaling for intermediate widths or ultrawide.

- **About section (homepage / projects / etc.)**
  - `.about-image img`: `width: 320px; max-width: 320px` — same pattern as hero image; no shrink until media query.
  - `.about-content`: `max-width: 485px` (acceptable for content column).

- **Decorative blurs**
  - `.about-section::before`: `width: 950px; height: 950px` — fixed size; can overflow or sit oddly on narrow or ultrawide.

- **Footer**
  - `.footer-logo`: `width: 56px; max-width: 56px` — fixed; minor but could be `max-width` only for consistency.

- **Vuln-PLC**
  - `.vulnplc-logo`: `width: 360px; max-width: 360px`.
  - `.vulnplc-overlay`: `width: 380px; max-width: 95%` — 95% helps; the 380px still sets a large default.

- **Page-level content**
  - Hero inners (services, contact, about, policy): `max-width: 900px` — good.
  - Content blocks (services-detail, contact-content, policy-content, about-content): `max-width: 1100px` — good.
  - All use fixed horizontal padding (e.g. `2.5rem`, `2.4rem`) with no reduction on small screens.

### 1.2 Absolute positioning and overlap risk

- **`.demo-hero-scale .hero-image`**: `position: absolute; right: 2.5rem; bottom: 2.25rem`. On narrow desktop / large tablet (e.g. 800–900px) the logo can overlap the hero text; only at 768px does it switch to `position: relative` and stack.
- **`.vulnplc-overlay`**: Centered with `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)` and fixed `width: 380px`. Below 900px it correctly becomes `position: static` and stacks, but there is no intermediate handling for tablet.

### 1.3 Single breakpoint and missing ranges

- **Only two breakpoints are used:** `768px` (mobile) and `900px` (vuln-plc gallery and overlay).
- **No tablet breakpoint** (e.g. 1024px): between 768px and ~1320px the layout is the same as desktop; header nav, hero, and services grid do not adapt for tablet.
- **No large-desktop / ultrawide:** Content is capped at 1320px/1100px/900px (good), but decorative elements (watermark, blur orbs) stay fixed in px and can look small or off on very wide screens.
- **Header:** No media query. On viewports where the nav + CTA don’t fit (e.g. ~900px), they can wrap awkwardly or overflow; there is no hamburger or stacked nav.
- **Footer:** `.footer-container` is flex with `justify-content: space-between` and no wrap; on small screens the logo and links can collide or overflow.

### 1.4 Services grid not responsive in the middle range

- **`.services-grid`**: `grid-template-columns: repeat(2, 1fr)` with no breakpoint. Cards stay two columns at all widths; only hero/about get a column layout at 768px. On phones the two-column grid is cramped; a breakpoint (e.g. 600px or 768px) to `1fr` would stack them.

### 1.5 Fixed horizontal padding everywhere

- Containers use `padding: 0 2.5rem` (or `2.4rem`, `2.3rem`). On very narrow viewports (e.g. 320px) 2.5rem (40px) each side leaves little room. No `clamp()` or smaller padding in a media query.

---

## 2. Safe Improvements That Preserve Existing Design

### 2.1 Replace fixed width + max-width with max-width (and optional width: 100%)

- **Images that must not grow but should shrink:** Use only `max-width` (and optionally `width: 100%`) so they scale down with the viewport. Preserve current “desktop” size by keeping that value as `max-width`.
  - `.hero-image img`: e.g. `max-width: 360px; width: 100%; height: auto` (remove fixed `width: 360px`).
  - `.about-image img`: same idea with `320px`.
  - `.footer-logo`: `max-width: 56px; width: auto`.
  - `.vulnplc-logo`: same as hero image.
  - **Result:** Same look at current desktop widths; better behavior at intermediate and small widths without changing layout at 1320px.

### 2.2 Responsive horizontal padding

- Replace fixed `2.5rem` (and similar) with something like `clamp(1rem, 5vw, 2.5rem)` for left/right padding on:
  - `.header-container`, `.hero-container`, `.about-container`, `.footer-container`, `.services-grid`, and page-level hero/content blocks.
- **Result:** Same at desktop; less padding on small screens, no extra space on ultrawide.

### 2.3 Hero watermark (::after) scaling

- Keep `background-size: 780px auto` for desktop. In the existing 768px query, you already use `520px auto`. Optionally add an intermediate size (e.g. at 1024px) so it scales gradually (e.g. `min(780px, 60vw)` or similar) instead of a single jump at 768px. Use only `background-size` so position and opacity stay as-is.

### 2.4 About section blur (::before)

- Replace fixed `950px` with something like `min(950px, 100vmax)` or a percentage so the orb doesn’t overflow on narrow viewports. Keep position and blur so the visual effect stays the same at desktop.

### 2.5 Services grid stacking on small screens

- Add a single rule inside the existing `@media (max-width: 768px)` (or a new `max-width: 600px`):  
  `.services-grid { grid-template-columns: 1fr; }`  
  So below that width cards stack; above it, two columns are unchanged. No change to card styling.

### 2.6 Tablet breakpoint (additive only)

- Add `@media (max-width: 1024px)` (or 992px) with **only**:
  - Slightly smaller hero watermark if desired.
  - Optional: slightly reduce `.demo-hero-scale .hero-image` right/bottom so it doesn’t touch text.
  Do **not** change flex direction, typography, or colors so desktop appearance is unchanged; use this only to tweak spacing/sizing.

### 2.7 Footer and header overflow

- **Footer:** Add `flex-wrap: wrap` and a small `gap` to `.footer-container` so on narrow screens the two groups wrap instead of colliding. No need to change alignment on desktop.
- **Header:** Add `flex-wrap: wrap` to `.header-container` and, if needed, a small gap so nav + CTA can wrap. Optional: in a 768px (or 1024px) query, reduce nav link spacing (e.g. `gap`) so more fits on one line before wrapping.

---

## 3. CSS Efficiency Improvements

### 3.1 Duplicate mobile block

- **Two separate `@media (max-width: 768px)` blocks** (lines ~222–250 and ~402–429) both set:
  - `.hero-container { flex-direction: column; align-items: center; text-align: center; }`
  - `.hero-image img { width: 280px; max-width: 85vw; height: auto; }`
  - And the second also disables backdrop-filter, hides hero/about pseudo-elements, and reduces image filter.
- **Recommendation:** Merge into one `@media (max-width: 768px)` block so mobile rules live in one place and order is clear (stability/visual fixes + layout). This avoids redundancy and possible order-dependent overrides.

### 3.2 Conflicting / overloaded class: `.about-content`

- **First definition (ABOUT SECTION):** `.about-content { flex: 1; max-width: 485px; }` — intended for the text column inside `.about-section` (index, demo, projects).
- **Second definition (ABOUT PAGE):** `.about-content { max-width: 1100px; margin: 0 auto; padding: 3.5rem 2.5rem 5rem; display: grid; gap: 2.2rem; }` — intended for the content wrapper on the About page (`<section class="about-content">`).
- Because the second comes later, **every** element with class `about-content` gets the grid/1100px rules, including the div inside `.about-section` on the homepage. That can break the intended flex layout (content column becoming a full-width grid).
- **Recommendation:** Scope so they don’t conflict:
  - `.about-section .about-content` for the column (flex: 1; max-width: 485px).
  - `.about-page .about-content` for the grid wrapper (max-width: 1100px; grid; etc.).
  No visual change when scoped correctly; layout and predictability improve.

### 3.3 Duplicate `.vulnplc-card figcaption`

- `.vulnplc-card figcaption` is defined twice (lines ~619–625 and ~659–670). The second overrides the first. Keep one block (prefer the more complete second) and remove the other to avoid confusion.

### 3.4 Repeated card styles

- `.service-block`, `.contact-card`, `.about-card`, and `.policy-card` share the same gradient, border, border-radius, padding, and box-shadow. You could introduce a shared class (e.g. `.card-base` or `.content-card`) and add it to each, then keep only page-specific overrides (e.g. heading size). **Optional:** Reduces duplication and keeps future design changes in one place; do only if you’re comfortable touching HTML.

### 3.5 Selector simplicity

- Many sections use a flat class (e.g. `.service-card h3`) which is fine. Where you have multiple definitions for the same class (e.g. `.about-content`), scoping as above also simplifies mental model and reduces specificity wars.

---

## 4. Optional Enhancements

- **Optional – Ultrawide:** For `min-width: 1600px` or similar, consider scaling the hero watermark or blur orbs with `vmin`/`vmax` so they don’t look tiny. Keep all other layout and colors as-is.
- **Optional – Touch targets:** In the 768px block, ensure `.header-nav a` and `.btn-consultation` have at least ~44px min-height or padding for touch. Does not change desktop.
- **Optional – Reduce motion:** Add a `@media (prefers-reduced-motion: reduce)` block that sets `transition: none` (or very short) on `.service-card`, `.btn-hero`, `.btn-consultation`, and similar. Preserves layout and colors.
- **Optional – High contrast / focus:** Ensure `:focus-visible` on nav and buttons has a visible outline (e.g. 2px solid accent) without changing default appearance.

---

## 5. Unused / Dead CSS

### 5.1 `css/main.css` is not loaded

- **Every HTML file** uses `<link rel="stylesheet" href="main.css">` (root `main.css`). **No page** links to `css/main.css`.
- `css/main.css` defines a different system (e.g. `:root` variables, `.header-inner`, `.logo`, `.main-nav`, `.hero`, `.section-intro`, `.service-grid`, `.button`) that does not match the current HTML (which uses `.header-container`, `.header-logo`, `.header-nav`, `.hero-section`, `.hero-container`, etc.).
- **Conclusion:** `css/main.css` is unused. It can be removed or archived if you want to avoid confusion; it has no effect on the live site.

### 5.2 In root `main.css`

- No other obviously dead selectors were found. The duplicate mobile block and the `.about-content` conflict are correctness/maintainability issues, not unused rules.

---

## Summary

- **Responsiveness:** Fixed image widths, a single mobile breakpoint, no tablet or ultrawide handling, and absolute positioning in the hero cause the site not to adapt cleanly across mobile, tablet, desktop, and ultrawide. Services grid stays two columns; header/footer can overflow.
- **Safe improvements:** Use `max-width` (and `width: 100%` where appropriate) for key images, fluid horizontal padding (`clamp`), one extra breakpoint for services grid stacking, optional tablet tweaks, and scoped `.about-content` so layout is correct on all pages. These preserve current look and behavior at desktop.
- **Efficiency:** Merge the two 768px blocks, scope `.about-content` (`.about-section` vs `.about-page`), remove duplicate `.vulnplc-card figcaption`, and optionally unify card styles.
- **Dead code:** `css/main.css` is unused; root `main.css` is the only active stylesheet.

No changes have been applied; this report is diagnostic and recommendation-only.
