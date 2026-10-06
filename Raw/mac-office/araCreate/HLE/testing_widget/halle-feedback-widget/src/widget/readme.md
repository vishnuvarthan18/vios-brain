# WIDGET

The embeddable script. Vanilla JS, **zero dependencies**, bundled by esbuild to
a single file and served from a CDN at a versioned path.

It runs on somebody else's live website, so the rules are stricter than
anywhere else in the repo:

- No dependencies, ever. Ask before adding one.
- Under **15 KB gzipped**. `make size` is a build gate, not a suggestion.
- No client-specific values in the source — it carries a public key and fetches
  its own configuration.
- Nothing written to the visitor's device. No `localStorage`, no
  `sessionStorage`, no cookies.
- Renders inside a Shadow DOM. No CSS crosses that boundary in either direction.
- It must never break the host site. Every entry point in `try/catch`, failures
  silent.

Run `make dev-widget` to rebuild on change, and `make size` to check the
budget. The bundle is written to `dist/v1.js`.

The workspace declares **no dependencies**, and that is enforced by its
`package.json` having no `dependencies` block at all. esbuild is a build tool
in the repo root, not a dependency of the widget.

Full rules in [`docs/agent-rules.md`](../../docs/agent-rules.md).

Depth is in [`docs/widget/`](../../docs/widget/).
