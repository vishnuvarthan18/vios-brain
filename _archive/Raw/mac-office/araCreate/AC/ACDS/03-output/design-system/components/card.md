# Card

Two real card types found on the live site: **blog card** (`.cms-blog-item`) and **project card** (`.cms-projects-items` + `.project-content-wrapper` + `.project-image-wrapper`). Verified directly against the original minified CSS (not the pretty-printed intermediate file — see Data Quality Note).

## For future agent
These weren't caught by the first automated component pass (filtered out by a `cms-` prefix the extraction script didn't expect). Found by checking real page HTML (`blogs/index.html`, `projects/index.html`) for repeating structural classes, then pulling their CSS directly from the original minified source.

## Data quality note

The audit pipeline's pretty-printed CSS file (`02-process/audit/live-site-pretty.css`) has a bug: it drops the first 1-2 characters of CSS property names (e.g. `border-style` → `order-style`, `width` → `idth`, `padding` → `adding`). This was caught while extracting the card component and fixed by re-reading from the original minified source file directly. Anything previously extracted FROM the pretty-printed file (not the original) should be treated as unverified until re-checked against the original minified CSS. The `button.md` component was pulled by a subagent from the minified source correctly and is not affected, but this hasn't been independently re-confirmed.

## Blog card — `.cms-blog-item`

```css
.cms-blog-item {
  border-style: dashed dashed solid solid;
  border-width: 1px;
  border-color: var(--brand-color-gray);
  width: auto;
  max-width: 100%;
  height: auto;
  padding: 20px;      /* merged from two rules: padding-bottom:0 + padding:20px */
  margin-bottom: 0;
}

.cms-blog-item.example-details,
.cms-blog-item.details {
  background-color: var(--brand-color-canvas);
}
```

Unusual, likely-deliberate detail: border style is `dashed dashed solid solid` — top and right sides dashed, bottom and left sides solid. Not a typical card border; worth confirming visually it's intentional before treating as a template default.

## Project card — three-part structure

```css
.cms-projects-items {
  border: 1px dashed #000;
  border-style: dashed dashed solid solid;   /* same mixed pattern as blog card */
  display: flex;
  flex-flow: column;
  width: auto;
  max-width: 100%;
  height: auto;
  margin-bottom: 20px;   /* one instance overrides to 10px */
  padding: 0;
}

.project-image-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;           /* image itself: object-fit: cover; height: 100% */
  margin: 0;
  padding: 0;
}

.project-content-wrapper {
  padding: 15px 15px 0;
}

.project-content {
  display: flex;
  flex-flow: column;
  gap: 4px;   /* grid-column-gap + grid-row-gap, both 4px */
}
```

## Pattern found

Both card types share the same mixed dashed/solid border treatment (`dashed dashed solid solid`) — this is consistent enough across two independent card types to likely be a deliberate stylistic choice (not drift), but hasn't been visually confirmed.

## Open questions

1. Confirm the mixed dashed/solid border is an intentional design choice, not a copy-paste artifact from the template.
2. `.cms-blog-item` and `.cms-projects-items` margin-bottom has two conflicting values (20px vs 10px) across instances — same drift pattern seen in the button component. Decide canonical value.
3. Re-verify all components already extracted via the pretty-printed CSS file against the original minified source, per the Data Quality Note above.
