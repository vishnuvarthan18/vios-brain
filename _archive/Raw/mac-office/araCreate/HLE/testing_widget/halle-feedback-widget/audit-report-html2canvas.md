# Capture audit — html2canvas test variant — 2026-09-21 16:25

Same clone capture.ts builds, rasterised with html2canvas 1.4.1 instead of
modern-screenshot's SVG-foreignObject domToBlob. Diagnostic only — see
docs/agent-task-html2canvas-full-check.md.

| Screen | Page | Scroll | Difference | Verdict |
|---|---|---|---|---|
| desktop | / | 0px | 15.8% | ❌ **BROKEN** |
| desktop | / | 900px | 7.0% | ✅ pass |
| desktop | / | 1800px | 5.4% | ✅ pass |
| desktop | / | 2700px | 25.3% | ❌ **BROKEN** |
| desktop | / | 3600px | 21.4% | ❌ **BROKEN** |
| desktop | / | 4500px | 4.9% | ✅ pass |
| desktop | /contact | 0px | 1.7% | ✅ pass |
| desktop | /contact | 900px | 2.6% | ✅ pass |
| desktop | /contact | 1800px | 2.8% | ✅ pass |
| desktop | /products-selection/polarizers | 0px | 3.4% | ✅ pass |
| desktop | /products-selection/polarizers | 900px | 30.8% | ❌ **BROKEN** |
| desktop | /products-selection/polarizers | 1800px | 3.7% | ✅ pass |
| desktop | /products-selection/polarizers | 2700px | 4.0% | ✅ pass |
| desktop | /products/glan-thompson-polarizing-prisms | 0px | 5.0% | ✅ pass |
| desktop | /products/glan-thompson-polarizing-prisms | 900px | 6.7% | ✅ pass |
| desktop | /products/glan-thompson-polarizing-prisms | 1800px | 53.0% | ❌ **BROKEN** |
| desktop | /products/glan-thompson-polarizing-prisms | 2700px | 51.5% | ❌ **BROKEN** |
| desktop | /products/glan-thompson-polarizing-prisms | 3600px | 36.4% | ❌ **BROKEN** |
| desktop | /privacy | 0px | 5.6% | ✅ pass |
| desktop | /privacy | 900px | 6.3% | ✅ pass |
| desktop | /privacy | 1800px | 4.0% | ✅ pass |
| desktop | /privacy | 2700px | 6.9% | ✅ pass |
| desktop | /privacy | 3600px | 5.4% | ✅ pass |
| desktop | /privacy | 4500px | 5.6% | ✅ pass |
| desktop | /privacy | 5400px | 5.7% | ✅ pass |
| desktop | /privacy | 6300px | 5.9% | ✅ pass |
| desktop | /privacy | 7200px | 6.8% | ✅ pass |
| desktop | /privacy | 8100px | 5.8% | ✅ pass |
| desktop | /privacy | 9000px | 12.9% | ❌ **BROKEN** |
| desktop | /privacy | 9900px | 11.5% | ❌ **BROKEN** |
| mobile | / | 0px | 32.5% | ❌ **BROKEN** |
| mobile | / | 844px | 8.5% | ✅ pass |
| mobile | / | 1688px | 12.4% | ❌ **BROKEN** |
| mobile | / | 2532px | 6.0% | ✅ pass |
| mobile | / | 3376px | 5.3% | ✅ pass |
| mobile | / | 4220px | 15.9% | ❌ **BROKEN** |
| mobile | / | 5064px | 35.7% | ❌ **BROKEN** |
| mobile | / | 5908px | 26.1% | ❌ **BROKEN** |
| mobile | / | 6752px | 10.0% | ✅ pass |
| mobile | / | 7596px | 8.4% | ✅ pass |
| mobile | / | 8440px | 8.3% | ✅ pass |
| mobile | /contact | 0px | 4.5% | ✅ pass |
| mobile | /contact | 844px | 9.1% | ✅ pass |
| mobile | /contact | 1688px | 5.5% | ✅ pass |
| mobile | /contact | 2532px | 5.6% | ✅ pass |
| mobile | /products-selection/polarizers | 0px | 5.7% | ✅ pass |
| mobile | /products-selection/polarizers | 844px | 9.1% | ✅ pass |
| mobile | /products-selection/polarizers | 1688px | 49.7% | ❌ **BROKEN** |
| mobile | /products-selection/polarizers | 2532px | 34.1% | ❌ **BROKEN** |
| mobile | /products-selection/polarizers | 3376px | 5.9% | ✅ pass |
| mobile | /products-selection/polarizers | 4220px | 4.9% | ✅ pass |
| mobile | /products-selection/polarizers | 5064px | 5.6% | ✅ pass |
| mobile | /products-selection/polarizers | 5908px | 6.6% | ✅ pass |
| mobile | /products/glan-thompson-polarizing-prisms | 0px | 9.9% | ✅ pass |
| mobile | /products/glan-thompson-polarizing-prisms | 844px | 9.5% | ✅ pass |
| mobile | /products/glan-thompson-polarizing-prisms | 1688px | 9.8% | ✅ pass |
| mobile | /products/glan-thompson-polarizing-prisms | 2532px | 19.7% | ❌ **BROKEN** |
| mobile | /products/glan-thompson-polarizing-prisms | 3376px | 47.7% | ❌ **BROKEN** |
| mobile | /products/glan-thompson-polarizing-prisms | 4220px | 53.5% | ❌ **BROKEN** |
| mobile | /products/glan-thompson-polarizing-prisms | 5064px | 36.9% | ❌ **BROKEN** |
| mobile | /products/glan-thompson-polarizing-prisms | 5908px | 24.8% | ❌ **BROKEN** |
| mobile | /privacy | 0px | 10.0% | ❌ **BROKEN** |
| mobile | /privacy | 844px | 9.9% | ✅ pass |
| mobile | /privacy | 1688px | 9.8% | ✅ pass |
| mobile | /privacy | 2532px | 10.3% | ❌ **BROKEN** |
| mobile | /privacy | 3376px | 9.7% | ✅ pass |
| mobile | /privacy | 4220px | 7.3% | ✅ pass |
| mobile | /privacy | 5064px | 10.6% | ❌ **BROKEN** |
| mobile | /privacy | 5908px | 9.8% | ✅ pass |
| mobile | /privacy | 6752px | 10.9% | ❌ **BROKEN** |
| mobile | /privacy | 7596px | 9.3% | ✅ pass |
| mobile | /privacy | 8440px | 10.0% | ❌ **BROKEN** |
| mobile | /privacy | 9284px | 9.3% | ✅ pass |

## Summary

- Screens checked: **73**
- Passed: **47**
- Broken: **26**

## Broken screens — pictures saved for each

- desktop https://halle-dev.webflow.io/ at 0px — 15.8% different, 18 bad regions → `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y0_browser.png` vs `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y0_capture.png`
- desktop https://halle-dev.webflow.io/ at 2700px — 25.3% different, 31 bad regions → `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y2700_browser.png` vs `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y2700_capture.png`
- desktop https://halle-dev.webflow.io/ at 3600px — 21.4% different, 36 bad regions → `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y3600_browser.png` vs `audit-out-html2canvas/desktop_https_halle_dev_webflow_io__y3600_capture.png`
- desktop https://halle-dev.webflow.io/products-selection/polarizers at 900px — 30.8% different, 42 bad regions → `audit-out-html2canvas/desktop_webflow_io_products_selection_polarizers_y900_browser.png` vs `audit-out-html2canvas/desktop_webflow_io_products_selection_polarizers_y900_capture.png`
- desktop https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 1800px — 53.0% different, 60 bad regions → `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y1800_browser.png` vs `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y1800_capture.png`
- desktop https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 2700px — 51.5% different, 60 bad regions → `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y2700_browser.png` vs `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y2700_capture.png`
- desktop https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 3600px — 36.4% different, 42 bad regions → `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y3600_browser.png` vs `audit-out-html2canvas/desktop_products_glan_thompson_polarizing_prisms_y3600_capture.png`
- desktop https://halle-dev.webflow.io/privacy at 9000px — 12.9% different, 24 bad regions → `audit-out-html2canvas/desktop_https_halle_dev_webflow_io_privacy_y9000_browser.png` vs `audit-out-html2canvas/desktop_https_halle_dev_webflow_io_privacy_y9000_capture.png`
- desktop https://halle-dev.webflow.io/privacy at 9900px — 11.5% different, 17 bad regions → `audit-out-html2canvas/desktop_https_halle_dev_webflow_io_privacy_y9900_browser.png` vs `audit-out-html2canvas/desktop_https_halle_dev_webflow_io_privacy_y9900_capture.png`
- mobile https://halle-dev.webflow.io/ at 0px — 32.5% different, 39 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y0_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y0_capture.png`
- mobile https://halle-dev.webflow.io/ at 1688px — 12.4% different, 16 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y1688_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y1688_capture.png`
- mobile https://halle-dev.webflow.io/ at 4220px — 15.9% different, 14 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y4220_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y4220_capture.png`
- mobile https://halle-dev.webflow.io/ at 5064px — 35.7% different, 50 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y5064_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y5064_capture.png`
- mobile https://halle-dev.webflow.io/ at 5908px — 26.1% different, 49 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y5908_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io__y5908_capture.png`
- mobile https://halle-dev.webflow.io/products-selection/polarizers at 1688px — 49.7% different, 60 bad regions → `audit-out-html2canvas/mobile_webflow_io_products_selection_polarizers_y1688_browser.png` vs `audit-out-html2canvas/mobile_webflow_io_products_selection_polarizers_y1688_capture.png`
- mobile https://halle-dev.webflow.io/products-selection/polarizers at 2532px — 34.1% different, 30 bad regions → `audit-out-html2canvas/mobile_webflow_io_products_selection_polarizers_y2532_browser.png` vs `audit-out-html2canvas/mobile_webflow_io_products_selection_polarizers_y2532_capture.png`
- mobile https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 2532px — 19.7% different, 20 bad regions → `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y2532_browser.png` vs `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y2532_capture.png`
- mobile https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 3376px — 47.7% different, 59 bad regions → `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y3376_browser.png` vs `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y3376_capture.png`
- mobile https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 4220px — 53.5% different, 59 bad regions → `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y4220_browser.png` vs `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y4220_capture.png`
- mobile https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 5064px — 36.9% different, 53 bad regions → `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y5064_browser.png` vs `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y5064_capture.png`
- mobile https://halle-dev.webflow.io/products/glan-thompson-polarizing-prisms at 5908px — 24.8% different, 39 bad regions → `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y5908_browser.png` vs `audit-out-html2canvas/mobile_products_glan_thompson_polarizing_prisms_y5908_capture.png`
- mobile https://halle-dev.webflow.io/privacy at 0px — 10.0% different, 0 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y0_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y0_capture.png`
- mobile https://halle-dev.webflow.io/privacy at 2532px — 10.3% different, 0 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y2532_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y2532_capture.png`
- mobile https://halle-dev.webflow.io/privacy at 5064px — 10.6% different, 0 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y5064_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y5064_capture.png`
- mobile https://halle-dev.webflow.io/privacy at 6752px — 10.9% different, 0 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y6752_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y6752_capture.png`
- mobile https://halle-dev.webflow.io/privacy at 8440px — 10.0% different, 0 bad regions → `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y8440_browser.png` vs `audit-out-html2canvas/mobile_https_halle_dev_webflow_io_privacy_y8440_capture.png`
