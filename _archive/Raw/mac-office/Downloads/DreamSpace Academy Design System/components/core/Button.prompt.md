Pill-shaped call-to-action button; use `primary` (solid purple) for the main action on a screen and `accent` (orange) sparingly for a single standout CTA.

```jsx
<Button variant="primary" size="md">Apply now</Button>
<Button variant="accent">Support a lab</Button>
<Button variant="secondary">Learn more</Button>
<Button variant="ghost">Skip</Button>
```

Variants: `primary`, `secondary` (outline), `accent` (orange), `ghost` (text-only). Sizes: `sm`, `md`, `lg`. `disabled` greys it out. Never use both `primary` and `accent` for competing actions in the same view — accent is reserved for one highlighted action.
