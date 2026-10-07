```jsx
<Field label="Places on this cohort" htmlFor="places" hint="Sixteen is the room's limit.">
  <NumberInput id="places" value={places} onChange={setPlaces} min={1} max={16} />
</Field>
```

Only where nudging by one is a real action — quantities, small counts. For a wide range where the exact number matters less than the position, use `Slider`; for a number people type in full, a plain `Input type="number"` is less furniture. The value is set in the mono face so digits do not shift width as they change. Steppers disable at the bounds rather than silently clamping.
