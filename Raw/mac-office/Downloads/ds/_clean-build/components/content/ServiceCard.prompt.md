The ACDS service/vertical tile, kept so existing imports keep working. A row-per-service reads better in most sections — reach for `ServiceList` first.

```jsx
<CardGrid>
  <ServiceCard number="01" icon="assets/icons/icon-product-design.svg" title="Engineering"
    description="from Prototype to Production — hardware and software technologies" href="/engineering" />
  <ServiceCard number="02" icon="assets/icons/icon-execution-and-production.svg" title="Manufacturing"
    description="from Drawing to Delivery — rapid prototyping through series production" href="/manufacturing" />
</CardGrid>
```

ACDS's "Learn more" is gone on purpose: the whole card is the link, so a second target inside it would be a second tab stop to the same place. The icon is decorative — `alt=""` is set for you, so the title has to say what the service is. The couplets are part of each vertical's name; do not paraphrase them.
