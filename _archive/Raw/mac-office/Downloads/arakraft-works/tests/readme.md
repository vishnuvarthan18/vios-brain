# TESTS

End-to-end tests run with Playwright against the production preview build, in
desktop Chrome and iPhone WebKit. They cover section order, prices, the photo
frame and CNC carousels, WhatsApp links and analytics payloads, accessibility
(axe) and SEO metadata. Run them with `make test`. The Playwright config is
`src/playwright.config.ts`, and `npm test` sets `NODE_PATH` so these tests
resolve packages from `src/node_modules`.
