# OmniRoute Docker Deployment

Standard production layout for OmniRoute behind Nginx.

## Structure

```text
omniroute/
├── docker-compose.yaml
├── .env
├── .env.example
├── .gitignore
├── README.md
└── nginx/
    └── omniroute.conf
```

## 1. Configure environment

```bash
cp .env.example .env
nano .env
```

Generate the WebSocket bridge secret:

```bash
openssl rand -hex 32
```

Put the generated value into `OMNIROUTE_WS_BRIDGE_SECRET`.

Set your real public hostname:

```dotenv
NEXT_PUBLIC_BASE_URL=https://omniroute.example.com
```

## 2. Start OmniRoute

```bash
docker compose pull
docker compose up -d
```

Check:

```bash
docker compose ps
docker compose logs -f omniroute
```

## 3. Local health check

The host port defaults to `30128`:

```bash
curl http://127.0.0.1:30128/api/monitoring/health
```

## 4. Nginx

Copy the Nginx configuration:

```bash
cp nginx/omniroute.conf /etc/nginx/sites-available/omniroute.conf
ln -s /etc/nginx/sites-available/omniroute.conf /etc/nginx/sites-enabled/omniroute.conf
```

Edit the hostname in the config if needed.

Test and reload:

```bash
nginx -t
systemctl reload nginx
```

## 5. DNS

Create an A/AAAA record for your subdomain pointing to the server.

Example:

```text
omniroute.example.com -> SERVER_IP
```

## 6. SSL

After DNS resolves:

```bash
certbot --nginx -d omniroute.example.com
```

Then access:

```text
https://omniroute.example.com
```

## Port mapping

The Compose file intentionally binds the host port to localhost:

```text
127.0.0.1:30128 -> container:20128
```

This prevents direct Internet access to OmniRoute on port 30128. Public traffic should go through Nginx/HTTPS.

To change the host port, edit `.env`:

```dotenv
OMNIROUTE_PORT=30228
```

Then:

```bash
docker compose up -d
```

The internal OmniRoute port remains `20128`.
