---
name: testing-browser-compatibility
description: |
  Test across multiple browsers and devices for cross-browser compatibility.
  Use when ensuring cross-browser or device compatibility with BrowserStack, Sauce Labs, LambdaTest, or Kobiton.
  Trigger with phrases like "test browser compatibility", "check cross-browser", "validate on browsers", "test on real devices", "kobiton test".

allowed-tools: Read, Write, Edit, Grep, Glob, Bash(npx playwright:*), Bash(npm:*), Bash(curl:*)
version: 2.0.0
author: Jeremy Longshore <jeremy@intentsolutions.io>
license: MIT
compatible-with: claude-code, codex, openclaw
tags: [testing, cross-browser, mobile-testing, real-device, cloud-testing]
---
# Browser Compatibility Tester

## Overview

Test web applications across multiple browsers, rendering engines, and real devices. Validates CSS rendering, JavaScript API support, layout consistency, and interactive behavior across Chromium (Chrome, Edge), Gecko (Firefox), and WebKit (Safari) -- locally with Playwright or on real devices via BrowserStack, Sauce Labs, LambdaTest, or Kobiton.

## Prerequisites

- Playwright installed: `npx playwright install --with-deps`
- Application running at a test URL
- For cloud testing: provider credentials in environment variables (see `${CLAUDE_SKILL_DIR}/references/cloud-providers.md`)

## Instructions

### Mode 1: Local Testing (Playwright)

Default mode. Zero cloud accounts needed.

1. Define the browser matrix from project `browserslist` config or use defaults:
   - Desktop: Chrome (latest), Firefox (latest), Safari (latest), Edge (latest)
   - Mobile: iPhone 14 (WebKit), Pixel 7 (Chromium)
   - Viewports: 375px, 768px, 1280px, 1920px

2. Scan the codebase for compatibility risks:
   - Grep for modern JS APIs: `IntersectionObserver`, `ResizeObserver`, `structuredClone`, `Array.at()`, `Promise.withResolvers()`
   - Grep for modern CSS: `container queries`, `has()`, `@layer`, `subgrid`, `color-mix()`, `@starting-style`
   - Cross-reference against caniuse data for the target matrix
   - Flag usage without polyfills or `@supports` feature detection

3. Write compatibility-focused tests:
   - Layout tests: key elements render at expected positions/sizes per viewport
   - CSS feature tests: modern features degrade gracefully behind `@supports`
   - JS API tests: polyfills load in older browsers
   - Form input tests: date, color, range inputs across engines
   - Media tests: video/audio, WebP/AVIF support
   - Accessibility: run axe-core per browser (`@axe-core/playwright`)

4. Execute and capture results:
   - `npx playwright test --project=chromium --project=firefox --project=webkit`
   - Screenshots per browser for visual comparison
   - Video traces for failing tests

### Mode 2: Cloud Real-Device Testing

Route here when the user needs real physical devices, broader OS coverage, or carrier network conditions that Playwright emulation cannot replicate. Read `${CLAUDE_SKILL_DIR}/references/cloud-providers.md` for full auth, API, and capabilities details.

**Provider selection:**

| Need | Provider |
|------|----------|
| Broadest browser/OS matrix (3,000+ combos) | BrowserStack |
| Enterprise CI/CD, Sauce Connect tunnel | Sauce Labs |
| Auto-healing selectors, smart testing | LambdaTest |
| Real physical devices, scriptless automation | **Kobiton** |

Never hardcode credentials. Set provider env vars (`BROWSERSTACK_USERNAME`/`ACCESS_KEY`, `SAUCE_USERNAME`/`ACCESS_KEY`, `LT_USERNAME`/`ACCESS_KEY`, `KOBITON_USERNAME`/`API_KEY`). See `${CLAUDE_SKILL_DIR}/references/cloud-providers.md` for full auth details.

1. Verify credentials are set for the chosen provider
2. Query available devices/browsers via provider API
3. Configure WebDriver or Appium capabilities
4. Execute tests against cloud grid
5. Retrieve session artifacts (screenshots, video, logs, network HAR)
6. Aggregate results into compatibility report

For CI/CD integration patterns, see `${CLAUDE_SKILL_DIR}/references/ci-cd-integration.md`.

### Browser-Specific Checks

- **Safari**: date input formatting, scroll behavior, backdrop-filter, PWA manifest, position: sticky in overflow
- **Firefox**: scrollbar styling, gap in flexbox, subpixel rendering, print stylesheets
- **Mobile**: touch events, viewport meta, safe area insets, virtual keyboard resize, 300ms tap delay

See `${CLAUDE_SKILL_DIR}/references/device-matrix.md` for pre-built matrices (top 10 browsers, mobile-first, enterprise, Kobiton real-device).

## Examples

**Playwright multi-browser config:**
```typescript
import { defineConfig, devices } from '@playwright/test';
export default defineConfig({
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 7'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 14'] } },
  ],
});
```

**Cross-browser layout test:**
```typescript
test('nav renders correctly across browsers', async ({ page }) => {
  await page.goto('/');
  const nav = page.locator('nav');
  await expect(nav).toBeVisible();
  const box = await nav.boundingBox();
  expect(box.width).toBeGreaterThan(300);
});
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|---------|
| WebKit fails, Chromium passes | CSS property unsupported in Safari | Add `-webkit-` prefix or `@supports` fallback |
| Date input renders differently | Browsers implement `<input type="date">` differently | Use custom date picker component |
| Test passes locally, fails on cloud | Real device rendering differs from emulation | Run critical paths on real devices for final validation |
| Kobiton device unavailable | Device in use or offline | Query `GET /v1/devices` for online devices; use `deviceGroup` capability |
| Cloud session timeout | Long test on slow device | Increase `sessionTimeout` capability; split into smaller test files |

## Output

- Playwright config with multi-browser projects
- Cross-browser test files in `tests/compatibility/`
- Compatibility matrix report (pass/fail per browser, viewport, device)
- Screenshots per browser for visual diff
- Unsupported API list with polyfill/fallback recommendations
- Cloud session URLs with video replay links (when using cloud providers)

## Resources

- Playwright browsers: https://playwright.dev/docs/browsers
- Can I Use: https://caniuse.com/
- BrowserStack Automate: https://www.browserstack.com/automate
- Sauce Labs: https://docs.saucelabs.com/
- LambdaTest: https://www.lambdatest.com/support/docs/
- Kobiton API: https://api.kobiton.com/docs/
- MDN Compat Data: https://github.com/mdn/browser-compat-data
