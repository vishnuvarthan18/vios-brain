```jsx
{loading ? <SkeletonTable rows={6} columns={4} /> : <Table columns={cols} rows={rows} />}
{loading ? <SkeletonStack lines={3} /> : <p>{text}</p>}
```

Match the skeleton to the real shape — a three-line stack where three lines are coming. A skeleton that looks nothing like the result is a second layout the visitor has to read twice. Individual bars are `aria-hidden`; the container carries `aria-busy` and an off-screen "Loading", so the state is announced once rather than as eighteen empty boxes. The sweep stops for anyone who asked for reduced motion.
