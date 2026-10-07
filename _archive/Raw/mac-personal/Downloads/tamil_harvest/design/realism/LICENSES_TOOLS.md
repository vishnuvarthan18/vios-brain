# Tools and their licenses (Prompt 4)

Every package is installed inside the project only (`design/realism/node_modules`, `design/realism/.venv`).
Nothing here is shipped to `website_live/` unless a row says so. No paid or account-based service is used.
License source: the package's own metadata (`importlib.metadata` / `package.json`), read on 2026-09-26.

| tool | version | where | license | used for | ships to site? |
|---|---|---|---|---|---|
| playwright-core | 1.63.0 | node_modules | Apache-2.0 | driving headless Chrome (rig, evaluators) | no |
| Chrome for Testing | 151.0.7922.34 | ~/Library/Caches/ms-playwright/chromium-1234 (already installed, reused, not downloaded) | Chromium BSD-style + Google terms for CfT builds | fixed render rig | no |
| pngjs | 7.0.0 | node_modules | MIT | decoding screenshots in the compat check | no |
| axe-core | 4.12.1 | node_modules | MPL-2.0 | the accessibility scan in `eval/cross_browser.mjs` (T15); injected into the page under test at run time only | **no** — never copied into `website_live/` |
| numpy | 2.5.3 | .venv | BSD-3-Clause (bundled parts 0BSD, MIT, Zlib, CC0-1.0) | measuring | no |
| scipy | 1.18.1 | .venv | BSD-3-Clause | measuring, EMD assignment | no |
| scikit-image | 0.26.0 | .venv | BSD-3-Clause | CIELAB, CIEDE2000 | no |
| opencv-python-headless | 5.0.0.93 | .venv | Apache-2.0 (wrapper MIT) | masks, resampling | no |
| Pillow | 12.3.0 | .venv | MIT-CMU (HPND) | image I/O | no |
| imageio | 2.37.4 | .venv (scikit-image dependency) | BSD-2-Clause | pulled in, not used directly | no |
| tifffile | 2026.9.20 | .venv (dependency) | BSD-3-Clause | pulled in | no |
| networkx | 3.7 | .venv (dependency) | BSD-3-Clause | pulled in | no |
| lazy-loader | 0.6 | .venv (dependency) | BSD-3-Clause | pulled in | no |
| packaging | 26.3 | .venv (dependency) | Apache-2.0 OR BSD-2-Clause | pulled in | no |
| pypdf | 6.19.0 | .venv | BSD-3-Clause | reading source PDFs for spec-card numbers (T1) | no |
| Node.js | 24.18.0 | system (nvm) | MIT | runtime | no |
| Python | 3.12.3 | system | PSF-2.0 | runtime | no |
| git | 2.50.1 | system | GPL-2.0 | version control (not linked) | no |
| ImageMagick | 6.9.1-0 | /opt/ImageMagick (system) | ImageMagick License (Apache-2.0 style) | not used (Pillow does the job) | no |
| T8 sounds (leaf rustle, stylus scratch, copper chime, chisel) | - | design/realism/t8/sound.js | own work, written in this repo, no third-party content | the optional sound layer | not yet (T8 is a demo page; a site page must keep it muted by default) |
| Web Audio API | browser built-in | browser | part of the browser, no package | playing the synthesized buffers | n/a |
| Semmozhi Tamil font | 3.0 | website_live/fonts (project's own) | SIL OFL 1.1 (website_live/fonts/OFL-Tamil.txt) | carved-letter test text in the rig stage | already on site |

**Sound (T8):** no audio file is used anywhere. All four sounds are synthesized in code in
`design/realism/t8/sound.js` (own work: shaped noise bands, inharmonic partials and exponential decays,
driven by a seeded PRNG). No CC0 or other third-party recording was downloaded or needed, and no sound
library is installed. The network is not used.

Not present: ffmpeg is not on PATH. Playwright's private ffmpeg build (ms-playwright/ffmpeg-1011, LGPL-2.1)
is only for Playwright's own video recording and is not used as a general tool.

Later tasks append rows here (for example three.js for T3) before using a package.
