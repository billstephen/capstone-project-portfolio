# Copilot-Assisted Development — Documentation

**Project:** Mini Portfolio App  
**Developer:** Alex Mercer  
**Tool:** GitHub Copilot (GPT-4o model)  
**Date:** 2025

---

## Overview

This document summarises how GitHub Copilot was used throughout the development
of the portfolio single-page application.  Each section below maps a project
requirement to the specific Copilot-assisted contribution, the prompt strategy
used, and the acceptance criteria applied before committing the generated code.

---

## 1. Base Layout & Navigation Structure

**Requirement:** Responsive single-page layout with fixed navigation.

**Copilot contribution:**
- Generated the semantic HTML skeleton (`<nav>`, `<section>`, `<footer>`) from
  a natural-language comment: `// responsive SPA with sticky nav, hero, about,
  projects, contact, footer — dark theme`.
- Suggested the CSS custom-property system (`:root` block with colour, font, and
  spacing tokens) after seeing the first two variable declarations.
- Auto-completed the `nav.scrolled` frosted-glass rule once the JS scroll
  listener was scaffolded.

**Review notes:**
- Verified that landmark roles (`role="navigation"`, `aria-label`) were present
  for accessibility; Copilot included them on the first suggestion.
- Adjusted the responsive breakpoint from `600px` to `768px` to match the
  project's target tablet viewport.

---

## 2. JSON-Driven Project Gallery

**Requirement:** Populate project cards from a structured JSON/JS data array.

**Copilot contribution:**
- Generated the `PROJECTS` array schema after a comment describing the required
  fields (`id`, `title`, `description`, `category`, `stack`, `icon`, `liveUrl`,
  `repoUrl`).
- Wrote the `renderProjects(projects)` function, including the `.map()` template
  literal that builds each `<article>` card, after a JSDoc comment was typed.
- Suggested staggered `setTimeout` fade-in for newly rendered cards unprompted.

**Functions generated with Copilot:**

| Function | Prompt trigger |
|---|---|
| `renderProjects(projects)` | JSDoc `@param` comment |
| `getCategories(projects)` | Inline comment `// extract unique categories` |
| `renderFilters(categories)` | JSDoc `@param` comment |
| `handleFilter(e)` | Inline comment `// re-render on filter click` |

**Review notes:**
- Replaced Copilot's initial `innerHTML +=` loop with a single `innerHTML =`
  assignment to avoid repeated DOM reflows.
- Added `aria-label` attributes to card links that Copilot omitted on first pass.

---

## 3. Contact Form Input Validation

**Requirement:** Client-side validation for name, email, phone, subject, and
message fields with clear inline error messages.

**Copilot contribution:**
- Generated the `setFieldValidity(inputEl, errorEl, isValid)` helper from a
  comment describing toggling `.error` and `.visible` CSS classes.
- Wrote the full submit handler logic after a comment: `// validate all fields,
  show errors, simulate async submit on success`.
- Suggested the `phoneHint` span content and real-time formatting feedback
  unprompted after seeing the phone field markup.

**Validation rules implemented:**

| Field | Rule | Function |
|---|---|---|
| Name | ≥ 2 characters | inline in submit handler |
| Email | RFC-5322-style regex | `validateEmail(email)` |
| Phone | 10 digits (optional) | `validatePhone(phone)` |
| Subject | Non-empty select | inline in submit handler |
| Message | ≥ 20 characters | `validateMessage(msg, min)` |

---

## 4. Helper / Utility Functions

**Requirement:** Standalone, tested helper functions with JSDoc documentation.

All four helpers below were generated primarily by Copilot:

### `validateEmail(email) → boolean`
**Prompt:** JSDoc comment with `@param {string} email` and `@returns {boolean}`.  
**Output:** Copilot generated the `/^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/` regex and
the `.trim().toLowerCase()` normalisation step.  
**Acceptance:** Manually tested against 8 valid and 6 invalid addresses.

### `validatePhone(phone) → boolean`
**Prompt:** Inline comment `// US phone: exactly 10 digits, ignore formatting`.  
**Output:** Copilot produced `phone.replace(/\D/g, '').length === 10`.  
**Acceptance:** Tested with formatted `(555) 123-4567`, raw `5551234567`,
international `+1-555-123-4567` (should fail), and empty string (optional field).

### `formatPhoneNumber(raw) → string`
**Prompt:** Inline comment `// real-time (XXX) XXX-XXXX formatter`.  
**Output:** Copilot wrote the digit-slicing logic with three conditional branches
covering partial input states (1–3 digits, 4–6, 7–10).  
**Acceptance:** Verified character-by-character formatting in the browser input
field; caret position restoration handled manually post-generation.

### `validateMessage(msg, min = 20) → boolean`
**Prompt:** JSDoc `@param` with optional `min` parameter.  
**Output:** `msg.trim().length >= min` with the default parameter suggestion.  
**Acceptance:** Confirmed default of 20 chars; edge-tested with whitespace-only input.

---

## 5. Inline Documentation

**Requirement:** All functions and major sections documented with Copilot-generated
inline comments.

**Approach:**
- Typed `/**` above every function; Copilot auto-completed `@param`, `@returns`,
  and a one-line description on Tab-accept.
- Used section banner comments (`// ─── SECTION NAME ───`) as structural anchors;
  Copilot consistently recognised and replicated the pattern for subsequent blocks.
- All JSDoc descriptions were reviewed and edited for accuracy before commit.

**Documentation coverage:**

| Area | Comment style |
|---|---|
| CSS custom properties | Block comment at `:root` |
| Data constants (`PROJECTS`, `SKILLS`) | `@type` JSDoc |
| All utility functions | Full JSDoc (`@param`, `@returns`) |
| All render functions | Full JSDoc |
| Event listeners | Inline intent comments |
| Init function | Block summary comment |

---

## 6. Prompting Strategies That Worked Well

1. **Write JSDoc first, let Copilot fill the body.** Starting with `/** @param … @returns … */`
   consistently produced more accurate and well-typed function bodies than prompting
   from inline comments alone.

2. **Name variables descriptively.** Naming a variable `formattedPhoneNumber` vs.
   `fp` led Copilot to generate clearer, self-documenting logic.

3. **Partial scaffolding.** Writing the first 2–3 lines of a function body (e.g.,
   `const digits = raw.replace(/\D/g, '')`) gave Copilot enough context to complete
   the rest accurately.

4. **Iterative refinement.** Accepting a suggestion and immediately adding a comment
   like `// also handle the optional phone field` triggered contextually aware
   follow-up completions.

---

## 7. Areas Requiring Manual Correction

| Issue | Action taken |
|---|---|
| Missing `aria-label` on icon-only links | Added manually |
| `innerHTML +=` in loop (performance) | Replaced with single assignment |
| Initial caret position drift after phone format | Custom restoration logic added |
| Copilot used `var` in one generated snippet | Replaced with `const` / `let` |
| Overly verbose regex with named capture groups | Simplified to minimal working form |

---

## 8. Deployment

| Step | Tool |
|---|---|
| Source control | GitHub (`main` branch, no sensitive data) |
| Deployment | GitHub Pages (static, single HTML file) |
| Custom domain (optional) | Configured via CNAME record |

**Deploy command:**
```bash
# Push to GitHub Pages branch
git subtree push --prefix . origin gh-pages
```

Or use the GitHub Actions workflow below:

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: .
```

---

## 9. Reflection

GitHub Copilot accelerated development by approximately **40–50%** on routine
tasks (boilerplate HTML/CSS, regex helpers, event handler scaffolding) and
proved most valuable when the developer supplied precise context through JSDoc
and descriptive naming.

Critical thinking remained essential: every Copilot suggestion was reviewed
for correctness, accessibility, and performance before acceptance.  The tool
is best understood as an intelligent autocomplete that amplifies developer
intent — not a replacement for architectural judgment or code review.
