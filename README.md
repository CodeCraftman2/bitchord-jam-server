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

## Get your own server link (Fly.io, step by step)

You will end up with a link like `https://my-jam.fly.dev`. Fly needs a card on
signup; this tiny server fits in a very small machine.

1. **Install the Fly CLI and log in** (Windows PowerShell shown):
   ```powershell
   iwr https://fly.io/install.ps1 -useb | iex
   fly auth signup      # or: fly auth login
   ```
   macOS/Linux: `curl -L https://fly.io/install.sh | sh`
2. **Get this repo:**
   ```bash
   git clone https://github.com/CodeCraftman2/bitchord-jam-server
   cd bitchord-jam-server
   ```
3. **Create your config.** The app name becomes your link, so it must be unique
   across Fly and is public once you share the link:
   ```bash
   cp fly.toml.example fly.toml
   ```
   Open `fly.toml` and change `app = "..."` to e.g. `my-jam-1234`. Optionally
   change `primary_region` (`fly platform regions` lists them; `bom` = Mumbai).
4. **Create the app and deploy:**
   ```bash
   fly apps create my-jam-1234      # same name as in fly.toml
   fly deploy
   fly scale count 1                # parties live in memory: keep ONE machine
   ```
5. **Check it works.** Your link is `https://my-jam-1234.fly.dev`:
   ```bash
   curl https://my-jam-1234.fly.dev/healthz
   ```
   It should answer OK. (`fly status` and `fly logs` help if it doesn't.)
6. **Use it in the BitChord app.** Easiest: open **Listen Together**, type your
   link into the **server address box**, and start a party. No rebuild needed;
   the app remembers it. To bake it into your own build instead, add this line to
   BitChord's `local.properties`:
   ```
   LISTEN_TOGETHER_SERVER=https://my-jam-1234.fly.dev
   ```
7. **Update later:** pull changes and run `fly deploy` again. To remove
   everything: `fly apps destroy my-jam-1234` (this takes the link down).

## Other hosts

- **Render:** New, Blueprint, pick this repo (`render.yaml`). Link:
  `https://<service>.onrender.com`. The free plan sleeps when idle.
- **Any Ubuntu VM** (Oracle Always Free etc.): point a DNS A record at the VM, then
  `sudo DOMAIN=jam.example.com bash deploy/setup.sh` (installs Go, systemd, and
  Caddy for automatic HTTPS + WebSockets).
- **Docker:** `docker build -t jam-server . && docker run -p 8000:8000 jam-server`
- **Local test only:** `go run .` then `curl localhost:8000/healthz`

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
