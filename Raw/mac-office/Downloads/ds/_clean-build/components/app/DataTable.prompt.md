The signed-in table. Use plain `Table` for static content; use this when rows are picked, sorted or acted on.

```jsx
<DataTable selectable sortable caption="Jobs"
  rowKey={r => r.id}
  selected={picked} onSelect={setPicked}
  columns={[
    { key: 'job', label: 'Job' },
    { key: 'client', label: 'Client' },
    { key: 'due', label: 'Due', numeric: true }
  ]}
  rows={[{ id: 1, job: <CellStack title="Bracket rev C" sub="Manufacturing" />, client: 'Client name', due: '00' }]}
  actions={r => <Button size="sm" variant="ghost" iconOnly aria-label={'Open ' + r.client}><Icon name="arrow-right" /></Button>} />
```

A selected row carries a tint **and** a gold left mark — the tint alone fails for anyone who cannot distinguish it. Row actions fade in on hover only where the pointer is fine; on touch and for keyboard they are always there, because a control that exists only on hover does not exist on a phone. Pair with `BulkBar` for what to do with a selection, and `SkeletonTable` while it loads.
