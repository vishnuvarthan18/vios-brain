The frame every signed-in araCreate screen sits in. On brand black, so the nav reads as chrome and the content area is the canvas.

```jsx
<AppShell rail={collapsed}
  nav={<>
    <AppBrand logo="assets/logos/aracreate-icon-t-w-b-y.svg" name="araCreate" />
    <NavList heading="Work" current={view} onSelect={setView} items={[
      { key: 'jobs', label: 'Jobs', icon: <Icon name="cog" />, count: 12 },
      { key: 'clients', label: 'Clients', icon: <Icon name="pin" /> }
    ]} />
    <NavFooter><NavList items={[{ key: 'settings', label: 'Settings' }]} /></NavFooter>
  </>}
  bar={<><Button variant="ghost" iconOnly aria-label="Menu"><Icon name="menu" /></Button><Search /></>}>
  <Toolbar title="Jobs" />
</AppShell>
```

The current item is marked with `aria-current` and a 3px gold rule — never covered by an invisible layer, which is the bug that killed 27 links on the old site. In the rail the labels stay in the DOM and are hidden visually, so a screen reader still reads them.
