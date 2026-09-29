# Canvas Headless POC

Drupal Canvas 1.12 + the experimental `canvas_headless` submodule, with a
Next.js frontend scaffolded from `drupal-canvas/headless-templates`.

```
drupal/    DDEV Drupal 11 site  -> https://canvas-headless-poc.ddev.site
frontend/  Next.js app          -> http://localhost:3000
```

## Start

```bash
cd ~/ddev/canvas-headless-poc/drupal && ddev start
cd ~/ddev/canvas-headless-poc/frontend && npm run dev
ddev drush -r ~/ddev/canvas-headless-poc/drupal uli /canvas   # admin login link
```

Use Chrome: over plain-http `localhost` the draft preview relies on partitioned
cookies, which Firefox/Safari refuse.

## What was set up

Drupal (`drupal/`):
- `drupal/canvas:^1.12`, `simple_oauth:^6.1`, `consumers`, `custom_elements`
- Enabled: `canvas`, `canvas_headless`, `canvas_oauth`
- Simple OAuth RSA keypair in `drupal/keys/` (local only, never commit)
- `canvas_headless.settings:frontends` = `http://localhost:3000`
  (Canvas UI: left rail > **Headless frontends**)
- OAuth consumer `cli` / `secret` (client_credentials, `canvas:js_component`,
  `canvas:asset_library`, `canvas:brand_kit`) for the Canvas CLI
- Demo: Canvas page 1 (`/page/1`) with a Hero component, published

Frontend (`frontend/`):
- `npx @drupal-canvas/create@latest frontend --template nextjs --site-url https://canvas-headless-poc.ddev.site --agents none`
- `npm run dev` exports `NODE_EXTRA_CA_CERTS` (mkcert root) so Node trusts the
  DDEV HTTPS cert when calling Drupal server-side.

## How it fits together

1. Canvas probes `http://localhost:3000/api/canvas/components` from the browser
   (401 = "Ready").
2. Opening the editor mints a single-use JWT assertion; the iframe loads
   `/api/draft?assertion=...`, which exchanges it at Drupal `/oauth/token` for a
   user-bound, read-only preview token (partitioned cookie).
3. Same load syncs the app's `components/*/component.yml` into Canvas as
   `js.*` components (stored on the frontend entry in config).
4. The catch-all route calls `GET /canvas/content-api?requestUri=/page/1` and
   renders the tree via `<CanvasComponentTree>`.

Add a component: `cd frontend && npx canvas scaffold --name my-hero`, set
`machineName: my-hero` and `status: true` in its `component.yml`, reload the
editor.

Docs: `drupal/web/modules/contrib/canvas/modules/canvas_headless/README.md` and
`drupal/web/modules/contrib/canvas/docs/user/src/content/docs/headless/`.
