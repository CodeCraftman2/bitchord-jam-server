# BitChord Jam Server — self-host helper

A small, self-contained setup for running your own **Listen Together** server
for the [BitChord](https://github.com/kushagrasinghx/BitChord) music app: Go,
one WebSocket per device, no database, no accounts. Pick a host below, deploy,
and point the app at your URL.

## Quick start

```bash
go run .            # http://localhost:8000
go test ./...
curl localhost:8000/healthz
python tools/jam_probe.py http://localhost:8000   # smoke-test a running server
```

## Deploy (pick one)

### Fly.io (Dockerfile)
```bash
cp fly.toml.example fly.toml      # fly.toml is git-ignored — keep it local
# edit `app` to a unique name, and primary_region to the closest one
fly launch --no-deploy --copy-config
fly deploy
```
Your server is `https://<app>.fly.dev`. Keep exactly **1 machine** — parties live in memory.

### Render
Use `render.yaml` (New → Blueprint), or a Web Service with build `go build -o server .`,
start `./server`, health check `/healthz`, 1 instance.

### Any Ubuntu VM (Oracle Always Free, etc.)
Point a DNS A record at the VM, then:
```bash
sudo DOMAIN=jam.example.com bash deploy/setup.sh
```
Installs Go, a systemd service and Caddy (automatic HTTPS + WebSocket proxy).

### Docker
```bash
docker build -t jam-server . && docker run -p 8000:8000 -e JAM_TRUST_PROXY=false jam-server
```

## Connecting the app
Build BitChord with your URL, e.g. in its `local.properties`:
```
LISTEN_TOGETHER_SERVER=https://<your-host>
```

## Is it safe? Does it expose my URL?
- **This repo contains no real URL, token or key.** `fly.toml` (your app name =
  your URL) is git-ignored; only `fly.toml.example` is committed. Don't commit
  `.env` files or `fly secrets` output.
- The server URL is public by nature — every phone in a party must reach it —
  so protect it with the built-in limits instead of hiding it: `JAM_MAX_PARTIES`,
  `JAM_CREATE_RATE_PER_MINUTE`, `JAM_MAX_MEMBERS`, `JAM_REQUEST_MAX_BYTES`,
  `JAM_ALLOWED_ORIGINS` (see `docs/PROTOCOL.md`).
- Set `JAM_TRUST_PROXY=true` **only** behind a proxy that sets `X-Forwarded-For`
  (Fly, Render, Caddy). On a bare port, leave it `false` or clients can spoof IPs.
- Don't share the URL in screenshots of your app settings if you'd rather not
  have strangers create parties on it.

## Layout
`main.go` server · `party/ hub/ presence/ protocol/ codes/ clock/ config/` internals ·
`deploy/` VM scripts · `tools/jam_probe.py` smoke test · `docs/PROTOCOL.md` full API and sync design.

## License / credit
GPL-3.0. The server code comes from the BitChord project by Kushagra Singh and
contributors (https://github.com/kushagrasinghx/BitChord); this repo repackages
it as a deployment helper. See `LICENSE`.
