---
source: personal Mac ~/Downloads/tamil_harvest/tools/font/README.txt
---

Semmozhi Brahmi font tool
=========================
Needs Python 3 with: fonttools, shapely, skia-pathops, brotli, uharfbuzz, scikit-image, scipy, pillow, playwright (chromium).

Files
  fit.py       reads the reference font (put it at ref/SemmozhiBrahmi-Regular.ttf), finds the centre-line of every letter
               and writes fitted.py (pen strokes on our grid).
  glyphs.py    our hand-drawn letters. With FITMODE=choose it swaps in the fitted strokes listed in choose.json.
  build.py     builds SemmozhiBrahmi-Regular.ttf / .woff / .woff2 into the folder you give it.
  score2.py    renders reference and new font in Chromium and prints a match score per character (0 to 1).

Run
  python3 fit.py
  FITMODE=choose python3 build.py out
  python3 score2.py out/SemmozhiBrahmi-Regular.ttf 20      # 20 lowest scores

Licence note: the fitted letters follow the centre-lines of Noto Sans Brahmi (SIL OFL 1.1, Copyright 2022 The Noto Project Authors).
