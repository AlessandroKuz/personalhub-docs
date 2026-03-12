# Hosting Options

Two approaches will be explored and compared hands-on before settling on one.

---

## Option A — Home Server (Dell Optiplex) + Cloudflare Tunnel

```
Internet
  → Cloudflare Edge (DDoS protection, CDN, TLS termination)
    → Cloudflare Tunnel (encrypted, outbound-only connection)
      → cloudflared daemon (Docker container on home server)
        → Caddy (internal reverse proxy)
          → web container (Django + Uvicorn)
```

**Key property:** `cloudflared` makes an *outbound* connection from your server to
Cloudflare. Your home IP never appears in DNS, no port forwarding is required, and
your router is never touched. This is a legitimate production pattern, not a workaround.

### Pros & Cons

| Pro | Con |
|---|---|
| €0/mo (electricity already paid) | Uptime depends on home power + ISP |
| Full hardware ownership | Single point of failure |
| DDoS fully absorbed by Cloudflare | Limited by home upload bandwidth |
| Home IP completely hidden | More initial setup steps |
| Excellent learning value | — |

### Setup Steps (when the time comes)

1. Create a Cloudflare account and add your domain
2. Go to Cloudflare Zero Trust → Tunnels → Create a tunnel
3. Download the tunnel token
4. Add `cloudflared` service to `docker-compose.prod.yml`:

```yaml
cloudflared:
  image: cloudflare/cloudflared:latest
  restart: always
  command: tunnel run
  environment:
    - TUNNEL_TOKEN=${CF_TUNNEL_TOKEN}
```

5. In the Cloudflare dashboard, configure the tunnel to route traffic to `web:8000`

---

## Option B — VPS + Caddy

```
Internet
  → Cloudflare (DNS + CDN, optional but recommended)
    → VPS public IP
      → Caddy (auto-HTTPS via Let's Encrypt)
        → web container (Django + Uvicorn)
```

For example [Hetzner](https://www.hetzner.com/cloud) CX23.

**Cost:** ~€4/mo fixed (2 vCPU, 4GB RAM, 40GB SSD).

### Pros & Cons

| Pro | Con |
|---|---|
| 99.9% uptime SLA | €4/mo fixed cost |
| Datacenter reliability | VPS IP is public (expected, manageable) |
| Simple, standard setup | — |
| Easy migration target | — |

### Caddyfile

```
yourdomain.com {
    reverse_proxy web:8000
}

docs.yourdomain.com {
    root * /srv/docs/site
    file_server
}
```

Caddy provisions and renews TLS certificates automatically via Let's Encrypt.
No certbot, no cron jobs, zero manual renewal.

---

## Docs Hosting

The MkDocs `site/` directory is pure static HTML. Two options:

**Option 1 — Same server (Caddy subdomain):**
Caddy serves `site/` directly as static files under `docs.yourdomain.com`.
Free, zero extra infrastructure.

**Option 2 — Cloudflare Pages:**
Push `site/` to a separate git branch; Cloudflare Pages deploys it automatically.
Free tier, globally distributed CDN, completely separate from your main server.

---

## Migration Between Options

Everything is Dockerized. Migration is:

```bash
# On the new host:
git clone https://www.github.com/AlessandroKuz/personalhub
cd personalhub
cp .env.example .env    # fill in prod values
docker compose -f docker-compose.prod.yml up -d
docker compose exec web uv run python manage.py migrate
```

The application has no host-specific state. Database lives in a named Docker volume
(back it up with `pg_dump` before migrating). Static files are rebuilt by `collectstatic`
during the Docker build step.
