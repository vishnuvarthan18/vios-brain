Underlined tab bar with a Golden Sun active indicator.

```jsx
<Tabs tabs={[
  { id: "eng", label: "Engineering", content: <p>…</p> },
  { id: "mfg", label: "Manufacturing", content: <p>…</p> },
  { id: "media", label: "Media", content: <p>…</p> },
]} />
```

Controlled via `value` + `onChange`, or uncontrolled with `defaultValue`.
