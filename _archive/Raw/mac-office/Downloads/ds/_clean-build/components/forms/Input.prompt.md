```jsx
<Input id="name" type="text" placeholder="Ada Lovelace" />
<Input size="sm" invalid />
<Search placeholder="Search courses" />
```

Always inside a `Field` with a real label — pass `label`, `hint` or `error` straight to `Input`, as ACDS did, and it wraps itself in one for you, but that is the compatibility path, not the pattern. Focus switches the dotted edge to solid so a dotted border and a dotted focus ring can never be confused.
