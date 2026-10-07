Use for a repeating thing in a list. A lone card is a box round nothing — use a section with a heading instead.

```jsx
<CardGrid>
  <Card kind="course" href="/apply">
    <CardMedia />
    <CardBody>
      <span className="ac-card__eyebrow">Six weeks · Tue &amp; Thu</span>
      <h3 className="ac-card__title">Software testing</h3>
      <p className="ac-card__text">From your first assertion to a suite that catches real regressions.</p>
    </CardBody>
    <CardFooter><span className="ac-card__price">EUR 400</span></CardFooter>
  </Card>
</CardGrid>
```

ACDS's `padding` still works, but `padSm` is the system's own answer; a raw length that is not on the spacing scale is passed straight through and will drift.

Make the whole card the link, not a "read more" inside it. `alt=""` on the card image when the title already says what it is. A card at rest is flat; the dotted edge separates it from the page, and on hover the edge closes and the card lifts.
