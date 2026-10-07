Use for something that *happens* — submit, open, save, apply; pass `href` and it becomes a real link instead.

```jsx
<Button>Send message</Button>
<Button variant="outline" href="/services">See the services</Button>
<Button variant="ghost" size="sm">Cancel</Button>
<Button iconOnly aria-label="Close"><Icon name="close" /></Button>
```

One primary button per view; everything else is `outline` or `ghost`. `danger` is for destructive and irreversible, never for cancel. `loading` sets `aria-busy` so the label stays and the width does not jump. Buttons deliberately carry no dotted edge.
