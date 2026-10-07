# Vidivu

Next.js 16 + Tailwind CSS v4 homepage. Dark, motorsport-engineering design system.

## Run locally
```bash
npm install
npm run dev
```
Open http://localhost:3000

## Deploy free (Vercel)
```bash
npx vercel
```
Or push to GitHub and import the repo at vercel.com — free tier, one click.

## Structure
- `app/page.tsx` — homepage assembly
- `components/Navbar.tsx`, `Hero.tsx`, `Sections.tsx` — page sections
- `app/globals.css` — design tokens (colors, spacing) as CSS variables + Tailwind v4 `@theme`

## Fonts
Currently using system font fallback (works offline/sandboxed). To use real Inter:
add back `next/font/google` Inter import in `app/layout.tsx` — it'll fetch fine on Vercel's network.

## Brand
Signature tricolor stripe uses original color stops (not any existing brand's exact hex) — safe to use as your own mark.
