# DOC Relay Visualizer

Directional Overcurrent relay operating zone visualizer.
Supports cross-polarization (phase-to-phase) and zero-sequence (earth fault) methods.

## Quick Start

```bash
# 1. Clone / copy this folder to your server
# 2. Build and start
docker compose up -d --build

# App is now at http://your-server-ip:3500
```

## Expose via your domain (Nginx reverse proxy)

If you already have Nginx on the host managing your domain:

```nginx
# /etc/nginx/sites-available/relay.yourdomain.com
server {
    listen 80;
    server_name relay.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name relay.yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/relay.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/relay.yourdomain.com/privkey.pem;

    location / {
        proxy_pass         http://localhost:3500;
        proxy_http_version 1.1;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Enable site and get SSL cert
sudo ln -s /etc/nginx/sites-available/relay.yourdomain.com /etc/nginx/sites-enabled/
sudo certbot --nginx -d relay.yourdomain.com
sudo nginx -s reload
```

## Expose via Traefik

Uncomment the Traefik labels in `docker-compose.yml` and set your domain.
Make sure the `web` network matches your Traefik network name.

## Useful commands

```bash
# View logs
docker compose logs -f

# Rebuild after editing index.html
docker compose up -d --build

# Stop
docker compose down

# Check container health
docker ps
```

## File structure

```
doc-relay-viz/
├── public/
│   └── index.html       # entire app (single file, no dependencies)
├── nginx/
│   └── default.conf     # nginx server config
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Changing the port

Edit `docker-compose.yml`:
```yaml
ports:
  - "8080:80"   # host_port:container_port
```
Then `docker compose up -d --build`.
