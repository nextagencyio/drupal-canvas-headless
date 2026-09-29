# Canvas Headless POC

Drupal Canvas (experimental `canvas_headless` submodule, Canvas ≥ 1.12) with a
Next.js frontend rendering Canvas pages, including live draft preview in the
Canvas editor.

- Drupal: `drupal/` → https://canvas-headless-poc.ddev.site
- Frontend: `frontend/` → http://localhost:3000

Requires DDEV, mkcert, Node 22.19+ or 24.5+ (not 23), and **Chrome** (the
preview cookie is refused by Firefox/Safari over plain-http localhost).

## Setup

### 1. Drupal

```bash
cd drupal
ddev start
ddev composer install
ddev drush si standard --account-name=admin --account-pass=admin -y
ddev drush en canvas canvas_headless canvas_oauth -y
```

### 2. Simple OAuth keys (signs preview tokens; gitignored)

```bash
mkdir -p keys
openssl genrsa -out keys/private.key 2048
openssl rsa -in keys/private.key -pubout -out keys/public.key
chmod 600 keys/*.key
ddev drush cset simple_oauth.settings private_key /var/www/html/keys/private.key -y
ddev drush cset simple_oauth.settings public_key /var/www/html/keys/public.key -y
```

### 3. Register the frontend

```bash
ddev drush php:eval '\Drupal::configFactory()->getEditable("canvas_headless.settings")->set("frontends", [["url" => "http://localhost:3000", "components" => []]])->save();'
```

Or in Canvas: left rail → **Headless frontends** → **Add frontend**.

### 4. Frontend

```bash
cd ../frontend
cp .env.example .env   # set CANVAS_SITE_URL=https://canvas-headless-poc.ddev.site
npm install
npm run dev            # dev script sets NODE_EXTRA_CA_CERTS to the mkcert root
```

### 5. Try it

```bash
cd ../drupal && ddev drush uli /canvas
```

1. **Headless frontends** should show `http://localhost:3000` as **Ready**.
2. Page menu (top center) → **New** → **New page**. Opening the editor syncs
   the app's components into the library and loads the preview from Next.js.
3. Library → Hero → `⋯` → **Insert**, edit props, **Review changes** → **Publish**.
4. View it on the frontend: http://localhost:3000/page/1

If the preview area is blank grey, click the fit-to-view icon by the zoom menu.

## Optional: Canvas CLI credentials

Only needed for `npx canvas ...` commands (`CANVAS_CLIENT_ID`/`SECRET` in `.env`):

```bash
ddev drush php:eval '\Drupal::entityTypeManager()->getStorage("consumer")->create(["label" => "Canvas CLI", "client_id" => "cli", "secret" => "secret", "confidential" => TRUE, "grant_types" => ["client_credentials"], "scopes" => ["canvas_js_component", "canvas_asset_library", "canvas_brand_kit"], "user_id" => 1])->save();'
```

## Add a component

```bash
cd frontend && npx canvas scaffold --name my-hero
```

In its `component.yml` set `machineName: my-hero` and `status: true`, then
reload the Canvas editor.

## References

- Module README: `drupal/web/modules/contrib/canvas/modules/canvas_headless/README.md`
- Setup guide: `drupal/web/modules/contrib/canvas/docs/user/src/content/docs/headless/setup.mdx`
- Templates: https://github.com/drupal-canvas/headless-templates
