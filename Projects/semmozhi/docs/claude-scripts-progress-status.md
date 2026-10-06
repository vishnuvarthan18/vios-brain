# Semmozhi scripts progress (updated 2026-09-25)

## Current state: all four fonts are v3.0, one shared style (on Mac ~/Downloads/tamil_harvest/website_live; run: python3 -m http.server 8000)
- SHARED STYLE (user chose "C Tapered ends"): stroke 72, smooth curves, free stroke ends taper to 40% width over 110 units (config taper 0.6). Same in Brahmi, Grantha, Tamil, Vatteluttu.
- Pipeline for all four is now tools/font/sf.py (fit|build|score) + best.py (tries 10 fit settings per glyph, keeps the best, using the tapered build). Brahmi now comes from Noto Sans Brahmi via sf.py (old hand-drawn/choose pipeline retired). Copy of tools + fitted stroke data + configs: tools/font_v3/ on the Mac. fill "nonzero" was the big Brahmi win.
- Scores (centre-line agreement with reference font within 6% of letter size; not a check against inscriptions):
  - Tamil: 349 entries, avg 99.99%, 349/349 >= 99%, weakest 99.4%
  - Vatteluttu: 272 entries, avg 99.94%, 272/272 >= 99%, weakest 99.1%
  - Grantha: 494 entries, avg 99.97%, 492/494 >= 99%, weakest 92.8%
  - Brahmi: 835 entries, avg 99.7%, 799/835 >= 99%; the rest are mostly dots/digits (candrabindu dot 48% = position), 𑀴𑀼/𑀴𑀽 (85-89%). NOT individually fixed yet.
- Pages updated to v3.0 (zip names SemmozhiX-3.0.zip, status notes). Old zips moved to website_live/_old_zips. website_live/_v3 holds the build bundle + apply.py (safe to delete manually; device cannot delete). semmozhi_v3.tar in website_live can be deleted by the user.
- Pages: index.html (Brahmi Lab), font.html (Brahmi font), grantha.html, tamil.html, vatteluttu.html, scripts.html (Old Tamil scripts overview — DONE), fonts.html (all four fonts index — DONE). Conjuncts in Grantha NOT built. Tamil has no Latin glyphs; new Brahmi font also has no Latin (old one did).
- Visual style: full temple/palm-leaf redesign done 2026-09-24, see website-live-2page-status.md for details.

## Vatteluttu — BUILT (draft), NOT YET PUBLISHED
- Permission from Elmar Kniprath (e-Vatteluttu OT, 2014) to use it as reference under SIL OFL 1.1. Do NOT record his email address anywhere.
- CONDITIONS: (1) do not reproduce letterforms exactly: our version has smoothing, uniform weight, tapered ends; (2) acknowledge e-Vatteluttu OT and him in licence + website (DONE); (3) send him font/page for review BEFORE publishing (TODO — still not sent as of 2026-09-25).
- Font: SemmozhiVatteluttu 3.0 draft. Type ordinary Tamil; font shows Vatteluttu.

## Next (in likely order)
1. Send Kniprath the Vatteluttu draft for review before publishing (he asked to see it first). Not yet sent.
2. Optional: fix remaining weak Brahmi entries (candrabindu dot position, 𑀴𑀼/𑀴𑀽) if the user wants Brahmi past 99.7%.
3. Optional: polish the new temple/palm-leaf animation — user flagged palm fronds look like wings, kolam row is small (see website-live-2page-status.md).
4. Ask which "more" scripts the user wants beyond Brahmi/Grantha/Vatteluttu/Tamil.
5. Expert reviews pending for all fonts (no expert has looked at any of them yet).
6. Later (paused, user said "don't care about server and domain first, build the website"): hosting/domain, Google Drive/rclone, server crawl test — see website-build-status.md.
