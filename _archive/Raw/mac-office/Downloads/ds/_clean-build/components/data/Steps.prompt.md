```jsx
<Steps items={[
  { title: 'Apply', text: 'Two minutes, and no payment.', state: 'done' },
  { title: 'A short call', text: 'Fifteen minutes.', state: 'current' },
  { title: 'Enrol' }
]} />
```

Completed steps get a tick as well as the gold fill — colour is never the only signal. The current step carries `aria-current="step"`. For alternative views rather than a sequence, use `Tabs`.
