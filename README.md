[README.md](https://github.com/user-attachments/files/33201877/README.md)
# Bikko MH — standalone Netlify package

This package contains the Bikko MH frontend and its own Netlify Function API.
It stores stock and sales in a Netlify Blobs store belonging to this Netlify
site. It does not call the Replit API, use the Replit PostgreSQL database, or
require `BIKKO_API_ORIGIN`.

## Data separation

- This Netlify site starts with its own empty stock and sales state.
- No Replit database records are copied or shared.
- The existing frontend may import legacy data saved in the current browser's
  local storage the first time that browser signs in. That import goes only to
  this Netlify site's store; it does not contact Replit.
- Strongly consistent reads and conditional writes protect stock and sales when
  several devices use this same Netlify site.
- Redeploying the code does not replace the Netlify Blobs store.

## Configure before publishing

In this Netlify site's environment variable settings, set:

- `BIKKO_MH_ACCESS_CODE` — the shared sign-in code for staff.
- `SESSION_SECRET` — a long, random signing secret for the 12-hour session
  cookie.

Keep both values private. They are read only by the Netlify Function and are
not included in this package or its frontend files. Do not add
`BIKKO_API_ORIGIN`; this package has no API proxy.

## Deploy

Use a Git-connected Netlify site or the Netlify CLI so Netlify deploys both the
static frontend and the function. A static-only drag-and-drop upload does not
include the API.

From this folder, after the site is linked and its environment variables are
set, run:

```sh
npm install
netlify deploy --prod
```

For a draft preview, run `netlify deploy` without `--prod`. The included
`netlify.toml` publishes `site/`, bundles
`netlify/functions/bikko-mh-api.mjs`, routes `/api/*` to that function, and
serves the frontend for client-side routes. For a Git-connected site, set this
package folder as the Netlify base directory. Netlify Functions automatically
provide the site context used by Netlify Blobs.

## Run the backend tests

With Node.js installed, run:

```sh
npm test
```

The tests use an in-memory stand-in for Netlify Blobs; they verify auth,
revision conflicts, stock adjustment, concurrent sales, and import behavior.
They do not connect to a live Netlify site.
