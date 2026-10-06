Use for a record's detail while the list stays put — reviewing rows one after another without losing your place.

```jsx
<Drawer open={!!picked} onClose={() => setPicked(null)} title={picked?.name}
  footer={<><Button variant="ghost" onClick={close}>Cancel</Button><Button>Save</Button></>}>
  <Field label="Status" htmlFor="s"><Select id="s" options={['Open', 'Closed']} /></Field>
</Drawer>
```

Same rule as `Modal`: if it could be a page, make it a page — a drawer cannot be linked to or bookmarked. Escape and focus trapping come from the browser; do not rebuild them.
