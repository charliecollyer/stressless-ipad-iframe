# Stressless iPad Iframe Test

Static pages, no build step, no framework, no dependencies:

- `kiosk-home.html` — the kiosk home screen (landscape iPad, PWA-style), with brand styling pulled from `Stressless Brand Guidelines 2025 Vol. 1.pdf`.
- `demo.html` — a desktop "virtual iPad" that frames `kiosk-home.html` for screen-sharing.
- `iframe-test.html` — diagnostic test to check whether a Stressless product/configurator page can be loaded inside an `<iframe>`, or whether it's blocked by `X-Frame-Options` / `Content-Security-Policy`.
- `adam.jpg` — hero product photo used by `kiosk-home.html`.

**Live URLs:**
- https://stressless-ipad-iframe.vercel.app/kiosk-home.html
- https://stressless-ipad-iframe.vercel.app/demo.html
- https://stressless-ipad-iframe.vercel.app/iframe-test.html

## `kiosk-home.html`

Landscape iPad kiosk home screen, rebuilt from the Claude Design "Direction A · Editorial" mockup as plain HTML/CSS/JS (no React). One screen that fills the viewport:

- **Status bar + header** — faux iOS status bar, the Stressless® wordmark + "A place that moves you" claim, and a "Studio kiosk · Welcome" greeting.
- **Hero tile** (left) — dark Slate Anthracite tile featuring the model beside the iPad (Stressless® Adam). `adam.jpg` sits on the right half, flipped, gradient-masked, and fades in on load. "Explore Adam" pill CTA.
- **Three secondary tiles** (right) — Order a catalogue (Oak), Order your samples (Willow), Design your own (Glacier).
- **Prize-draw + newsletter card** (bottom) — competition copy plus a name / surname / email / GDPR-consent signup form.

When a tile opens an external page, it loads full-screen in an iframe and a circular **Home** button appears bottom-right to return to the kiosk.

**Tile behaviour:**
| Tile / control | Behaviour |
|---|---|
| Hero tile / "Explore Adam" | Loads the Stressless Adam product page (`ADAM_PAGE_URL`) full-screen in an iframe. |
| Design your own | Loads the Stressless Adam configurator (`CONFIGURATOR_URL`) full-screen in an iframe. |
| Order a catalogue | Placeholder — shows a "coming soon" modal (no destination yet). |
| Order your samples | Placeholder — shows a "coming soon" modal (was a Marketo form previously; parked for now). |
| Newsletter form | **Placeholder** — validates locally (first name, surname, real email, consent) and shows a "You're in the draw" confirmation. Not yet wired to Marketo. |

`ADAM_PAGE_URL` and `CONFIGURATOR_URL` are JS constants near the top of the `<script>` block — edit there to repoint either tile.

### Marketo: parked for now

The previous build embedded live Marketo forms (newsletter **523** and samples **1378**). Those were **removed** when the page was rebuilt from the new design — the in-page newsletter form is a local placeholder. When Marketo is reintegrated, the form-settings/locale notes and the per-form Custom-CSS fixes from the earlier build are recoverable from git history (commit `3ccb9ab` and the README at that revision). Known limitation from before, still relevant: Marketo forms key their headline/consent copy off the `stressless.com` page URL, so a bare embed on this kiosk URL shows placeholder copy — the fix is on the Marketo side (hardcode the GB locale).

### Open question: can Stressless pages be framed?

The Adam page and configurator tiles depend on `stressless.com` / `shop.stressless.com` allowing themselves to be loaded in an `<iframe>`. If they send `X-Frame-Options: DENY`/`SAMEORIGIN` or a restrictive CSP `frame-ancestors`, the tile will open a blank frame. Use `iframe-test.html` (below) to confirm framing works on the target iPad before relying on it.

## `demo.html`

A desktop "virtual iPad" for screen-sharing the kiosk in meetings — open `/demo.html` and you get a landscape iPad mock (1194 × 834 screen) with `kiosk-home.html` running inside it in an iframe. A small script scales the whole device down to fit the browser window (never up past 1:1), so it looks right on any laptop. No interaction wiring of its own — it's purely a frame around the real kiosk page.

## `iframe-test.html`

Standalone diagnostic, unrelated to `kiosk-home.html`. Tests whether a given URL can be framed at all.

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

Static HTML, no build step. Deployed to Vercel directly from this repo (`vercel --prod --yes`). Note the root URL (`/`) is not mapped — there's no `index.html`, so always link to the full path (`/kiosk-home.html`, `/iframe-test.html`).

## Local testing without deploying

No server-side behaviour, so any static file server works, e.g.:
```
python3 -m http.server 8910
```
then open `http://localhost:8910/kiosk-home.html`. Useful for fast iteration with a headless browser (Playwright) before pushing a Vercel deploy.
