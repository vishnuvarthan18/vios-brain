# Image provenance — current redesign

The initial redesign reused existing user-supplied and previously prepared assets. A follow-up image edit on 23 September replaced the cropped restaurant-menu source with a complete square product composition. Production images are resized and compressed into AVIF, WebP and JPEG; source files are preserved.

`src/data/studio-images.json` records exact source paths and optimized dimensions for the current image set. Production derivatives live in `src/public/images/studio/` with 480-pixel alternatives. The selected main banner is served at sufficient resolution on mobile to support its cropped composition.

- Main banner: `prodcut-images-final/main-website-banner.png`.
- Photo frames: couple, family and birthday images from the final folder.
- Key tags: supplied photo, calendar and music-pair images. Round/event and patterned images are preserved but not crowded into the main page. The calendar is an illustrative design; customer dates must be checked before engraving. The site makes no claim that the sample music graphic is a functional scannable code.
- Restaurant menus: `output/imagegen/menu-uncropped/menu-uncropped.png`, created with one built-in imagegen edit using the prior menu crop and supplied `menu.jpg` poster. The edit retains the Culture Cafe design, extends the framing to show all book edges, removes the foreground signs and uses a square composition. It is a product design visual, not a new workshop photograph. The exact prompt is saved alongside it in `prompt.txt`; production derivatives are `menu-full.*`. The prior crop is preserved in source.
- Souvenirs: supplied couple cutout, white-framed portrait and success award. Other provided designs remain available in source, rather than being shown repeatedly.
- Other laser work: supplied ALUMKA Caterers nameplate example; it is not presented as a customer endorsement.
- CNC work gallery: six existing images from `prodcut-images-final/cnc/`: carved figure panel, wedding cutwork panel, carved door, Culture Cafe sign, round tables and Sri Lanka display. These previously prepared product visuals are preserved as supplied and compressed to `cnc-wall-art`, `cnc-cutwork`, `cnc-door`, `cnc-sign`, `cnc-tables` and `cnc-display` in the studio assets. No new imagery was generated for this gallery. The former single fabrication illustration remains in source.
- Workshop: original `2023-09-09-laser-engraver-workshop-18.JPG`, `2023-09-09-laser-engraver-workshop-6.JPG` and `2022-04-18-collective-cnc-fixing-8.jpg` from `prodcut-images-final/workshop-area-photos/`. Captions describe the visible work. Unselected workshop photographs remain in source.
- The supplied bird mark, favicon and existing social-preview image are preserved.
