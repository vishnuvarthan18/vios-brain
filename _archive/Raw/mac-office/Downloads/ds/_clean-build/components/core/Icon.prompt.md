Small interface glyph — arrow, check, close, chevron, clock, mail, pin, cog, alert, search. Inherits `currentColor`.

```jsx
<Icon name="arrow-right" />                     {/* decorative */}
<Icon name="alert" a11yLabel="Warning" size="lg" /> {/* content */}
```

Every icon is decoration or content and must declare which — omitting `label` marks it `aria-hidden`. Never set its colour; set the text colour of what contains it. For the brand's **isometric service icons and illustrations**, place the SVG files from `assets/icons/` and `assets/illustrations/` directly instead. Never an emoji, never an icon font.
