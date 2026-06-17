# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static HTML files, no build step, no framework, no dependencies, deployed to Vercel as a static site from this repo. There are no tests, no linter, and no package manifest — keep it that way. Avoid adding tooling (bundlers, frameworks, `package.json`) unless explicitly asked; pages should stay single-file static HTML.

The root URL (`/`) is not mapped — there's no `index.html`, so always link to the full path (`/kiosk-home.html`, `/demo.html`, `/iframe-test.html`).

## Commands

- **Preview locally:** `python3 -m http.server 8910` then open `http://localhost:8910/kiosk-home.html`. No server-side behaviour, so any static file server works.
- **Verify visual changes:** Playwright + the installed Chrome are available for headless screenshots/interaction checks against the local server — use this to confirm HTML/CSS/JS changes before deploying (there's no test suite to run).
- **Deploy to production:** `vercel --prod --yes` (project is already linked via `.vercel/`; aliases to `stressless-ipad-iframe.vercel.app`). Deploying is what makes `vercel.json` headers and `robots.txt` take effect — they are not visible on the local file server.

## Files

- `kiosk-home.html` — the kiosk home screen (landscape iPad, PWA-style). Rebuilt from the Claude Design "Direction A · Editorial" mockup as plain HTML/CSS/JS (no React — the prototype's CDN React/Babel was deliberately not carried over). Dark Stressless® Adam hero tile (photo fades in from the right, `adam.jpg`), three secondary tiles, and a prize-draw + newsletter card. On-brand styling from the Stressless Brand Guidelines 2025 (Source Sans 3 stands in for Myriad Pro).
- `demo.html` — a desktop "virtual iPad" page: a scaled landscape iPad mock framing `kiosk-home.html` in an iframe, for screen-sharing the kiosk in meetings.
- `iframe-test.html` — throwaway diagnostic tool. Tests whether a Stressless product/configurator page can be loaded inside an iframe, i.e. whether it's blocked by `X-Frame-Options`/CSP. Keep edits minimal.
- `adam.jpg` — hero product photo used by `kiosk-home.html`.
- `vercel.json`, `robots.txt` — search-engine suppression (see below).

## Architecture notes

- **Tile navigation in `kiosk-home.html` is a single-page, in-place iframe swap, not real routing.** Tiles that open an external Stressless page set the `src` of a hidden full-screen `#iframe-view` overlay and reveal a circular Home button (bottom-right) that clears it. Destination URLs (`ADAM_PAGE_URL`, `CONFIGURATOR_URL`) are JS constants at the top of the `<script>` block — edit there to repoint a tile. This whole approach hinges on the target pages allowing themselves to be framed; `iframe-test.html` exists to check that on the target iPad.
- **The newsletter form is a local placeholder, not a real integration.** It validates client-side and shows a confirmation state. A previous build embedded live Marketo forms (523/1378); those were removed in the rebuild and are recoverable from git history if reintegrated. Marketo forms key their copy off the `stressless.com` page URL, so a bare embed on this kiosk URL shows placeholder copy — that fix is on the Marketo side.
- **Search-engine suppression is three layers:** `X-Robots-Tag: noindex, nofollow` on every path (`vercel.json`), `robots.txt` disallowing all crawlers, and a `noindex` `<meta>` in each page. This is **not** access control — anyone with the direct URL can still open the pages. Use Vercel Deployment Protection if a real gate is needed.

See `README.md` for fuller detail on tile behaviour, the Marketo history, and the `iframe-test.html` testing procedure.
