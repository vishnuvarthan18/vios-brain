```jsx
<Toolbar title="Jobs" count="248 rows" actions={<Button>New job</Button>}>
  <Search placeholder="Search jobs" />
</Toolbar>
<FilterBar><Tag onRemove={drop}>Engineering</Tag><Tag onRemove={drop}>Open</Tag></FilterBar>
<BulkBar count={3} noun="job" onClear={clear} actions={<Button size="sm" variant="ghost">Assign</Button>} />
```

`BulkBar` replaces the toolbar once rows are picked rather than floating over the page — a floating bar covers the rows the visitor is trying to check. It carries `role="status"` so the count is announced as it changes.
