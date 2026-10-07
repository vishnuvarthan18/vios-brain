```jsx
<Slider id="lead" label="Lead time" value={days} onChange={setDays} min={1} max={90}
  format={n => n + ' days'} />
```

Use where the position matters more than the digits — a threshold, a tolerance, a range filter. **Always show the value**: a slider with no readout is a guess. If the exact number is what the visitor cares about, use `NumberInput` instead, or pair the two.
