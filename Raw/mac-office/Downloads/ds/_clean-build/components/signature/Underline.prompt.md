```jsx
<Underline href="/services">See the services</Underline>
<Underline persistent>Apply</Underline>
```

It wipes in rather than fading, and it is transform-only so it never forces layout. Use `persistent` inside a card footer, where the link is already obviously a link.
