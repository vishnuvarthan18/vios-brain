```jsx
<ChartShell title="Jobs closed" series={['This year', 'Last year']}
  axis={['Jan', 'Apr', 'Jul', 'Oct']}
  actions={<SegmentedControl name="range" options={['30d', '90d', '1y']} />}>
  <Bars values={[4, 9, 6, 11, 8, 13]} muted={[0, 1]} />
</ChartShell>
```

This is a frame, not a charting engine — for anything beyond bars, put a real chart inside `ChartShell` and let it supply the marks while the shell supplies the title, legend, gridlines and axis. **Gold is series one**, because the brand's single accent should carry the number that matters; everything after it is neutral. Always label the axis: a chart with no units is decoration.
