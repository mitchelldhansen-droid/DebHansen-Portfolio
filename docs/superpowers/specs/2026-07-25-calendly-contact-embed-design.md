# Calendly Embed in Contact Section — Design

**Date:** 2026-07-25
**File affected:** `index.html` only

## Goal

Add a Calendly scheduling calendar to the contact section so visitors can either
book a time slot directly or use the existing contact form. Visual order:
"Get In Touch" heading → calendar → "OR" divider → contact form. Styling must
match the site's dark green/teal palette.

## Embed Method: Direct iframe (no Calendly script)

The site's CSP intentionally sets `script-src 'none'`. Calendly's official
`widget.js` would be blocked — and we don't want to weaken that policy.
For inline embeds, `widget.js` only constructs an iframe, so we author the
iframe directly:

```html
<iframe
  src="https://calendly.com/debhansen38/30min?embed_type=Inline&embed_domain=embodiedactorsstudio.com&hide_gdpr_banner=1&hide_event_type_details=1&background_color=1a4d2e&text_color=ffffff&primary_color=7dd3c0"
  width="100%" height="700" frameborder="0"
  title="Schedule a session with Deb Hansen"></iframe>
```

- Color params (hex without `#`): background `1a4d2e` (--dark-green), text
  `ffffff`, primary/accent `7dd3c0` (--light-teal).
- CSP change: add `frame-src https://calendly.com;` to the policy in
  `index.html`. `script-src 'none'` stays. `thanks.html` unchanged.
- Verification step: confirm in the browser that Calendly honors the color
  params. If it does not (plan-dependent theming), fall back to a light-card
  look and revisit.

## Layout (revised during implementation: header-on-top, side-by-side)

Original design stacked calendar/OR/form in the right column; user revised to
the side-by-side option after seeing it in the browser. Final structure:

1. **`.contact-header`** — centered block on top: eyebrow ("Get Started"),
   "Get In Touch" h2, intro paragraph ending with the exact copy
   **"In person preferred, Google Meet available."**, then Location/Serving
   details in a centered flex row (`.contact-header-details`).
2. **`.contact-layout`** — grid `1fr auto 1fr`:
   - **Calendar card** (left) — frosted-glass styling matching `.contact-form`
     (`rgba(255,255,255,0.05)` background, backdrop blur, 1px translucent
     border, padding). Inside: a teal uppercase label with the exact copy
     **"Book a 30 min time slot to get started"**, then the iframe.
   - **OR divider** (middle) — vertical: thin `rgba(255,255,255,0.2)` lines
     above/below "OR" in teal, uppercase, letter-spaced; `align-self: stretch`.
   - **Existing contact form** (right) — untouched (Formspree action,
     honeypot, fields all as-is).

The iframe URL also includes `hide_event_type_details=1` — the user does not
want the event's meeting address advertised on the page (it still appears
inside the Calendly booking flow, which is acceptable).

## Responsive behavior

The existing mobile breakpoint collapses `.contact-layout` to one column
(stack order: header → calendar → OR → form), flips the divider back to
horizontal lines, and stacks the header details. Iframe keeps
`height: 700px` (min-width 320px is satisfied at all viewports) and scrolls
internally.

## Out of scope

- No changes to `thanks.html`, Formspree flow, or other sections.
- No Calendly popup/badge widgets.
- No JavaScript.
