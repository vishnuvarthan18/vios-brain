```jsx
<Pagination page={3} pages={12} onChange={setPage} />
<Pagination page={3} pages={12} hrefFor={p => '/journal/page/' + p} />
```

Prefer `hrefFor` on a content site — real links can be opened in a new tab and shared. The current page is marked `aria-current="page"`.
