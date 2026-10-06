The ACDS name for `Stat`, kept so existing imports keep working. New code writes `Stat` inside a `StatRow`.

```jsx
<StatRow>
  <StatBlock value="300+" label="Clients empowered" />
  <StatBlock value="8+" label="Group companies" />
  <StatBlock value="20+" label="Years of service" accent />
</StatRow>
```

The `+` is not decoration — it is what makes the figure true, and every number comes from the brand facts. `accent` no longer paints the figure Golden Sun: gold is never text in this system, so it darkens to the heading colour.
