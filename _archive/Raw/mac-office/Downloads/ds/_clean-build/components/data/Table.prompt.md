Use for data where rows and columns both mean something. Never for layout — that is a grid.

```jsx
<Table caption="Group companies by country" sortable hover
  columns={[{ key: 'company', label: 'Company' }, { key: 'country', label: 'Country' }, { key: 'since', label: 'Since', numeric: true }]}
  rows={[{ company: 'araCreate GmbH', country: 'Germany', since: '2003' }]} />
```

Numeric columns get `numeric: true`, which right-aligns them in the mono face so digits line up. A text sort puts 100 before 20 — this one does not. On a phone it scrolls rather than shrinking the type.
