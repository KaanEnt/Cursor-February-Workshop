---
description: Apply when generating or editing user-facing text content in frontend code — headings, subheadings, hero copy, CTAs, button labels, descriptions, benefit statements, taglines, placeholder text, or any copy that a website visitor would read. Do NOT apply for purely structural/logic changes, styling, or refactors that don't touch visible text.
globs: ["**/*.tsx", "**/*.jsx", "**/*.html", "**/*.vue", "**/*.svelte", "**/*.astro"]
alwaysApply: false
---

# Copy Writing Hook

When generating or editing user-facing text in frontend code, you MUST apply the copy-writer skill before writing any copy.

## Activation

This rule applies whenever you are:
- Writing new page content, section copy, headings, subheadings, or CTAs
- Editing existing user-facing text strings in components
- Creating landing pages, hero sections, feature sections, or benefit blocks
- Writing button labels, form labels, error messages, empty states, or onboarding text
- Generating placeholder or sample content that will be visible to users

This rule does NOT apply to:
- Code comments, variable names, or internal documentation
- Pure layout/styling changes with no text modifications
- Logic refactors that don't change visible text

## Process

### 1. Read the skill
Read `.agents/skills/copy-writer/SKILL.md` to load the full punchy copy methodology (VBF framework, BBQ test, SMIT, jargon rules, word swaps). If you need deeper guidance on a specific technique, read the relevant file from `.agents/skills/copy-writer/references/`.

### 2. Gather page context
Before writing any copy, read the full file (and any related layout/page files) to understand:
- What the page is about and who it's for
- The existing tone, voice, and messaging structure
- How the new/edited copy fits into the overall page narrative

### 3. Write copy using the methodology
- Lead with **value**, then **benefits**, then **features** (VBF order)
- Keep sentences under 20 words
- Start with active verbs — no preambles like "Our platform enables..."
- Use the word swap list (leverage → use, identify → find, enable → let/help, etc.)
- Make it pass the **BBQ test**: would a customer say this to a friend?
- Ensure each block has a clear **SMIT** (Single Most Important Takeaway)
- Champion the customer, not the product

### 4. Embed directly in code
Write the final punchy copy directly into the JSX/HTML. Do not output a separate copy document — put it right in the component where it belongs. If editing existing copy, replace in-place.
