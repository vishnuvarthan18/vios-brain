# Verification — website updates, 23 September 2026

CNC gallery and spacing finalisation: added an automatic six-image carousel with a custom WhatsApp enquiry below. It pauses during hover/focus, outside the viewport, in hidden tabs and for reduced-motion preferences; manual navigation pauses automatic playback. Arrow, keyboard, swipe, wraparound, pause and reduced-motion checks pass in Chrome and WebKit. All 18 browser cases passed after correcting the synthetic Chrome touch fixture. The production build passes. Desktop/mobile screenshots were reviewed. Collection gaps now use a single 88px desktop / 56px mobile rhythm instead of doubled adjacent padding; excess fixed text heights were removed.

23 September image-framing correction: replaced the tightly cropped menu source with a square image edit showing the full book. Chrome and WebKit checks at 320, 390, 768, 1024 and 1440 pixels confirm a 1:1 rendered image, square source dimensions, `object-fit: contain`, the new asset URL and no horizontal overflow. Desktop/mobile screenshots were visually inspected; all four book edges are visible. The production build passes.

Follow-up menu redesign: collection navigation, section numbering, source data and structured data now follow Photo Frames → Souvenirs → Key Tags → Restaurant Menus. The menu panel uses a compact light layout with the complete photograph. Desktop/mobile screenshots were reviewed; all 10 affected Chrome/WebKit cases pass for collection order/prices, WhatsApp links, responsive layout, accessibility and rendered images/metadata.

- TypeScript production build and lint pass.
- All 16 browser cases are covered across Chrome and iPhone-sized WebKit. The first run passed 14 cases; the two contrast failures were corrected and affected accessibility, layout and navigation cases passed on both browsers. A final accessible-name correction was checked separately on both browsers.
- The visible collection order is Photo Frames, Souvenirs, Key Tags and Restaurant Menus. Starting prices are LKR 1,200, 2,000, 180 and 2,500 respectively, taken from the supplied flyers and explicitly presented as range prices. Individual sizes, designs and pair prices are not invented. CNC has custom enquiries without prices.
- Photo-frame controls, keyboard arrows, thumbnails, horizontal swipe, vertical-scroll handling and suppression of accidental order clicks after a swipe pass.
- WhatsApp links use the correct number, selected design and range context, and request the full price catalogue. CNC messages request project details, dimensions and quantity. Popup navigation and local analytics events were checked with requests intercepted; no message or order was sent.
- Mobile navigation passes focus containment, Escape, focus restoration and anchor navigation checks.
- Layout checks pass at widths of 320, 375, 390, 768, 1024 and 1440 pixels without horizontal page overflow. Interactive thumbnail targets meet the checked 44-pixel minimum.
- Scroll reveals and progress work; reduced-motion preferences disable these effects and image transitions. The map loads only near the visit section.
- Automated WCAG A/AA checks pass on the page and mobile navigation, including the visible-label/accessibility-name rule.
- Responsive AVIF image URLs return successfully. Pre-rendered markup includes four collection offers with the correct starting prices. Internal links and the 404 route pass.
- Desktop and mobile screenshots were visually reviewed for the banner, photo-frame selector, laser collections, CNC, actual workshop photographs, five-step process, contact and location sections.

## Mobile Lighthouse, local production build

- Performance: 90/100
- Accessibility: 100/100
- Best practices: 100/100
- SEO: 100/100
- Largest Contentful Paint: 3.4 s
- Cumulative Layout Shift: 0.002
- Initial transfer: 693 KiB

The final Lighthouse run preceded a whitespace correction to the brand's accessible label, which was subsequently verified with the browser accessibility checks. These are local browser/emulation and laboratory results, not physical-device or field measurements. Real analytics account delivery and WhatsApp app behaviour are not claimed verified. The existing private Sites audience is preserved.
