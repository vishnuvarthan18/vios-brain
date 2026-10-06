```jsx
<DatePicker value={start} onChange={setStart} min={new Date()}
  footer={<><Button size="sm" variant="ghost" onClick={clear}>Clear</Button><Button size="sm" onClick={pickToday}>Today</Button></>} />
```

**Always offer a typed alternative.** A calendar is slow for a date someone already knows — pair it with an `Input` and let people type. For a date of birth, typing wins outright; nobody pages back thirty years.

Today is an **outline**, a selected day is a **gold fill**. A filled "today" is indistinguishable from a selection, which is the commonest date-picker bug there is. Every day carries a full `aria-label` because "14" on its own tells a screen-reader user nothing.
