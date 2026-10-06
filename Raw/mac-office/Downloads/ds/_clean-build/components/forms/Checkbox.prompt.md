The ACDS name for `Choice type="checkbox"`, kept so existing imports keep working. New code writes `Choice`.

```jsx
<Checkbox label="Subscribe to the newsletter" defaultChecked />
<Checkbox label="Send me the recordings" hint="Only for the course you take." />
<Checkbox label="Disabled" disabled />
```

Square, like everything else here. A checkbox takes effect when the form is submitted; a `Switch` takes effect immediately. Several of them belong in a `ChoiceGroup`.
