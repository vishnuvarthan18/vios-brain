# Template Source CSS Audit
araCreate Template · Webflow Export

**Date:** 2026-10-06  
**File analyzed:** `aracreate-template.webflow.b200a682a.css` (289 KB, 15,826 lines)  
**Source:** Webflow export / original template (NOT production site)

---

## 1. Colors

### Primary Palette (Hex Values)

| Color | Count | Usage | Notes |
|-------|-------|-------|-------|
| #fff (white) | 25 | Primary bg, text | Highest frequency; solid white |
| #333 (dark gray) | 5 | Text, primary dark | Default text color |
| #222 (very dark) | 6 | Headings, dark text | - |
| #0000 (transparent black) | 18 | Fills, overlays | Transparent variant of #000 |
| #ddd (light gray) | 6 | Borders, dividers | - |
| #ccc (medium gray) | 5 | Borders, subtle dividers | - |
| #999 (gray) | 2 | Secondary text | - |
| #2e2e2e (near-black) | 2 | Dark text/bg | - |

### Color Overlays & Transparency

| Value | Count | Usage |
|-------|-------|-------|
| #2e2e2e33 (dark 20%) | 15 | Shadow/overlay | Most common overlay |
| #2e2e2e80 (dark 50%) | 10 | Darker overlays | - |
| #2e2e2e66 (dark 40%) | 2 | Medium overlay | - |
| #0000001a (black 10%) | 2 | Subtle shadows | Used in badge/small elements |
| #3347a099 (blue 60%) | 4 | Hover state overlay | Primary accent with transparency |
| #fff0 (white transparent) | 5 | Reset color | - |
| #75869600 (grayish transparent) | 5 | Reset/overlay | - |
| #3336 (dark 20%) | 1 | Focus ring | - |

### Accent Colors (Limited, Strategic Use)

| Color | Count | Usage |
|-------|-------|-------|
| #3347a0 (primary blue) | 4 | Primary CTA, links | Clear brand primary |
| #3898ec (bright blue) | 2 | Button default | Secondary action color |
| #0082f3 (sky blue) | 2 | Accent | - |
| #2f9bff (vibrant blue) | 1 | Accent variant | - |
| #ea384c (red) | 1 | Error/warning | One-off use |
| #ff0 (yellow) | 1 | Mark/highlight | Mark element default |
| #ffdede (light red) | 1 | Error bg | Light error background |

### Green Tones (Accent/Positive)

| Color | Count | Usage |
|-------|-------|-------|
| #b0eed8 (light mint) | 1 | Positive/success | - |
| #90e4c6 (mint) | 1 | Accent variant | - |

### Assessment

**Likely real values:** The core palette is clean: neutrals (#fff, #333, #222, grays), one primary blue (#3347a0), secondary blues (#3898ec, #0082f3), and scattered accent colors (red, mint). 

**Noise/utility:** All the overlay versions with opacity (#2e2e2e33, etc.) appear functional; they're not noise but rather deliberate shadow/overlay treatments. The #0000 and #fff0 entries are CSS resets.

**Color count:** ~40 unique hex/rgba values, but ~15–20 are actual brand colors; the rest are overlays and resets.

---

## 2. Typography

### Font Families

| Family | Count | Notes |
|--------|-------|-------|
| Poppins, sans-serif | 41 | **Primary typeface** — clearly dominant |
| Red Hat Mono, sans-serif | 2 | Monospace variant — code/technical |
| webflow-icons | 2 | Icon font | System/UI icons |
| Arial, sans-serif | 1 | System fallback | Minimal use |
| Helvetica Neue, Helvetica, Ubuntu, Segoe UI, Verdana, sans-serif | 1 | Legacy fallback stack | Rare |
| serif | 1 | Fallback | - |
| sans-serif | 1 | Generic fallback | - |
| monospace | 1 | Generic fallback | - |
| inherit | 1 | Inherit from parent | - |

**Assessment:** Poppins is the **de facto standard font** (41 occurrences). Red Hat Mono is a secondary typeface for code blocks.

### Font Sizes (Top Values)

| Size | Count | Notes |
|------|-------|-------|
| 14px | 30 | **Most common** — body text baseline |
| 13px | 17 | Small/secondary text |
| 18px | 9 | Subheadings/larger text |
| 15px | 9 | Medium text variant |
| 24px | 7 | Large headings |
| 35px | 6 | Display size |
| 16px | 6 | Standard/slightly large |
| 12px | 6 | Captions/small text |
| 45px, 34px | 5 each | Large display |
| 37px, 30px | 4 each | Display |
| 21px, 20px | 4 each | Medium headings |

**Total unique sizes:** 172 (highly varied — likely includes many one-off component sizes)

**Assessment:** A rough scale emerges: 12px (min), 14px (base), 15–16px (medium), 18–24px (headings), 30–45px (display). Sizes 50px–280px appear to be layout/sizing hacks, not typography.

### Font Weights

| Weight | Count | Notes |
|--------|-------|-------|
| 500 | 23 | **Most common** — medium-weight default |
| 400 | 17 | Regular text |
| 600 | 10 | Semi-bold — headings |
| 300 | 9 | Light — secondary/subtle |
| 200 | 2 | Extra-light |
| 100 | 2 | Thin |
| bold (700) | 4 | Bold |
| normal | 6 | Normal weight inherit |

**Assessment:** Clear weight hierarchy: 300 (light), 400 (regular), 500 (medium/default), 600 (semi-bold), with rare 700 (bold). No 800/900 (black) weights used. Weight 500 is the **design default**.

### Line Heights (Top Values)

| Height | Count | Notes |
|---------|-------|-------|
| 1.5em | 23 | **Most common** — relaxed leading |
| 1.4em | 16 | Standard leading |
| 22px, 1.9em | 4 each | Tighter/slightly larger |
| 0 | 4 | Icon fonts/no-leading (reset) |
| inherit, 1em | 3 each | Inherited/minimal |
| 2.1em | 3 | Loose (special case) |
| 1.6em, 1.8em, 2em | 2 each | Variations |

**Total unique values:** 89 (many are component-specific one-offs)

**Assessment:** Core system: 1.4em–1.5em is standard (most compositions), with 1.6em–2.1em for emphasis and 1em or 0 for compact/icon contexts. No tight (<1.2em) leading observed.

### Letter Spacing (Tracking)

| Value | Count | Usage |
|-------|-------|-------|
| .03em | 10 | Tight — often buttons/CTA |
| .07em | 6 | Open — often headings |
| .02em | 6 | Very tight — body/dense |
| 0 | 5 | No tracking |
| .05em | 2 | Light tracking |
| .04em | 2 | Minimal tracking |
| .15em | 4 | Very open — display/branding |
| .08em, .12em | 1 each | Variations |
| 50px (!), -.03em, -.02em | 1 each | Outliers/possible errors |

**Assessment:** Intentional system: very tight (0–.03em) for body/CTAs, open (.07em–.15em) for headings/display. The 50px entry is likely a typo or margin misclassification. The value system suggests deliberate hierarchy.

### Text Transform

| Value | Count | Usage |
|-------|-------|-------|
| none | 3 | No transform (default) |
| uppercase | 1 | All caps — rare, special use |
| inherit | 1 | Inherited | - |

**Assessment:** Mostly no transform; uppercase is rare/deliberate (accent). No lowercase or small-caps.

---

## 3. Spacing (Padding & Margin)

### Padding Values (Most Common)

| Value | Count | Notes |
|-------|-------|-------|
| 0 | 11 | Reset/no padding |
| 8px 12px | 3 | Compact button/element |
| 20px | 3 | Standard padding |
| 40px, 30px | 2 each | Generous padding |
| 10px, 10px 20px | 2 each | Standard/medium |
| 9px 15px | 1 | Button (default .w-button) |
| 27px 77px, 25px 52px | 2 each | Large horizontal padding |
| 47px 45px, 30px 45px | 2 each | Block-level padding |

**Assessment:** Rough scale: 8–10px (tight), 20px (medium), 30–40px (generous), 45–77px (block sections). No consistent 4px/8px grid observed; values are varied but appear intentional.

### Margin Values (Most Common)

| Value | Count | Notes |
|-------|-------|-------|
| 0 | 9 | Reset/no margin |
| auto | 5 | Center alignment |
| 0 0 10px | 2 | Vertical spacing |
| Others | 1 each | Scattered one-offs |

**Assessment:** Margin usage is minimal; mostly reset (0) or auto-center. Vertical rhythm is loose (10px between sections).

### Spacing Assessment

**Pattern observed:** No strict 4px or 8px modular scale. Spacing is **ad-hoc and component-specific**. Values cluster around 8–10px (compact), 20px (medium), 30–45px (generous), but there's no systematic ratio.

**Recommendation for tokens:** Extract clusters → loose grid (8px base, 10px/20px/30px/40px/50px+ variants).

---

## 4. Border Radius

| Value | Count | Observation |
|-------|-------|-------------|
| 20px | 13 | **Primary radius** — standard rounded corners |
| 9px | 4 | Medium roundness |
| 4px | 3 | Subtle roundness |
| 0 | 3 | No roundness (reset) |
| 50%, 100% | 2, 1 | Circles/pills |
| 100px | 1 | Large radius (buttons?) |
| 200px | 1 | Extra-large (pills) |
| 3px, 2px | 1 each | Minimal roundness |

**Assessment:** Clear system: 0 (sharp), 4–9px (subtle–medium), **20px (standard/default)**, 50–100%+ (pills/circles). The 20px radius is the **design default**.

---

## 5. Box Shadow

| Value | Count | Context |
|-------|-------|---------|
| 0 4px 9px -3px #3347a099 | 4 | Primary shadow (blue-tinted) — most common |
| 0 4px 9px -3px #2e2e2e33 | 1 | Alternate shadow (dark-tinted) |
| 0 0 3px #3336 | 1 | Tight outline/focus ring |
| 0 0 0 2px #fff | 1 | White outline |
| 0 0 0 1px #0000001a, 0 1px 3px #0000001a | 1 | Double shadow (outline + drop) — badge/delicate |
| none | 2 | No shadow |

**Assessment:** Limited shadow vocabulary: one primary shadow (blue), one alternate (dark), rare outlines/focus rings. The shadow system is **minimal and intentional**. Most components use either a prominent drop shadow or no shadow.

---

## 6. Transitions & Animations

### Transition Values

| Value | Count | Context |
|-------|-------|---------|
| opacity .4s | 6 | Fade in/out — most common |
| background-color .8s cubic-bezier(.165, .84, .44, 1), transform .8s cubic-bezier(.165, .84, .44, 1) | 3 | Smooth color + transform shift |
| transform .8s cubic-bezier(.165, .84, .44, 1), background-color .8s cubic-bezier(.165, .84, .44, 1) | 1 | Reverse order variant |
| color .6s | 1 | Text color shift |
| background-color .1s, color .1s | 1 | Quick color updates (hover) |
| all .3s | 1 | Catch-all general transition |
| none | 1 | No transition |

**Easing observed:** 
- Cubic-bezier(.165, .84, .44, 1) — **appears to be a custom ease (likely ease-out or custom bounce-like)**
- Linear implied for others

**Durations observed:** 
- 0.1s (quick hover)
- **0.4s (standard fade)**
- 0.6s (text color)
- **0.8s (emphasis moves)**

### Animation Duration
- 8s (one instance) — likely a loop/keyframe animation

**Assessment:** Clean motion system: quick hover reactions (0.1s), standard fades (0.4s), emphasis moves (0.8s). The custom cubic-bezier is used for transform+color combos, suggesting a unified interaction language. No complex keyframe sequences observed.

---

## 7. Breakpoints (Media Queries)

| Breakpoint | Count | Direction | Notes |
|------------|-------|-----------|-------|
| max-width: 767px | 5 | Down | Tablet/mobile breakpoint |
| max-width: 479px | 5 | Down | Mobile breakpoint |
| max-width: 991px | 4 | Down | Large tablet/medium breakpoint |
| min-width: 1920px | 2 | Up | Large desktop |
| min-width: 1440px | 2 | Up | Desktop |
| min-width: 1280px | 2 | Up | Large desktop |
| min-width: 768px | 1 | Up | Tablet+ |

**Observed breakpoints (sorted):** 479px, 768px, 767px, 991px, 1280px, 1440px, 1920px

**Assessment:** **Mixed mobile-first and desktop-first** approaches coexist (both max-width and min-width queries). Core breakpoints are: **479px (mobile)**, **767px (tablet)**, **991px (medium)**, with plus-up variants at 1280px, 1440px, 1920px. A cleaner system would pick one direction (prefer mobile-first).

---

## 8. Design System Readiness

### What's Real (Foundations)

✓ **Primary font:** Poppins (clear, consistent)  
✓ **Core color palette:** Neutrals + one primary blue + limited accents (~15 true colors)  
✓ **Font weight hierarchy:** 300/400/500/600 (clean range)  
✓ **Leading system:** 1.4–1.5em (standard), 1.6–2.1em (accent)  
✓ **Shadow vocabulary:** 1–2 primary shadows (minimal, intentional)  
✓ **Rounded corners:** Clear default (20px) with variants (4px, 9px, 50%+)  
✓ **Motion:** 3 core durations (0.1s, 0.4s, 0.8s) with one custom easing  

### What's Noise (or Ad-Hoc)

✗ **Typography scale:** 172 unique font sizes (real scale is ~15–20 sizes; rest are one-offs)  
✗ **Spacing scale:** No clear modular grid; values are scattered (8px/10px/20px/30px/40px)  
✗ **Line-height specifics:** 89 unique values; core system is ~5–8 (1em, 1.4em, 1.5em, 1.6em, 1.8em, 2.1em)  
✗ **Breakpoints:** Mixed mobile-first + desktop-first; inconsistent widths (479, 767, 768, 991)  

### Extraction Strategy for Tokens

1. **Colors:** 15–20 key colors + 3–4 shadow overlays  
2. **Typography:** Poppins only; 6–8 font sizes (core: 12, 14, 16, 18, 24, 32, 45, 65px); weights: 300, 400, 500, 600  
3. **Spacing:** Define 8px base → 8, 12, 16, 20, 24, 32, 40, 48, 56, 64px scale  
4. **Radius:** 0, 4, 9, 20, 50%  
5. **Shadow:** 1 primary, 1 alternate (optional focus rings)  
6. **Motion:** opacity 0.4s, color 0.6s, transform 0.8s + custom cubic-bezier  
7. **Breakpoints:** Standardize → mobile-first: 480px, 768px, 992px, 1280px, 1440px  

---

## Files & Artifacts

**Audit Input:** `/Users/vishnuvarthanvenkatapathy/araCreate/AC/ACDS/01-input/template-source/araCreate Template/_assets/cdn.prod.website-files.com/6378124f2d048e055904a382/css/aracreate-template.webflow.b200a682a.css`

**Next Step:** Use the extracted values above to build design tokens → component library.
