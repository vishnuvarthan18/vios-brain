---
name: Wedding2day Design System
colors:
  surface: '#fcf9f8'
  surface-dim: '#dcd9d9'
  surface-bright: '#fcf9f8'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f6f3f2'
  surface-container: '#f0eded'
  surface-container-high: '#eae7e7'
  surface-container-highest: '#e5e2e1'
  on-surface: '#1c1b1b'
  on-surface-variant: '#564145'
  inverse-surface: '#313030'
  inverse-on-surface: '#f3f0ef'
  outline: '#897174'
  outline-variant: '#ddbfc3'
  surface-tint: '#a73453'
  primary: '#6c0029'
  on-primary: '#ffffff'
  primary-container: '#8b1e3f'
  on-primary-container: '#ff9db0'
  inverse-primary: '#ffb2bf'
  secondary: '#755b00'
  on-secondary: '#ffffff'
  secondary-container: '#fed255'
  on-secondary-container: '#735a00'
  tertiary: '#333432'
  on-tertiary: '#ffffff'
  tertiary-container: '#4a4a48'
  on-tertiary-container: '#bbb9b7'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#ffd9de'
  primary-fixed-dim: '#ffb2bf'
  on-primary-fixed: '#3f0015'
  on-primary-fixed-variant: '#871b3c'
  secondary-fixed: '#ffe08e'
  secondary-fixed-dim: '#ecc246'
  on-secondary-fixed: '#241a00'
  on-secondary-fixed-variant: '#584400'
  tertiary-fixed: '#e4e2df'
  tertiary-fixed-dim: '#c8c6c4'
  on-tertiary-fixed: '#1b1c1a'
  on-tertiary-fixed-variant: '#474745'
  background: '#fcf9f8'
  on-background: '#1c1b1b'
  surface-variant: '#e5e2e1'
typography:
  display-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 30px
    fontWeight: '700'
    lineHeight: 38px
    letterSpacing: -0.02em
  headline-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 24px
    fontWeight: '700'
    lineHeight: 32px
  headline-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  body-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 26px
  body-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  button-text:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 24px
  label-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 14px
    fontWeight: '500'
    lineHeight: 20px
    letterSpacing: 0.01em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 4px
  xs: 8px
  sm: 12px
  md: 16px
  lg: 24px
  xl: 32px
  container-margin: 16px
  gutter: 12px
  tap-target-min: 48px
---

## Brand & Style

The design system is engineered for a high-utility B2B marketplace environment, specifically tailored for wedding vendors and service providers. The brand personality is **authoritative, dependable, and efficient**. It prioritizes clarity and ease of use for a demographic that may include non-technical users operating in high-pressure, outdoor, or mobile-first environments.

The visual style is **Corporate Modern with a Utility focus**. It avoids decorative flourishes in favor of high-contrast elements, clear information hierarchy, and large tap targets. The aesthetic borrows the structural reliability of industrial marketplaces while injecting a sense of professional elegance through its refined color palette, ensuring the platform feels like a serious business tool rather than a social media app.

## Colors

The palette is designed for maximum legibility under varied lighting conditions, such as outdoor event sites.

- **Primary (Deep Maroon):** Used for primary actions, branding, and active states. It conveys maturity and tradition.
- **Accent (Gold):** Used sparingly for highlighting premium listings, ratings, or verified badges.
- **Background (Off-white):** A soft, warm neutral that reduces eye strain compared to pure white while maintaining high contrast with text.
- **Text (Near-black):** Applied to all body and heading content to ensure WCAG AAA compliance for readability.
- **Semantic Colors:** Success (Green) and Error (Red) are used for transaction statuses and form validation, utilizing slightly desaturated tones to remain professional.

## Typography

This design system utilizes **Plus Jakarta Sans** for its exceptional legibility and modern, approachable geometric terminals. The type scale is intentionally large to accommodate users in active environments.

- **Scale:** A strict minimum of 16px is maintained for all body copy to ensure readability for all age groups.
- **Weights:** Use Semi-bold (600) and Bold (700) for headers to create a clear visual anchor on content-heavy pages.
- **Buttons:** Text in primary buttons is set to 18px Bold to emphasize the primary call to action.
- **Spacing:** Increased line-heights (1.5x) are applied to body text to prevent "text crowding" on mobile screens.

## Layout & Spacing

The design system employs a **Fluid Grid** model optimized for mobile devices, using a base-4 spacing scale.

- **Margins:** A standard 16px horizontal margin is applied to the main container to prevent content from hitting the screen edges.
- **Vertical Rhythm:** Use 24px (lg) or 32px (xl) spacing between logical sections (e.g., between a product description and the vendor profile) to maintain a clean, uncluttered interface.
- **Tap Targets:** Every interactive element (buttons, icons, list items) must adhere to a minimum tap target of 48x48px to ensure ease of use for non-technical or elderly users.
- **B2B Listing Density:** While whitespace is generous, list items are structured to show critical information (Price, Location, Availability) at a glance without requiring a tap.

## Elevation & Depth

To maintain a utilitarian and trustworthy feel, the design system uses a **Tonal Layering** approach combined with subtle shadows.

- **Level 0 (Background):** Off-white (#FAF8F5).
- **Level 1 (Cards/Surface):** Pure White (#FFFFFF) with a very soft, diffused shadow (Offset: 0, 4px; Blur: 12px; Opacity: 5% Black). This differentiates the content area from the page background.
- **Level 2 (Modals/Overlays):** White with a more pronounced shadow (Blur: 24px; Opacity: 10% Black) to focus user attention.
- **Outlines:** Use a 1px solid border (#E0DCD6) for input fields and secondary containers instead of shadows to keep the UI looking "structured" and "tool-like."

## Shapes

The design system uses a **Rounded** shape language to soften the industrial nature of a marketplace and make the app feel welcoming.

- **Components:** Buttons and Cards utilize a 12px (0.75rem) corner radius.
- **Input Fields:** 8px (0.5rem) radius for a more structured, functional appearance.
- **Visual Consistency:** Ensure that imagery (vendor logos, product photos) follows the 12px rounding to maintain a unified visual language across the platform.

## Components

### Buttons
- **Primary:** Full-width (on mobile), Deep Maroon background, White text. 12px corner radius. Used for "Contact Vendor" or "Book Now."
- **Secondary:** Transparent background, Deep Maroon 1px border and text. Used for "View Details" or "Add to Shortlist."

### Cards
- White background, 12px radius, soft shadow.
- Cards should have a "Header" section for the service name and a "Footer" section for price and location, clearly separated by whitespace.

### Input Fields
- 16px body text, 8px radius, #E0DCD6 border.
- Floating labels or persistent top-aligned labels are preferred for clarity during data entry.
- Active state: 2px border in Deep Maroon.

### Chips
- Used for categories (e.g., "Catering," "Photography").
- Pill-shaped (fully rounded), light grey background with 14px labels.

### Lists
- Standardized row height of 72px for list items.
- High-contrast dividers (1px, #E0DCD6) between items to ensure distinct clickable areas.

### Bottom Sheets
- Used for filtering and sorting options.
- Features a handle at the top and uses the Level 2 elevation shadow.