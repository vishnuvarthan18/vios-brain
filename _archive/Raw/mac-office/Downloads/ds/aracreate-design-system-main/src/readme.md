# SOURCE

The design systems themselves — one folder per platform that hosts or generates
them. Each folder is self-contained: it keeps whatever layout its own platform
expects, so it can be consumed directly by that platform without a build step in
between.

| Folder | Platform | Notes |
| --- | --- | --- |
| [`claude-design-system/`](claude-design-system/) | [Claude Design](https://claude.ai/design) | Master copy, imported by Claude Design. Kept in that platform's own layout — **do not restructure or reformat**. |

Because each platform owns the layout of its own folder, the araCreate repo
conventions (file headers, naming, README casing) apply to the repo root and to
this file, **not** to the contents of the platform folders. Anything inside them
is the platform's format, kept byte-for-byte as that platform expects it.

Add a new platform as a sibling folder named after it (e.g. `storybook/`), and
add a row to the table above.
