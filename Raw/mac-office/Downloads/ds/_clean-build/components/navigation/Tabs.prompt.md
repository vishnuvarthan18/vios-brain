Use for alternative views of the same kind of thing, where nobody needs two at once.

```jsx
<Tabs a11yLabel="Verticals" tabs={[
  { id: 'eng', label: 'Engineering', content: <p>From prototype to production.</p> },
  { id: 'man', label: 'Manufacturing', content: <p>From drawing to delivery.</p> },
  { id: 'med', label: 'Media', content: <p>From sketch to screen.</p> }
]} />
```

Not for sequential steps — that is `Steps`. Do not use tabs to hide bulk on a long page; people do not find tabbed content they were not looking for.
