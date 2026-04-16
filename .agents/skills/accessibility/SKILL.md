---
name: accessibility
description: >
  WordPress accessibility standards and patterns for Team51 projects targeting
  WCAG 2.1 AA compliance. Activate when building or reviewing themes, blocks,
  forms, navigation, or any user-facing output for accessibility, a11y, ARIA,
  screen readers, keyboard navigation, focus management, or color contrast.
---

# Accessibility (a11y)

## Overview

Team51 projects target **WCAG 2.1 Level AA** compliance. Accessibility is not an optional enhancement — it is an expected baseline for all partner sites. This skill covers WordPress-specific accessibility patterns for themes, blocks, and plugins.

## Semantic HTML

### Use correct elements

- Use `<nav>` for navigation, `<main>` for primary content, `<aside>` for sidebars, `<header>` and `<footer>` for page landmarks.
- Use heading levels (`<h1>`–`<h6>`) in logical order. Never skip levels for styling — use CSS instead.
- Use `<button>` for actions and `<a>` for navigation. Never use `<div>` or `<span>` as interactive elements.
- Use `<ul>` / `<ol>` for lists, `<table>` for tabular data (with `<thead>`, `<th scope>`).

### Landmarks

Every page should have these ARIA landmarks (via semantic HTML or explicit roles):

| Landmark | Element | Notes |
|---|---|---|
| Banner | `<header>` (site-level) | One per page |
| Navigation | `<nav>` | Label with `aria-label` if multiple navs exist |
| Main | `<main>` | One per page |
| Complementary | `<aside>` | Sidebars, related content |
| Content Info | `<footer>` (site-level) | One per page |

When a page has multiple navigations (primary nav, footer nav, breadcrumbs), give each a distinct `aria-label`:

```html
<nav aria-label="<?php esc_attr_e( 'Primary navigation', 'theme-slug' ); ?>">
```

## Keyboard Navigation

### Focus management

- All interactive elements must be reachable via Tab and operable via Enter/Space.
- **Visible focus indicators are required.** Never use `outline: none` without providing an equivalent visible focus style.
- Provide `:focus-visible` styles that meet a 3:1 contrast ratio against adjacent colors.

```css
:focus-visible {
    outline: 2px solid var(--wp--preset--color--primary);
    outline-offset: 2px;
}
```

### Skip link

Every theme must include a skip-to-content link as the first focusable element:

```php
<a class="skip-link screen-reader-text" href="#main-content">
    <?php esc_html_e( 'Skip to content', 'theme-slug' ); ?>
</a>
```

The target (`#main-content`) must exist on the `<main>` element.

### Focus trapping

Modal dialogs, off-canvas menus, and other overlays must trap focus within the component when open, and return focus to the trigger element when closed. Use the WordPress `wp.a11y` utilities or a lightweight trap library.

### Keyboard patterns

| Component | Expected keyboard behavior |
|---|---|
| Dropdown menu | Arrow keys to navigate, Escape to close, Enter to activate |
| Tab panel | Arrow keys switch tabs, Tab moves to panel content |
| Modal dialog | Tab cycles within modal, Escape closes |
| Accordion | Enter/Space toggles, arrows move between headers |
| Carousel/slider | Arrows move slides, pause on focus/hover |

Follow the [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) for widget keyboard patterns.

## ARIA Usage

### Prefer semantic HTML over ARIA

ARIA is a supplement, not a replacement. If a native HTML element provides the semantics, use it:

```html
<!-- BAD -->
<div role="button" tabindex="0" onclick="submit()">Submit</div>

<!-- GOOD -->
<button type="submit">Submit</button>
```

### When ARIA is necessary

- **Live regions** for dynamic content updates (AJAX, real-time data):

```php
<div aria-live="polite" aria-atomic="true">
    <?php echo esc_html( $status_message ); ?>
</div>
```

- **`aria-expanded`** for collapsible elements (accordions, dropdowns).
- **`aria-current="page"`** for the current page in navigation.
- **`aria-describedby`** and **`aria-labelledby`** to associate descriptions and labels.
- **`aria-hidden="true"`** for decorative elements that should be ignored by screen readers.

### WordPress `wp_kses` and ARIA

When using `wp_kses()`, ensure ARIA attributes are in the allowed list:

```php
$allowed = array(
    'div'  => array( 'aria-live' => true, 'aria-atomic' => true, 'class' => true ),
    'span' => array( 'aria-hidden' => true ),
);
echo wp_kses( $html, $allowed );
```

## Screen Reader Text

Use the WordPress `screen-reader-text` CSS class for content that should be available to assistive technology but not visible:

```css
.screen-reader-text {
    clip: rect(1px, 1px, 1px, 1px);
    clip-path: inset(50%);
    height: 1px;
    width: 1px;
    margin: -1px;
    overflow: hidden;
    padding: 0;
    position: absolute;
    word-wrap: normal !important;
}
```

Use it for:

- Additional context on links: `<a href="...">Read more<span class="screen-reader-text"> about {title}</span></a>`
- Icon-only buttons: `<button><svg ...><span class="screen-reader-text">Close menu</span></button>`
- Form field context that is visually implied but not explicit.

## Color and Contrast

### Minimum ratios (WCAG 2.1 AA)

| Element | Minimum contrast ratio |
|---|---|
| Normal text (< 18pt / < 14pt bold) | 4.5:1 |
| Large text (≥ 18pt / ≥ 14pt bold) | 3:1 |
| UI components and graphical objects | 3:1 |
| Focus indicators | 3:1 against adjacent colors |

### theme.json color management

Define accessible color palettes in `theme.json`. Avoid color combinations that fail contrast requirements:

```json
{
  "settings": {
    "color": {
      "palette": [
        { "slug": "primary", "color": "#1a3c5e", "name": "Primary" },
        { "slug": "background", "color": "#ffffff", "name": "Background" }
      ]
    }
  }
}
```

- Do not rely on color alone to convey information (links, errors, status). Provide additional indicators (underlines, icons, text labels).
- Test color combinations with tools like the WebAIM Contrast Checker.

## Forms

- Every `<input>`, `<select>`, and `<textarea>` must have an associated `<label>` (via `for`/`id` pairing or wrapping).
- Mark required fields with both visual indicator and `aria-required="true"` (or the `required` attribute).
- Display error messages adjacent to the field and associate them with `aria-describedby`.
- Group related fields with `<fieldset>` and `<legend>`.
- Ensure form validation messages are announced to screen readers (use `aria-live` or `wp.a11y.speak()`).

## Media

### Images

- All `<img>` elements must have an `alt` attribute.
- **Informative images**: descriptive alt text that conveys the content or function.
- **Decorative images**: empty alt (`alt=""`) and optionally `aria-hidden="true"`.
- **Complex images** (charts, infographics): provide a text alternative nearby or via `aria-describedby`.

### Video and audio

- Provide captions for video content.
- Provide transcripts for audio content.
- Ensure media players are keyboard-operable.
- Auto-playing media must have a visible pause/stop control.

## Block Accessibility

When building custom Gutenberg blocks:

- Use `get_block_wrapper_attributes()` in render output — it preserves ARIA attributes set by the editor.
- Ensure block output uses semantic HTML, not generic `<div>` soup.
- If a block creates interactive UI (tabs, accordions, sliders), implement the full keyboard pattern.
- Test blocks with a screen reader (VoiceOver on macOS, NVDA on Windows) before shipping.
- Use `wp.a11y.speak()` in the editor for dynamic status announcements:

```js
import { speak } from '@wordpress/a11y';

speak( 'Event saved successfully.', 'polite' );
```

## Testing

- **Keyboard**: navigate the entire page using only Tab, Shift+Tab, Enter, Space, Escape, and arrow keys.
- **Screen reader**: test critical flows with VoiceOver (macOS) or NVDA (Windows).
- **Automated**: run axe-core or Lighthouse accessibility audits. These catch ~30% of issues — manual testing is still required.
- **Zoom**: test at 200% and 400% zoom. Content must reflow without horizontal scrolling at 320px equivalent width.
- **Color**: verify contrast ratios for all text and interactive elements.

## Boundaries

- For output escaping and sanitization (relevant to ARIA attribute handling), see `security.md`.
- For CSS conventions (including focus styles), see `coding-standards.md`.
- For block development patterns, see `block-editor-development` skill.
