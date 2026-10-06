```jsx
<SegmentedControl name="range" value={range} onChange={setRange} options={['30d', '90d', '1y']} />
```

Use it when the options are short and seeing all of them is the point — a date range, a view switch. **Four is the ceiling**; past that it is a `Select`. Not for more than one choice at a time (that is `ChoiceGroup`), and not for actions (that is `Button`). The selected segment is gold with brand-black text at 9.51:1.
