```jsx
<Banner tone="accent" title="Read-only until 18:00 CET" action={<Button size="sm" variant="inverse">What changed</Button>}>
  Scheduled maintenance. Nothing you have saved is affected.
</Banner>
```

One banner at a time, at the very top of the shell. `tone="error"` uses `role="alert"` and interrupts a screen reader — reserve it for something blocking. If the visitor can dismiss it and lose nothing, give it `onClose`; if they cannot, do not offer one.
