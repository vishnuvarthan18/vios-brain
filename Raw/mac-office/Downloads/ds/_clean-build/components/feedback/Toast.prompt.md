```jsx
<ToastRegion>
  <Toast tone="success" title="Message sent" onClose={dismiss}>We reply within two working days.</Toast>
</ToastRegion>
```

**Never put anything necessary in a toast** — it disappears. If the visitor must read it, it is an `Alert` or a `Modal`. The region is `aria-live="polite"` deliberately.
