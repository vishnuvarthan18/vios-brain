```jsx
<Field label="Client" htmlFor="client">
  <Combobox id="client" options={clients} value={client} onChange={setClient}
    placeholder="Start typing a name" emptyText="No client by that name. Add one first." />
</Field>
```

Use it once a `Select` becomes unusable — roughly twenty options and up. Below that a select is better: it gets the phone's native picker and autofill for free. The matched substring is marked by **weight, not colour** — a gold highlight measures 1.67:1 and a coloured one fights the selected state.
