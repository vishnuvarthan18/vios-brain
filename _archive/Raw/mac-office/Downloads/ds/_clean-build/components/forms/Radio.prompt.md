The ACDS name for `Choice type="radio"`, kept so existing imports keep working. New code writes `Choice`.

```jsx
<ChoiceGroup>
  <Radio name="plan" label="Monthly" defaultChecked />
  <Radio name="plan" label="Annual" />
</ChoiceGroup>
```

Group them with a shared `name`. Radios are square here rather than round — they stay distinguishable from a checkbox by the filled square inside, versus a tick.
