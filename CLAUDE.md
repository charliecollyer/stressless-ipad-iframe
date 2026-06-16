# Project context

Single static HTML file (`iframe-test.html`) that tests whether a Stressless product/configurator page can be loaded inside an iframe — used to check if the page is blocked by `X-Frame-Options`/CSP before embedding it in an iPad kiosk app.

No build step, no framework, no dependencies. Treat this as a throwaway diagnostic tool, not a product — keep edits minimal and avoid adding tooling (bundlers, frameworks, package.json) unless explicitly asked.

Deployed to Vercel as a static site from this repo. The root URL (`/`) is not mapped to `iframe-test.html` — there's no `index.html`, so always link to `/iframe-test.html` directly.
