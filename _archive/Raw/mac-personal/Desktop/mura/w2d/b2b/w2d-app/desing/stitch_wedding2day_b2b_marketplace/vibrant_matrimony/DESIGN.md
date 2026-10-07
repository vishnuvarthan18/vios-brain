---
name: Vibrant Matrimony
colors:
  surface: '#fbf9f9'
  surface-dim: '#dbdad9'
  surface-bright: '#fbf9f9'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f5f3f3'
  surface-container: '#efeded'
  surface-container-high: '#e9e8e7'
  surface-container-highest: '#e3e2e2'
  on-surface: '#1b1c1c'
  on-surface-variant: '#5c403b'
  inverse-surface: '#303031'
  inverse-on-surface: '#f2f0f0'
  outline: '#906f6a'
  outline-variant: '#e5beb7'
  surface-tint: '#bb1809'
  primary: '#b81406'
  on-primary: '#ffffff'
  primary-container: '#dc321f'
  on-primary-container: '#fffbff'
  inverse-primary: '#ffb4a7'
  secondary: '#5f5e5e'
  on-secondary: '#ffffff'
  secondary-container: '#e2dfde'
  on-secondary-container: '#636262'
  tertiary: '#5a5c5d'
  on-tertiary: '#ffffff'
  tertiary-container: '#737576'
  on-tertiary-container: '#fcfdfe'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#ffdad4'
  primary-fixed-dim: '#ffb4a7'
  on-primary-fixed: '#400100'
  on-primary-fixed-variant: '#920600'
  secondary-fixed: '#e5e2e1'
  secondary-fixed-dim: '#c8c6c5'
  on-secondary-fixed: '#1c1b1b'
  on-secondary-fixed-variant: '#474746'
  tertiary-fixed: '#e1e3e4'
  tertiary-fixed-dim: '#c5c7c8'
  on-tertiary-fixed: '#191c1d'
  on-tertiary-fixed-variant: '#454748'
  background: '#fbf9f9'
  on-background: '#1b1c1c'
  surface-variant: '#e3e2e2'
typography:
  headline-xl:
    fontFamily: Plus Jakarta Sans
    fontSize: 48px
    fontWeight: '800'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 24px
    fontWeight: '700'
    lineHeight: 32px
  body-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
  headline-lg-mobile:
    fontFamily: Plus Jakarta Sans
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 36px
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
  sm: 16px
  md: 24px
  lg: 48px
  xl: 80px
  gutter: 24px
  margin-mobile: 16px
  margin-desktop: 64px
---

## Brand & Style

The design system is centered around a celebration of joy, energy, and clarity. Drawing inspiration from the provided logo, the aesthetic is **Modern Corporate with a Friendly twist**. It prioritizes a high-contrast relationship between a bold, energetic red and a pristine white canvas, evoking feelings of excitement and efficiency.

The target audience consists of couples and vendors who need a reliable, fast, and optimistic platform. The visual style uses generous whitespace and soft geometry to balance the intensity of the primary red, ensuring the interface remains approachable and easy to navigate during high-stakes planning moments.

## Colors

The palette is dominated by the **Primary Red (#E53925)**, used strategically for calls-to-action, brand signifiers, and critical interactive states. To maintain a clean and sophisticated look, the background remains predominantly white.

- **Primary:** The vibrant red from the logo. Use for primary buttons, active icons, and progress indicators.
- **Secondary:** A deep charcoal/black for high-contrast typography and structural elements.
- **Neutral:** A range of cool greys. Use for borders, secondary text, and subtle surface variations to create depth without clutter.
- **Surface:** Off-white or very light grey is used for card backgrounds and section containers to differentiate from the main page background.

## Typography

This design system utilizes **Plus Jakarta Sans** across all levels to ensure a cohesive, friendly, and modern tone. The geometric nature of the font complements the rounded shape language of the UI.

- **Headlines:** Use heavy weights (Bold/ExtraBold) with tight letter spacing for a punchy, editorial feel. 
- **Body:** Standard weights with generous line heights ensure maximum readability for long-form content like vendor descriptions or planning tips.
- **Labels:** Use semi-bold or bold weights, often in all-caps for metadata, small captions, or button text to distinguish them from body copy.

## Layout & Spacing

The layout follows a **Fluid Grid** model based on a 12-column system for desktop and a 4-column system for mobile. 

- **Grid:** On desktop, use a 12-column grid with 24px gutters. Content should be centered with a max-width of 1280px.
- **Rhythm:** An 8px linear scale (base 4px) governs all padding and margins. 
- **Mobile Adaptivity:** Margins shrink to 16px on mobile devices. Cards and input fields should transition to full-width or stacked layouts to maintain touch-target accessibility.

## Elevation & Depth

To keep the design feeling light and clean, the design system employs **Tonal Layers** combined with **Low-contrast Outlines**. 

- **Shadows:** Use extremely soft, ambient shadows (0px 4px 20px rgba(0,0,0,0.05)) for primary cards to lift them off the background.
- **Tonal Tiers:** Use subtle background color shifts (e.g., White to #F8F9FA) to define different functional areas of a page without relying on heavy borders.
- **Interactivity:** Elements like buttons should feel tactile, using a slight scale-down (0.98x) on press rather than heavy shadow changes.

## Shapes

In alignment with the logo's aesthetic, the design system uses a **Rounded** shape language. This softens the intensity of the primary red and makes the interface feel more welcoming.

- **Core Elements:** Buttons, input fields, and small cards use a 12px (0.75rem) corner radius.
- **Large Containers:** Hero sections or large modal windows should use `rounded-xl` (1.5rem) to emphasize the soft, modern container style.
- **Icons:** Should be encased in circular or heavily rounded square containers when used as standalone brand elements.

## Components

- **Buttons:** Primary buttons are solid Primary Red with White text. Secondary buttons use a Primary Red outline with Red text. High-emphasis buttons utilize 12px rounded corners.
- **Inputs:** Clean white backgrounds with a 1px Light Grey border. On focus, the border transitions to Primary Red with a subtle outer glow.
- **Cards:** White surfaces with a 12px radius and a very soft ambient shadow. Avoid heavy borders; use white space to separate content within the card.
- **Chips:** Small, highly rounded (pill-shaped) elements. For categories, use a light red tint background with dark red text.
- **Icons:** Use a medium stroke weight. When icons represent primary actions, they should be rendered in Primary Red.
- **Lists:** Clean, borderless list items separated by subtle horizontal dividers (#F0F0F0) or simple vertical spacing.