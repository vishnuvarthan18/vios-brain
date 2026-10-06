# SOURCE

The design systems themselves — one folder per platform. Each folder is
self-contained in the layout its platform expects, so it can be consumed
directly with no build step.

| Folder | Platform | Notes |
| --- | --- | --- |
| [`claude-design-system/`](claude-design-system/) | [Claude Design](https://claude.ai/design) | **Export of the master**, which lives in Claude Design. Do not edit here; do not restructure or reformat. |

Because each platform owns its folder's layout, the araCreate repo conventions
apply to the repo root and this file, **not** to the platform folders.

Add a new platform as a sibling folder named after it (e.g. `storybook/`) and a
row above.
