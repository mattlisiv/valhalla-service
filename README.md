# valhalla-service

Valhalla routing service behind an Nginx reverse proxy with HTTPS certificates from Let's Encrypt.

## What this stack does

- Runs `valhalla` only on the internal Docker network (`valhalla:8002`).
- Publishes only `nginx` on ports `80` and `443`.
- Terminates TLS in Nginx for `api.pasalo.io`.
- Uses Certbot for certificate issuance and automatic renewal.

## 1) Prerequisites (AWS + DNS)

1. Point DNS `A` record for `api.pasalo.io` to your EC2 public IP.
2. EC2 Security Group inbound rules:
   - `80/tcp` from `0.0.0.0/0` (Let's Encrypt challenge + HTTP redirect)
   - `443/tcp` from `0.0.0.0/0`
3. Install Docker + Docker Compose plugin on EC2.

## 2) Project setup

```bash
git clone https://github.com/mattlisiv/valhalla-service.git .
mkdir -p valhalla/custom_files
cd valhalla/custom_files
wget https://download.geofabrik.de/north-america/us-latest.osm.pbf
cd ../..
```

## 3) Issue the first TLS certificate

For first-time issuance, stop anything using port 80 and run Certbot in standalone mode:

```bash
docker compose run --rm --service-ports certbot \
  certonly --standalone \
  -d api.pasalo.io \
  --agree-tos --no-eff-email \
  -m admin@pasalo.io
```

> Replace `admin@pasalo.io` with your real notification email.

## 4) Start services

```bash
docker compose up -d
```

This starts:
- `valhalla` (internal only)
- `nginx` (public `80/443`)
- `certbot` (background renew loop)

## 5) Certificate renewals

Renewals are handled automatically by the `certbot` service every 12 hours. Nginx reads the certs from `./certbot/conf`.

After a successful renewal, reload Nginx to pick up renewed certs:

```bash
docker compose exec nginx nginx -s reload
```

(Optional) add this as a cron job on the host:

```cron
0 3 * * * cd /path/to/valhalla-service && docker compose exec -T nginx nginx -s reload
```

## Security notes for publishing this repo

- **Do not commit TLS private keys** from `certbot/conf`.
- **Do not commit map data** from `valhalla/custom_files`.
- Keep your EC2 instance patched and restrict SSH access to trusted IPs.

A `.gitignore` is included to prevent common sensitive/runtime files from being committed.
