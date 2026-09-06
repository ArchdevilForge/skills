---
name: design
description: Design or redesign frontend pages and product UI. Use for layouts, visual systems, dashboards and interaction design; not copy-only edits.
---

# Frontend design

Match the user's audience, brand, references and existing stack. For a redesign, preserve working routes, content, analytics and accessibility unless the requested change includes them. Ask only when preserve vs overhaul is genuinely unclear.

## Route before reading

| Task | Read only what applies |
|---|---|
| Landing page, portfolio, visual refresh | [Visual guidance](references/ai-tells.md) |
| Dashboard, admin, data table, product UI | [UI contract](references/ui-contract.md); [UX patterns](references/ux-system.md) for interaction changes |
| Existing-site redesign or named visual pattern | [Vocabulary and redesign](references/vocabulary.md) |
| Motion or scroll choreography requested | [Motion examples](references/motion.md); optional [dials](references/dials.md) to describe intensity |
| Concrete palette, typography or chart recommendations | [Design database](references/uiux-db.md) |
| A particular design system is required | Its section in [official sources](references/appendix.md); verify current API before installation |
| User asks to maintain a reusable block collection | [Block guidance](references/block-library.md) |

## Implement

- Reuse the closest existing page, components and tokens. Use native HTML/CSS before adding libraries; keep the project's framework and package manager.
- Let hierarchy and content choose the layout. Avoid repeating a generic template, but do not replace clear tables, lists or familiar controls merely to look different.
- Images, animation and light/dark variants are scope choices, not mandatory features. Prefer supplied assets; generate missing visuals when needed or requested. Label mock data and generated product illustrations; never invent testimonials, customers or product evidence.
- Use semantic controls, labels, keyboard access, visible focus and readable contrast. Provide the loading/empty/error states that the changed flow can actually enter. Honor reduced-motion preferences for nonessential animation.
- Keep layout responsive without clipping text or hiding essential actions. Test the themes the project supports; do not add a theme system just for this task.
- Use CSS for simple motion. For JS animation, reuse installed tools, clean up listeners/timelines, avoid frame-by-frame React state updates, and keep expensive assets off the critical loading path.

## Finish

Run affected project checks and [visual QA](references/visual-qa.md) for rendered UI changes. Fix failures caused by the change; report unavailable checks separately from implementation status. A code-only review does not prove visual acceptance.
