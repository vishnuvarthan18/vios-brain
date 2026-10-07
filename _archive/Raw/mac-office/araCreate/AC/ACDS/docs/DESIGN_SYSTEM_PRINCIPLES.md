# Design System — Principles & Checklist

Reference doc. Read before building anything. Design-first: visual system before code system.

## For future agent
This is the rulebook for the ACDS design system. Built design-first — visual/brand layer comes before component code. Source of truth will be an existing live website (template-based), not built from zero. Read this + STATE.md before doing design system work.

---

## 1. What a design system actually is

Not a style guide. Not a component library. Not a Figma file.

A design system = **one shared language** across three layers:

1. **Foundations** — the raw decisions (color, type, space, motion, etc.)
2. **Components** — foundations assembled into reusable UI pieces
3. **Patterns** — components assembled into real flows (forms, nav, checkout, onboarding)

If you only build layer 2 (components) without layer 1 (foundations) being explicit and named, it's not a system — it's a kit. Kits drift. Systems don't.

This layering descends from Brad Frost's Atomic Design (Atoms→Molecules→Organisms→Templates→Pages). [IBM Carbon](https://carbondesignsystem.com/guidelines/2x-grid/overview/) names all three tiers explicitly in its own nav (Foundations → Components → Patterns) — the cleanest real-world match. See also [Material Design 3 Foundations](https://m3.material.io/foundations) and [Atlassian Design System](https://atlassian.design/foundations).
→ [Atomic Design, Brad Frost](https://atomicdesign.bradfrost.com/chapter-2/)

---

## 2. The non-negotiable core (every DS needs these)

### Foundations (Design Tokens)
- **Color** — primitive palette (raw hex/oklch) → semantic roles (`bg.default`, `text.danger`, `border.focus`) → component-level tokens. Never reference raw hex in a component.
- **Typography** — type scale (sizes), font weights, line-heights, letter-spacing, font families. Named by role (`heading.lg`, `body.sm`), not by pixel value.
- **Spacing** — one spacing scale (4px or 8px base grid). Every margin/padding in the product pulls from this scale. No arbitrary values.
- **Sizing** — icon sizes, avatar sizes, container widths, breakpoints.
- **Radius** — corner radius scale.
- **Elevation/Shadow** — shadow scale tied to z-index/layering logic.
- **Motion** — duration + easing tokens. Consistent animation feel is as much a brand signal as color.
- **Iconography** — one icon set, one stroke weight, one grid size.
- **Z-index scale** — named layers (dropdown, modal, toast, tooltip) so stacking never gets guessed.

### Components
- Each component has: states (default/hover/active/focus/disabled/loading/error), variants (size, emphasis), and documented props/anatomy.
- Accessibility is part of the component, not a separate audit: focus rings, contrast ratios (WCAG AA minimum), keyboard nav, ARIA roles.
  → [WCAG 2.1 contrast minimum (4.5:1 normal text, 3:1 large text)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html) · [GOV.UK Design System — "we aim to meet level AA for styles, components and patterns"](https://design-system.service.gov.uk/accessibility/accessibility-strategy/)
- Responsive behavior defined per component, not per page.

### Patterns
- Common flows documented as compositions of components (e.g. "form validation pattern," "empty state pattern," "loading state pattern").
- Content/voice guidelines travel with patterns — microcopy is part of the system, not an afterthought.

### Documentation layer
- Principles (the "why" — design values, brand personality, what we optimize for)
- Usage guidelines (do/don't per component)
- Changelog / versioning (DS is a product, it ships versions)
- Contribution rules (who can propose/add a component, review process)

"A design system isn't a project. It's a product, serving products" — Nathan Curtis, 2016, the phrase that coined this framing. See [IBM Carbon's contribution model](https://carbondesignsystem.com/getting-started/contributing/overview/) for a real, active governance process (incubation space, contribution paths, component checklist, office hours).

---

## 3. Best practice for building a DS from an existing live site (your actual situation)

You already have a live site. Don't invent a new system in a vacuum — **extract and systemize what's already live**, then fix inconsistencies deliberately.

Order of operations:

1. **Audit first.** Screenshot every page/state of the live site. Catalog every color, font size, spacing value, button style, card style actually in use. You will find more variation than you expect — that's normal, that's the problem the DS fixes.
2. **Find the patterns already there.** Most templates have 70% of a system already, just unnamed and inconsistent. Group near-duplicates (e.g. 6 shades of blue that should be 1).
3. **Name and formalize foundations** from what you audited — don't redesign colors/type from scratch unless there's a reason. Systemizing ≠ redesigning.
4. **Only then build/rebuild components**, using the new tokens.
5. **Reconcile drift** — anywhere the live site breaks the new system, decide: fix the site, or make an documented exception.
6. **Gap-fill** — states/components the live site never needed yet (empty states, error states, dark mode) but the product will need.

This order matters: token-first prevents you from re-baking the same inconsistencies into new components.

---

## 4. What's different in the AI era (2025–2026)

Design systems are now consumed by two audiences: humans (designers/devs) and AI agents (Claude, Cursor, Figma MCP, v0, etc.) generating UI on your behalf. That changes what "good" looks like. This is an active, fast-moving conversation (every source below is dated Sept 2025–June 2026) — not yet settled industry practice, but the direction every major vendor is building toward.

- **Tokens must be machine-readable, not just documented.** W3C Design Tokens format (JSON) or Style Dictionary — not a PDF or a Notion page. An AI agent should be able to read your tokens as structured data and use them correctly without guessing.
  → [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/) · [Design Tokens Format Module 2025.10 spec](https://www.designtokens.org/TR/drafts/format/) · [Style Dictionary](https://styledictionary.com/)
- **Semantic naming over describable naming.** `color.danger.text` not `color.red-600`. Figma's own guidance states semantic tokens (e.g. `color-primary-500`) outperform raw primitives for AI reliability.
  → [Figma — LLM Context Design](https://www.figma.com/resource-library/llm-context-design/)
- **Components need machine-discoverable specs.** Props, variants, and valid states as structured data (TypeScript types, Figma Code Connect, component manifests) — not only prose. Figma Code Connect exists specifically to "enhance the Figma MCP server's ability to guide AI agents with more precise implementation details."
  → [Figma Code Connect docs](https://developers.figma.com/docs/code-connect) · [Introducing Code Connect, Figma blog, Apr 2024](https://www.figma.com/blog/introducing-code-connect/)
- **One source of truth, read directly by AI tooling.** The Figma Dev Mode MCP Server (June 2025) surfaces variables/tokens/styles directly to AI agents — Copilot, Cursor, Windsurf, Claude Code are named explicitly. This folds "sync design and code" and "AI reads your tokens" into one mechanism rather than two separate problems.
  → [Introducing the Figma MCP Server, Jun 2025](https://www.figma.com/blog/introducing-figma-mcp-server/) · [Chromatic/Storybook MCP servers, Mar 2026](https://www.chromatic.com/blog/introducing-published-storybook-mcp-servers/) (confirms this is a multi-vendor trend, not Figma-only)
- **Constrain, don't just document.** Lint rules / CI checks that reject hardcoded hex values or arbitrary spacing in code review. AI agents (and humans) take the path of least resistance — make the correct token the easiest option, enforced automatically.
  → [Smashing Magazine — How To Make Your Design System AI-Ready, Jun 2026](https://www.smashingmagazine.com/2026/06/how-make-design-system-ai-ready/) · [Supernova — AI-Ready Design Systems, Sept 2026](https://www.supernova.io/blog/ai-ready-design-systems-preparing-your-design-system-for-machine-powered-product-development)
- **Accessibility and content rules as explicit, checkable rules** — not prose — because both AI and automated QA need to verify against something concrete (contrast ratio thresholds, required ARIA attributes).
- **Governance itself is shifting toward AI consumption.** Figma frames design systems as becoming "carriers of craft" that must be built for AI as a first-class consumer, with governance increasingly embedded in AI tooling rather than only in docs.
  → [Figma — 5 Shifts Redefining Design Systems in the AI Era, Nov 2025](https://www.figma.com/blog/5-shifts-redefining-design-systems-in-the-ai-era/)

Bottom line: a 2026 design system is trending toward a **structured, machine-readable product** (tokens as data, components as typed specs) with a human-readable layer on top. Treat this as the direction, not yet a finished standard.

---

## 5. Minimum viable DS structure (practical folder shape)

```
design-system/
├── tokens/              # source of truth, JSON (W3C token format)
│   ├── color.json
│   ├── typography.json
│   ├── spacing.json
│   ├── radius.json
│   ├── shadow.json
│   └── motion.json
├── components/          # per component: anatomy, states, variants, code
├── patterns/            # composed flows
├── docs/
│   ├── principles.md
│   ├── accessibility.md
│   └── contribution.md
└── CHANGELOG.md
```

---

## 6. Next step for this project

1. Audit the live site (screenshots + inventory of actual colors/type/spacing in use).
2. Extract foundations from that audit → propose a token set.
3. Review proposed tokens with you before touching components.
4. Then look at what's already built, map it against the new system, flag drift.

---

## Sources

**Structure / layering**
- [Atomic Design, Brad Frost](https://atomicdesign.bradfrost.com/chapter-2/)
- [IBM Carbon Design System — Foundations/Components/Patterns](https://carbondesignsystem.com/guidelines/2x-grid/overview/)
- [Material Design 3 Foundations](https://m3.material.io/foundations)
- [Atlassian Design System](https://atlassian.design/foundations)

**Design tokens standard**
- [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/)
- [Design Tokens Format Module 2025.10 (spec)](https://www.designtokens.org/TR/drafts/format/)
- [Style Dictionary](https://styledictionary.com/)

**"Design system as a product"**
- "A Design System Isn't a Project. It's a Product, Serving Products," Nathan Curtis, 2016 — [Medium mirror](https://medium.com/eightshapes-llc/a-design-system-isn-t-a-project-it-s-a-product-serving-products-74dcfffef935) (primary eightshapes.com domain was unreachable at research time — verify directly before citing as primary)
- [IBM Carbon — Contributing](https://carbondesignsystem.com/getting-started/contributing/overview/)

**AI-era design system practices (2025–2026, active/evolving — not yet settled standard)**
- [Figma Code Connect docs](https://developers.figma.com/docs/code-connect)
- [Introducing Code Connect, Figma, Apr 2024](https://www.figma.com/blog/introducing-code-connect/)
- [Introducing the Figma Dev Mode MCP Server, Jun 2025](https://www.figma.com/blog/introducing-figma-mcp-server/)
- [Figma — LLM Context Design](https://www.figma.com/resource-library/llm-context-design/)
- [Figma — 5 Shifts Redefining Design Systems in the AI Era, Nov 2025](https://www.figma.com/blog/5-shifts-redefining-design-systems-in-the-ai-era/)
- [Smashing Magazine — How To Make Your Design System AI-Ready, Jun 2026](https://www.smashingmagazine.com/2026/06/how-make-design-system-ai-ready/)
- [Supernova — AI-Ready Design Systems, Sept 2026](https://www.supernova.io/blog/ai-ready-design-systems-preparing-your-design-system-for-machine-powered-product-development)
- [Chromatic/Storybook — Published Storybook MCP Servers, Mar 2026](https://www.chromatic.com/blog/introducing-published-storybook-mcp-servers/)

**Accessibility baseline**
- [WCAG 2.1 — Contrast Minimum (SC 1.4.3)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html)
- [GOV.UK Design System — Accessibility strategy](https://design-system.service.gov.uk/accessibility/accessibility-strategy/)
- [USWDS — Accessibility](https://designsystem.digital.gov/documentation/accessibility/)

Nothing gets built until the audit is done and tokens are agreed.
