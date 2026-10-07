Use for **actions**.

```jsx
<Dropdown trigger="Share" items={[
  { label: 'Copy link', onSelect: copy },
  { separator: true },
  { label: 'Open in Instagram', href: 'https://instagram.com' }
]} />
```

Do not use it for choosing a value in a form — that is a `Select`, which works with the phone's native picker and with autofill. A menu that closes and dumps focus at the top of the page is worse than one that does not close.
