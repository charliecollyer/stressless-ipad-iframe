# Project context

Static HTML files, no build step, no framework, no dependencies, deployed to Vercel as a static site from this repo. The root URL (`/`) is not mapped — there's no `index.html`, so always link to the full path (`/kiosk-home.html`, `/demo.html`, `/iframe-test.html`).

- `iframe-test.html` — throwaway diagnostic tool. Tests whether a Stressless product/configurator page can be loaded inside an iframe, i.e. whether it's blocked by `X-Frame-Options`/CSP. Keep edits minimal; avoid adding tooling.
- `kiosk-home.html` — the actual kiosk home screen (landscape iPad, PWA-style). Rebuilt from the Claude Design "Direction A · Editorial" mockup as plain HTML/CSS/JS (no React — the prototype's CDN React/Babel was deliberately not carried over). Dark Stressless® Adam hero tile (photo fades in from the right, `adam.jpg`), three secondary tiles, and a prize-draw + newsletter card. On-brand styling from the Stressless Brand Guidelines 2025 (Source Sans 3 stands in for Myriad Pro). See `README.md` for tile behaviour and the Marketo placeholder note.
- `demo.html` — a desktop "virtual iPad" page: a scaled landscape iPad mock framing `kiosk-home.html` in an iframe, for screen-sharing the kiosk in meetings (`/demo.html`).
- `adam.jpg` — hero product photo used by `kiosk-home.html`.

Avoid adding tooling (bundlers, frameworks, package.json) unless explicitly asked — pages should stay single-file static HTML.
