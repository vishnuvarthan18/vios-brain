# PUBLIC

Static assets served at the site root.

Empty as of M0. The widget bundle is **not** served from here in production —
it goes to object storage behind a CDN at a versioned path, because Webflow's
Assets panel rejects `.js`. See [`docs/build-plan.md`](../../../docs/build-plan.md) §2.
