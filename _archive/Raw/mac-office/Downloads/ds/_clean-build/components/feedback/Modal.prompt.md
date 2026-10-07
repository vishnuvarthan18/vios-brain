Use when the page must not continue until something is decided.

```jsx
<Modal open={open} onClose={close} title="Let's talk about your idea"
  footer={<><Button variant="ghost" onClick={close}>Cancel</Button><Button>Send message</Button></>}>
  <p>We reply within two working days.</p>
</Modal>
```

Do not use a modal for anything that could be a page — modals cannot be linked to, bookmarked or found again, and they are hostile on a small screen. Always leave a visible close control; Escape alone is not discoverable.
