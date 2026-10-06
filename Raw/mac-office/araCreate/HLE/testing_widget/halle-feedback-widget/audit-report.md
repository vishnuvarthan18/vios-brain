# Capture audit — 2026-09-21 15:47

Capture code under test: `https://feedback.arametrics.app/capture.js`

Each row is one screenful. "Difference" is the share of pixels where the
widget's picture disagrees with what the browser actually shows. Under
10.0% is a pass — web fonts alone account for a few percent.

| Screen | Page | Scroll | Difference | Verdict |
|---|---|---|---|---|
| desktop | / | 0px | 19.6% | ❌ **BROKEN** |
| desktop | / | 900px | 20.5% | ❌ **BROKEN** |
| desktop | / | 1800px | 19.8% | ❌ **BROKEN** |
| desktop | / | 2700px | 16.7% | ❌ **BROKEN** |
| desktop | / | 3600px | 20.2% | ❌ **BROKEN** |
| desktop | / | 4500px | 15.1% | ❌ **BROKEN** |
| desktop | /about-us | 0px | 2.5% | ✅ pass |
| desktop | /contact | 0px | 7.9% | ✅ pass |
| desktop | /contact | 900px | 15.8% | ❌ **BROKEN** |
| desktop | /contact | 1800px | 15.7% | ❌ **BROKEN** |
| mobile | / | 0px | 35.7% | ❌ **BROKEN** |
| mobile | / | 844px | 25.1% | ❌ **BROKEN** |
| mobile | / | 1688px | 21.5% | ❌ **BROKEN** |
| mobile | / | 2532px | 18.9% | ❌ **BROKEN** |
| mobile | / | 3376px | 20.0% | ❌ **BROKEN** |
| mobile | / | 4220px | 20.2% | ❌ **BROKEN** |
| mobile | / | 5064px | 27.4% | ❌ **BROKEN** |
| mobile | / | 5908px | 27.5% | ❌ **BROKEN** |
| mobile | / | 6752px | 36.7% | ❌ **BROKEN** |
| mobile | / | 7596px | 25.9% | ❌ **BROKEN** |
| mobile | / | 8440px | 24.4% | ❌ **BROKEN** |
| mobile | /about-us | 0px | 6.0% | ✅ pass |
| mobile | /contact | 0px | 12.1% | ❌ **BROKEN** |
| mobile | /contact | 844px | 18.1% | ❌ **BROKEN** |
| mobile | /contact | 1688px | 23.1% | ❌ **BROKEN** |
| mobile | /contact | 2532px | 22.7% | ❌ **BROKEN** |

## Summary

- Screens checked: **26**
- Passed: **3**
- Broken: **23**

## Pages using the pattern that caused the hero bug

**https://halle-dev.webflow.io/** (desktop)

- pseudo-element background: `SECTION.section.position-relative::before` → url("https://cdn.prod.website-files.com/6672e259ffca23748c51b4cd/6a5604034ee8e47ba075dd10_hero-curve-stretch.svg")


## Broken screens — pictures saved for each

- desktop https://halle-dev.webflow.io/ at 0px — 19.6% different, 22 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y0_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y0_capture.png`
- desktop https://halle-dev.webflow.io/ at 900px — 20.5% different, 32 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y900_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y900_capture.png`
- desktop https://halle-dev.webflow.io/ at 1800px — 19.8% different, 36 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y1800_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y1800_capture.png`
- desktop https://halle-dev.webflow.io/ at 2700px — 16.7% different, 21 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y2700_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y2700_capture.png`
- desktop https://halle-dev.webflow.io/ at 3600px — 20.2% different, 32 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y3600_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y3600_capture.png`
- desktop https://halle-dev.webflow.io/ at 4500px — 15.1% different, 21 bad regions → `audit-out/desktop_https_halle_dev_webflow_io__y4500_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io__y4500_capture.png`
- desktop https://halle-dev.webflow.io/contact at 900px — 15.8% different, 23 bad regions → `audit-out/desktop_https_halle_dev_webflow_io_contact_y900_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io_contact_y900_capture.png`
- desktop https://halle-dev.webflow.io/contact at 1800px — 15.7% different, 19 bad regions → `audit-out/desktop_https_halle_dev_webflow_io_contact_y1800_browser.png` vs `audit-out/desktop_https_halle_dev_webflow_io_contact_y1800_capture.png`
- mobile https://halle-dev.webflow.io/ at 0px — 35.7% different, 42 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y0_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y0_capture.png`
- mobile https://halle-dev.webflow.io/ at 844px — 25.1% different, 33 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y844_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y844_capture.png`
- mobile https://halle-dev.webflow.io/ at 1688px — 21.5% different, 36 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y1688_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y1688_capture.png`
- mobile https://halle-dev.webflow.io/ at 2532px — 18.9% different, 27 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y2532_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y2532_capture.png`
- mobile https://halle-dev.webflow.io/ at 3376px — 20.0% different, 29 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y3376_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y3376_capture.png`
- mobile https://halle-dev.webflow.io/ at 4220px — 20.2% different, 29 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y4220_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y4220_capture.png`
- mobile https://halle-dev.webflow.io/ at 5064px — 27.4% different, 45 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y5064_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y5064_capture.png`
- mobile https://halle-dev.webflow.io/ at 5908px — 27.5% different, 50 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y5908_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y5908_capture.png`
- mobile https://halle-dev.webflow.io/ at 6752px — 36.7% different, 64 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y6752_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y6752_capture.png`
- mobile https://halle-dev.webflow.io/ at 7596px — 25.9% different, 48 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y7596_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y7596_capture.png`
- mobile https://halle-dev.webflow.io/ at 8440px — 24.4% different, 39 bad regions → `audit-out/mobile_https_halle_dev_webflow_io__y8440_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io__y8440_capture.png`
- mobile https://halle-dev.webflow.io/contact at 0px — 12.1% different, 16 bad regions → `audit-out/mobile_https_halle_dev_webflow_io_contact_y0_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io_contact_y0_capture.png`
- mobile https://halle-dev.webflow.io/contact at 844px — 18.1% different, 23 bad regions → `audit-out/mobile_https_halle_dev_webflow_io_contact_y844_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io_contact_y844_capture.png`
- mobile https://halle-dev.webflow.io/contact at 1688px — 23.1% different, 37 bad regions → `audit-out/mobile_https_halle_dev_webflow_io_contact_y1688_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io_contact_y1688_capture.png`
- mobile https://halle-dev.webflow.io/contact at 2532px — 22.7% different, 36 bad regions → `audit-out/mobile_https_halle_dev_webflow_io_contact_y2532_browser.png` vs `audit-out/mobile_https_halle_dev_webflow_io_contact_y2532_capture.png`
