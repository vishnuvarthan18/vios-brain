# Semmozhi website (local) status — 2026-09-24

Folder: Mac ~/Downloads/tamil_harvest/website_live. Run: python3 -m http.server 8000, open http://localhost:8000

Pages (nav order): index.html (Brahmi Lab, home), scripts.html (Old Tamil scripts overview), grantha.html, vatteluttu.html, tamil.html, fonts.html (all four fonts, v3.0). font.html = Brahmi font details.
Shared shell: js/app.js (nav, footer, art()), css/style.css (older backup: css/style.before-tamil.css).

Style v2 (user rejected v1 as "worst", chose "Temple and palm-leaf"): kumkum-red header with gold temple border; palm-leaf paper background; olai-style cards with string holes; gold diamond dividers under h1/h2. Every page gets a full-width animated scene (art() in app.js, inserted at top of main): sunrise with turning rays, three gopurams rising, palms, drifting clouds, birds, row of flickering deepam lamps, kolam row that draws itself, floating Tamil/Brahmi/Grantha letters. Cards fade in on scroll. Respects prefers-reduced-motion. Dark theme = deep maroon with gold.
Possible polish: palm fronds look like wings; kolam row is small.
Fixed earlier: unescaped quotes in tamil.html / vatteluttu.html sample chips.
Clutter the user can delete: js/app-1.js, css/style-1.css, semmozhi_v3.tar, _v3/, _old_zips/.
Not done: Vatteluttu waits for Kniprath review before going live; hosting/domain paused.
