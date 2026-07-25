# Calendly Contact Embed Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Embed a color-matched Calendly booking calendar in the contact section of the portfolio site, above an "OR" divider and the existing contact form.

**Architecture:** Direct `<iframe>` embed (no Calendly script — the site's CSP keeps `script-src 'none'`). The right column of the `.contact-layout` grid becomes a `.contact-actions` wrapper holding a frosted-glass calendar card, an OR divider, and the untouched existing form. CSP gains `frame-src https://calendly.com`.

**Tech Stack:** Static HTML/CSS (single `index.html`, styles inline in `<head>`). No build step, no JS.

## Global Constraints

- Spec: `docs/superpowers/specs/2026-07-25-calendly-contact-embed-design.md`
- Calendar label copy, verbatim: **"Book a 30 min time slot to get started"**
- Calendly URL params, verbatim: `embed_type=Inline&embed_domain=embodiedactorsstudio.com&hide_gdpr_banner=1&background_color=1a4d2e&text_color=ffffff&primary_color=7dd3c0`
- `script-src 'none'` must remain in the CSP — do not add any `<script>` tags.
- No changes to the Formspree form fields, `thanks.html`, or any other section.
- File is Prettier-formatted (4-space indent, wrapped attributes) — match that style.
- No automated test suite exists; each task verifies in the browser (site is static — open `index.html` directly) plus a console check for CSP violations.

---

### Task 1: CSP update + Calendly iframe HTML

**Files:**
- Modify: `index.html:8` (CSP meta content)
- Modify: `index.html:2298-2384` (wrap form, add calendar card + divider markup)

**Interfaces:**
- Produces: class names `contact-actions`, `calendly-card`, `calendly-label`, `calendly-frame`, `contact-divider` — Task 2 styles exactly these.

- [ ] **Step 1: Add `frame-src` to the CSP in `index.html` (line 8)**

Replace the CSP `content` value so it reads (only change: `frame-src https://calendly.com;` inserted after `form-action`):

```
default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self' data:; form-action 'self' https://formspree.io; frame-src https://calendly.com; script-src 'none'; frame-ancestors 'none'; base-uri 'self';
```

- [ ] **Step 2: Add the calendar card, divider, and wrapper in the contact section**

In `index.html`, directly after the `</div>` closing `.contact-info` (line 2296) and before the `<form` tag (line 2298), the second grid child becomes a wrapper. Insert before the form:

```html
<div class="contact-actions">
    <div class="calendly-card">
        <div class="calendly-label">
            Book a 30 min time slot to get started
        </div>
        <iframe
            class="calendly-frame"
            src="https://calendly.com/debhansen38/30min?embed_type=Inline&embed_domain=embodiedactorsstudio.com&hide_gdpr_banner=1&background_color=1a4d2e&text_color=ffffff&primary_color=7dd3c0"
            title="Schedule a session with Deb Hansen"
            loading="lazy"
        ></iframe>
    </div>

    <div class="contact-divider">OR</div>
```

Then close the wrapper: after the `</form>` tag (line 2384), add `</div>` before the closing `</div>` of `.contact-layout`. The form element and all its contents are NOT modified — only re-indented one level if keeping Prettier formatting.

- [ ] **Step 3: Verify in browser**

Open `index.html` in Chrome. Expected: contact section shows label + Calendly calendar (may render with default colors pending Task 2 card styling, but the iframe must load), "OR" text, then the form. Console shows **no CSP violation** mentioning calendly.com. The calendar is interactive (month grid clickable).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add Calendly iframe embed and OR divider to contact section"
```

---

### Task 2: Calendar card + divider CSS

**Files:**
- Modify: `index.html:1190-1196` area (insert new rules after `.btn-submit:hover`, before the `/* Footer */` comment at line 1197)

**Interfaces:**
- Consumes: class names from Task 1 (`calendly-card`, `calendly-label`, `calendly-frame`, `contact-divider`).

- [ ] **Step 1: Insert the new CSS rules**

After the `.btn-submit:hover` block (ends line 1195) and before `/* Footer */`, insert:

```css
            /* Calendly Booking Card + OR Divider */
            .calendly-card {
                background-color: rgba(255, 255, 255, 0.05);
                backdrop-filter: blur(10px);
                -webkit-backdrop-filter: blur(10px);
                padding: 2rem;
                border: 1px solid rgba(255, 255, 255, 0.1);
            }

            .calendly-label {
                font-size: 0.75rem;
                font-weight: 600;
                letter-spacing: 2px;
                text-transform: uppercase;
                color: var(--light-teal);
                margin-bottom: 1.25rem;
            }

            .calendly-frame {
                display: block;
                width: 100%;
                height: 700px;
                border: 0;
            }

            .contact-divider {
                display: flex;
                align-items: center;
                gap: 1.25rem;
                margin: 2.5rem 0;
                color: var(--light-teal);
                font-size: 0.85rem;
                font-weight: 600;
                letter-spacing: 3px;
            }

            .contact-divider::before,
            .contact-divider::after {
                content: "";
                flex: 1;
                height: 1px;
                background-color: rgba(255, 255, 255, 0.2);
            }
```

- [ ] **Step 2: Verify styling in browser (desktop)**

Reload `index.html` (hard refresh). Expected: calendar sits in a frosted card visually matching the form card; teal uppercase label above it; OR divider shows thin translucent lines either side of teal "OR"; Calendly interior shows dark-green background, white text, teal accents. If Calendly ignores the color params (interior renders white), note it and flag to the user per spec fallback — do not improvise a fix.

- [ ] **Step 3: Verify mobile layout**

Narrow the window below the `.contact-layout` breakpoint (single column). Expected order: contact info → calendar card → OR → form; no horizontal overflow; calendar scrolls internally.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Style Calendly card and OR divider to match contact section"
```
