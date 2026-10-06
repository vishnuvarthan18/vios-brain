A token the visitor can take off — a filter, a chosen option.

```jsx
<TagGroup>
  <Tag onRemove={drop} removeLabel="Remove Engineering">Engineering</Tag>
  <Tag>Berlin</Tag>
</TagGroup>
```

Without `onRemove` it is a plain token. Tags are the only pill-shaped thing in the system, because the source styles them that way.
