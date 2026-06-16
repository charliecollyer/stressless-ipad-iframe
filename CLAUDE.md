# Project context

Two static HTML files, no build step, no framework, no dependencies, deployed to Vercel as a static site from this repo. The root URL (`/`) is not mapped — there's no `index.html`, so always link to the full path (`/kiosk-home.html`, `/iframe-test.html`).

- `iframe-test.html` — throwaway diagnostic tool. Tests whether a Stressless product/configurator page can be loaded inside an iframe, i.e. whether it's blocked by `X-Frame-Options`/CSP. Keep edits minimal; avoid adding tooling.
- `kiosk-home.html` — the actual kiosk home screen design (PWA-style). This is real, evolving work, not a throwaway — tiles linking to iframed Stressless pages, two embedded Marketo forms (modal + inline newsletter), on-brand styling from the Stressless brand guidelines PDF. See `README.md` for full details on tile behaviour, the Marketo CSS fixes applied, and known/accepted limitations (locale-dependent placeholder text).

Avoid adding tooling (bundlers, frameworks, package.json) to either file unless explicitly asked — both should stay single-file static HTML.
