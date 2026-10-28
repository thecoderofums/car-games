# Car Racing Game — repository notes

## What this repo is

A **single static page**: the file `car` (HTML + inline CSS + inline JS, an arrow-key
car racing game). There is no package manager, no build step, no framework, no
database and no external service — so there is nothing to install, migrate or seed,
and no credentials are required.

Note the file has **no `.html` extension**; it must be served as `index.html`.

## Running it

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

The `web` service (plain `python:3.12-slim`, no app image is ever built or pulled)
bind-mounts the checkout at `/app` and serves the source itself:

- the extension-less `car` file is copied to `/srv/site/index.html` **inside the
  container** (the repo is never written to) every second, so editing `car` reaches
  the preview in about a second without a restart;
- `python3 -m http.server 3000 --bind 0.0.0.0 --directory /srv/site` serves it;
- `python:3.12-slim` has no curl/wget, so the compose healthcheck probes `/` with a
  `python3 -c` urllib one-liner instead.

`BASE44_PREVIEW_MODE` is passed through to the service but nothing reads it: the
static page has no host/origin allowlist to extend (python's http.server does not
check `Host`), so no sandbox-only code override exists here.

## Verifying it works

```bash
curl -s -o /dev/null -w '%{http_code} %{content_type}\n' http://localhost:3000/   # 200 text/html
docker compose -f docker-compose.base44.yml ps                                    # web: healthy
```

There is no test suite. Behaviour worth checking in the browser: the red car is
visible inside the grey track, `ArrowUp`/`ArrowDown` change speed, `ArrowLeft`/
`ArrowRight` turn it and the car stays inside the game area.
