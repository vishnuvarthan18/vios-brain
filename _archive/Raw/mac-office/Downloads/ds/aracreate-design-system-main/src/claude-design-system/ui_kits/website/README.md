# araCreate Group — Website UI Kit

A high-fidelity recreation of the araCreate Group marketing site (`website-webflow/`), rebuilt on the design-system primitives.

## Run
Open `index.html`. It loads `../../styles.css` and `../../_ds_bundle.js`, then the section files.

## Screens / sections
- **Navbar.jsx** — sticky bar + full-screen Golden Sun menu overlay (Home / About / Projects / Contact + Services list).
- **Hero.jsx** — numbered eyebrow, light display headline, dual CTA, isometric illustration.
- **TrustedBy.jsx** — client logo strip.
- **About.jsx** — golden-wash band, duotint image, stat blocks, founding story.
- **Services.jsx** — 5 service verticals via `ServiceCard`.
- **Projects.jsx** — duotint project cards with tags.
- **Contact.jsx** — interactive form (submits to a thank-you state).
- **SiteFooter.jsx** — dark footer with service / company / group columns, legal + copyright.

## Interactions
Menu opens/closes; nav switches between Home, Projects, About, Contact; the contact form submits to a success state.

## Notes
The live site's animated isometric "machine" illustrations are many-layered SVG sprites; this kit substitutes the simpler isometric illustrations shipped in `assets/illustrations/`. Real machine sprites were intentionally omitted.
