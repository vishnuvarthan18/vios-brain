Wrap every form control.

```jsx
<Field label="Email" htmlFor="email" required hint="We will not add you to anything.">
  <Input id="email" type="email" aria-describedby="email-hint" />
</Field>
<Field label="Phone" htmlFor="tel" error="That number is too short to be valid.">
  <Input id="tel" invalid />
</Field>
```

Never label a field with a placeholder alone — it disappears the moment someone types. Hints go under the field, not in a tooltip. One idea per field.
