---
tags: chat
date: 2026-06-05
source: Claude personal account
uuid: ff72bd8f-07b2-401f-bd56-9a1ae4b01098
---
# Designing UI/UX wireframes

## Summary
**Conversation Overview**

The person came to Claude seeking help designing UI/UX wireframes for a mobile marketplace app focused on event decoration items (mandap, backdrop, lighting, props, flowers, name boards). Through a structured intake process using interactive question components, Claude established that the person had a mobile app with existing flows and needed screen layouts created. The person then provided a comprehensive feature list covering the full scope of the app rather than selecting specific screens.

The app is a buy-and-sell marketplace for event decor items targeting manufacturers and decorators, with geographic focus implied to be Tamil Nadu, India. Key features specified included OTP-based signup/login with a manufacturer/decorator role tag, listing creation with category/condition/photos/price/quantity/location fields, browse with search and filters, listing detail pages, a contact/interest flow with privacy controls, an admin web portal for user and listing approval, and push notifications for interest and nearby listings.

Claude first delivered an interactive HTML wireframe widget covering all eight screens with clickable tab navigation, inline annotations explaining UX decisions, and a working admin portal panel. The person then requested actual SVG or image files suitable for pasting directly into Figma. Claude responded by generating a Python script that produced a single combined SVG file (1800×2524px, approximately 58KB) containing all eight screens: seven mobile phone frames (375×760px each, arranged in a 4-column grid) and one web admin portal frame (900×640px). The SVG used standard shapes, system fonts, and no external dependencies to ensure Figma compatibility. Claude also provided instructions for importing, ungrouping layers, and replacing placeholder image boxes with real assets in Figma.

## Chat

**Vishnu:** Hi Claude! Could you design UI/UX wireframes? If you need more information from me, ask me 1-2 key questions right away. If you think I should give you more context or upload anything to help you do a better job, let me know. Use any tools you have access to—like Google Drive, web search, etc.—if they'll help.

If it makes sense, create something we can look at together—like a visual, a checklist, or something interactive. Thanks for your help!

**Claude:** _[tool: ask_user_input_v0]_

**Vishnu:** Q: What are you building? (Pick the closest fit)
A: Mobile app

Q: What stage are you at?
A: I have flows, need screen layouts

**Claude:** Two quick follow-ups before I start:

_[tool: ask_user_input_v0]_

**Vishnu:** • One-account signup/login + OTP, with a manufacturer/decorator tag 
• Create a listing: title, category (mandap, backdrop, lighting, props, flowers, name boards…), condition (used/new), photos, price, quantity, location (city/district) 
• Browse + search + filter by category, condition, price, location 
• Listing detail page 
• Contact / express interest → connect (reveal contact or simple chat) 
• Admin web portal: approve users & listings (your trust gate), remove spam 
• Notifications: someone's interested in your item; new item in your area

**Claude:** _[tool: view]_

_[tool: visualize:read_me]_

_[tool: visualize:show_widget]_

Here's what's covered across the 8 screens — click through each tab to explore:

**Mobile screens (1–7)**
- Login/OTP — single flow for both new and returning users, auto-creates account on first login
- Profile setup — role card (Manufacturer / Decorator) with clear visual distinction; admin-gated
- Browse — horizontal category chips, always-visible condition + location filters, 2-col listing grid
- Listing detail — blurred contact until interest expressed, quantity for bulk sellers, save option
- Create listing — 3-step progress, pill-based category picker, GPS-prefilled location
- Notifications — two types: interest alerts + "new near you" discovery nudges
- Connect — single bottom sheet with message vs. reveal-contact modes; buyer number stays hidden

**Admin portal (screen 8)**
- Dashboard stats + sidebar nav; Pending Users and Pending Listings as separate queues with approve/reject per row

**A few open decisions to think through:**
- Should decorators also be able to list, or only browse? (affects listing flow access)
- Price negotiation — fixed price only, or allow "contact for price"?
- Photo minimum/maximum count per listing?
- District filtering — dropdown or map-based radius?

Let me know which screens you'd like to refine, or if you want to add flows like saved listings, profile page, or the photo upload step.

**Vishnu:** i need like images or svg to past in figma

**Claude:** _[tool: bash_tool]_

_[tool: present_files]_

One SVG file, 1800×2524px, all 8 screens on a single artboard. Here's how to use it in Figma:

**Paste into Figma:**
- Download the file → drag it directly onto a Figma canvas, or `File → Place image`
- All screens land as a single group — ungroup once and each screen is its own element
- Select individual screens, copy, and paste as needed

**To make it fully editable in Figma:**
- After importing, right-click → "Ungroup" to separate the screens
- Each text, rect, and shape becomes an individual Figma layer you can restyle
- Replace the image placeholder rectangles (the X-fill boxes) with real photos

**What's included:**
- Screens 1–7: mobile phone frames (375×760) with status bar, bottom nav, and all UI elements
- Screen 8: admin web portal (900×640) with sidebar, stats row, and approve/reject table

Let me know if you want any screen as a standalone SVG, or if you'd like me to adjust spacing, font sizes, or any specific screen layout.
