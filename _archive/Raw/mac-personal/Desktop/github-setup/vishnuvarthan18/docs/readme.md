# DOCS

Notes for maintaining the profile page. Not a project — this repo ships one
markdown file that GitHub renders somewhere else.

## How it renders

GitHub shows `README.md` from a repo whose **name matches the username exactly**
on that user's profile page. Two conditions, both silent when broken:

- The repo must be named `vishnuvarthan18`. Rename it and the profile page
  reverts to the default view with no warning.
- The repo must be **public**. A private profile repo renders nothing.

There is no build step and no preview. What is committed to `main` is what
appears, within a minute or so.

## Rules for the page itself

- **No stat widgets, streak counters or icon walls.** They are served by
  third-party hosts, so the page breaks when one goes down or rate-limits, and
  they say nothing a reader came for.
- **No claim that has to be maintained by hand.** A "currently working on"
  section is accurate for a fortnight and misleading for a year. Anything with a
  date or a count in it will be wrong before you next look at this file.
- **Images, if any, are PNG and committed here** under `assets/`, referenced by
  relative path. GitHub serves raw `.svg` as `text/plain`, so an SVG in an
  `<img>` tag does not render.
- **Links must resolve.** A dead link on the first page anyone sees is worse
  than no link. Check them when a repo is renamed or made private.

## Conventions

This repo ships documentation rather than a product, so the project scaffold
does not apply — no `src/`, no `Makefile`, no `VERSION`. What does apply is
everything in
[conventions](https://github.com/vishnuvarthan18/conventions): commit format,
licensing, README casing, and the rule against restating a rule that lives
somewhere else.
