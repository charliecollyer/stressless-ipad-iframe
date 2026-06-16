# Stressless iPad Iframe Test

Two static pages, no build step, no framework, no dependencies:

- `iframe-test.html` — diagnostic test to check whether a Stressless product/configurator page can be loaded inside an `<iframe>`, or whether it's blocked by `X-Frame-Options` / `Content-Security-Policy`.
- `kiosk-home.html` — the actual kiosk home screen design (PWA-style), with brand styling pulled from `Stressless Brand Guidelines 2025 Vol. 1.pdf`.

**Live URLs:**
- https://stressless-ipad-iframe.vercel.app/iframe-test.html
- https://stressless-ipad-iframe.vercel.app/kiosk-home.html

## `kiosk-home.html`

Stressless logo header + four tiles + an inline newsletter signup, all on one screen. A "Home" button is fixed top-left at all times and returns to the tile screen from anywhere.

**Tiles:**
| Tile | Behaviour |
|---|---|
| Request Samples | Opens a modal with embedded Marketo form **1378**, submits directly to Marketo. On success, shows a "Thanks!" message and auto-returns home after 3s. |
| Product Configurator | Loads the Stressless Adam configurator (`CONFIGURATOR_URL`) full-screen in an iframe. |
| Sample Iframe | Loads the real `requestasample` landing page (`SAMPLE_PAGE_URL`) full-screen in an iframe — see "Open question" below. |
| Stressless Adam | Loads the Stressless Adam product page (`ADAM_PAGE_URL`) full-screen in an iframe. |

All four URLs are set as JS constants near the top of the `<script>` block — edit there to repoint any tile.

**Newsletter form:** Marketo form **523** is embedded directly in the page (not a modal), below the tiles, via `MktoForms2.loadForm("//lp.stressless.com", "853-XWI-508", 523)`.

### Open question: locale-dependent form content

Both Marketo forms (and the "Sample Iframe" page) pull some content from the **page URL** at runtime — e.g. the sample-request form's country/locale dropdown, and form 523's headline/subheader/consent text are populated by scripts that key off `stressless.com` URL context. On this kiosk PWA the URL is never `stressless.com`, so:
- Form 523 currently shows literal placeholder copy ("Newsletter Headline", "Newsletter SubHeader", "Consent Text here") instead of real text.
- The "Sample Iframe" tile exists specifically to test whether loading the *real* `requestasample` page in an iframe (rather than embedding the bare form) lets those scripts pick up the right context from the iframe's own URL.

This is a known, accepted limitation for now — not a bug to chase. If real copy is needed without the "Sample Iframe" workaround, the fix is on the Marketo side (hardcode the GB locale, or have the URL-detection script also check the iframe context), not in this file.

### Newsletter form styling — what was fixed and why

The base Marketo form renders almost unstyled by default; the "real" design comes from three CSS layers Marketo's `forms2.min.js` injects automatically: base `forms2.css`, theme `forms2-theme-simple.css`, and the per-form **Custom CSS** (`ThemeStyleOverride`, copied from Marketo's form-settings UI — see below). That custom CSS is the source of truth for the intended look (white card, centered stacked fields, navy uppercase pill button, cream `<section>` background). It mostly works once embedded, but had to be patched locally for problems that exist independently of our embed:

1. **`section:has(.mktoForm)` rule needs an actual `<section>`.** The custom CSS keys the cream-card background/padding/margin off a `<section>` ancestor. The newsletter wrapper was changed from `<div id="newsletter-section">` to `<section id="newsletter-section">` so this rule actually matches.
2. **`.mktoFormCol` margin overflows a narrow container.** Marketo's CSS sets `width:100%` *and* `margin:20px` (all four sides) on `.mktoFormCol` — fine in a wide page, but overflows a ~480px-wide embed. Patched to `margin: 20px 0` (vertical only). Note this is literally what Marketo's own `@media (max-width:1024px)` block already does — we just apply it unconditionally since our embed is always narrow.
3. **Invalid Marketo selector for full-width inputs.** Their custom CSS has `.mktoFieldWrap input:not[type:checkbox]{width:100%}` — missing brackets, so it's invalid CSS and never applies in any browser. Replaced with the valid equivalent: `input:not([type="checkbox"])`.
4. **Marketo's form JS defaults to two-column "Layout Left" mode.** At runtime, Marketo's JS sets an inline `width:100px` on `.mktoLabel` and `width:150px` on the input — a side-by-side label/field layout — which conflicts with the stacked, centered layout the custom CSS is designed for. Forced `.mktoLabel { width: 100% !important; float: none !important; }` so the label fills its row and the field wraps below it (the inline styles otherwise out-rank any non-`!important` CSS, including Marketo's own).
5. **Button color/specificity clash inside Marketo's own CSS.** The "Simple" theme's default button stylesheet (`buttonStyleId 11`) uses `.mktoForm .mktoButtonWrap.mktoSimple .mktoButton` (4 classes → high specificity) for a default green gradient button. The custom CSS's intended navy pill button uses `.mktoForm button` (1 class) — lower specificity, so it loses on color/border/padding even though it wins on shape (`border-radius`, `text-transform`) since those properties aren't contested. Patched `#newsletter-section .mktoButton` (ID-scoped, wins outright) to apply the custom CSS's intended navy (`#141923`, darker navy `#05143C` on hover/active).

All of these overrides live in the `<style>` block in `kiosk-home.html`, scoped under `#newsletter-section`, each with a comment explaining which Marketo bug/conflict it's patching. **None of them override Marketo's *design intent*** — they only fix things that are broken or context-dependent in Marketo's own CSS/JS. If Marketo's form settings change in the future (new theme, edited custom CSS, layout setting changed from "Left" to "Above"), re-check whether these patches are still needed.

The exact custom CSS Marketo serves for both forms can be re-fetched any time via:
```
curl -s "https://lp.stressless.com/index.php/form/getForm?munchkinId=853-XWI-508&form=523"
```
(swap `523` for `1378` for the samples form). Look at the `ThemeStyleOverride` field in the JSON response.

### Known non-issues (already diagnosed, not bugs)

- Console error `$ is not defined` — Marketo's form embed includes a jQuery-loader `<script>` field that races with other inline scripts; cosmetic only, doesn't affect form function. Not currently patched (would need preloading jQuery before `forms2.min.js`).
- Console error on `atob` (reCAPTCHA/translate helper script) — unrelated third-party Marketo script error, no visible effect on the form.
- Inline `width: 2049px`-style values Marketo's JS sets on `.mktoForm` — harmless; the custom CSS's `width:100% !important` always wins over a non-`!important` inline style.

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
