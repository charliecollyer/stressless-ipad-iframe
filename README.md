# Stressless iPad Iframe Test

Two static pages:

- `iframe-test.html` — diagnostic test to check whether a Stressless product/configurator page can be loaded inside an `<iframe>`, or whether it's blocked by `X-Frame-Options` / `Content-Security-Policy`.
- `kiosk-home.html` — draft kiosk home screen with two tiles: "Request Samples" (opens a modal with an embedded Marketo form, submits directly to Marketo) and "Product Configurator" (opens the configurator full-screen in an iframe). A "Home" button is always visible and returns to the tile screen.

**Live URLs:**
- https://stressless-ipad-iframe.vercel.app/iframe-test.html
- https://stressless-ipad-iframe.vercel.app/kiosk-home.html

## How to test

1. Open the live URL on the target device (e.g. iPad).
2. Watch the status bar at the top:
   - Turns green ("Load event fired") if the iframe received a load event — but this can still fire on a blocked/blank page.
   - Turns red after 6s if no load event fired at all — usually means framing is blocked.
3. Visually check the checkered area below: if the real Stressless page renders, framing works. If it stays blank/checkered, it's blocked.
4. If content loads, manually verify:
   - Interacting with a configurator option responds correctly.
   - Add an item to cart, then click "Reload current" — check whether the cart count persists (tests whether session/cookies survive inside the iframe on this device).
5. Use the URL field + "Load URL" button to test other Stressless pages without redeploying.

## Deployment

Static HTML, no build step. Deployed to Vercel directly from this repo. Note the root URL (`/`) is not mapped — use the full path `/iframe-test.html`.
