# Nginx Configuration

This document describes the nginx configuration for the server-app ecosystem, handling multiple Django applications under the onkoto.com.br domain.

## Overview

The nginx setup uses a single server block for `onkoto.com.br` and `www.onkoto.com.br` with HTTPS (port 443) and HTTP redirect (port 80). It serves multiple Django applications as subdirectories, each proxied to their respective Gunicorn Unix sockets.

## Architecture

### Upstream Definitions

Each Django application has an upstream block defining the Unix socket connection:

```nginx
upstream app_name_app {
    server unix:/var/server-app/run/app_name.sock fail_timeout=0;
}
```

Defined applications:
- Construtora
- Medicos
- Emprestimos
- SISU
- Mais Médicos
- Rotinas
- TDS (IoT)
- Eleição

### Server Block Structure

#### HTTPS Server (Port 443)
- **SSL/TLS**: Managed by Certbot with Let's Encrypt certificates
- **Security Headers**: HSTS, X-Content-Type-Options, X-Frame-Options, X-XSS-Protection
- **Logging**: Separate access and error logs
- **Client Settings**: 20M max body size, 128k body buffer

#### Location Blocks

Each application has a dedicated location block:

```nginx
location /app_name/ {
    proxy_pass http://app_name_app;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off;
    proxy_buffering off;
    
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

#### Static and Media Files

- **Application-specific static/media**: Served directly with caching
  - `/static/app_name/` → `/var/server-app/www/app_name/static/`
  - `/media/app_name/` → `/var/server-app/www/app_name/media/`

- **Legacy static/media**: Fallback locations
  - `/static/` → `/var/server-app/www/static/`
  - `/media/` → `/var/server-app/www/media/`

#### Root Location
- Serves static website files from `/var/www/html`
- Custom error pages for 404 and 5xx errors

### HTTP Server (Port 80)
- Redirects all traffic to HTTPS
- Handles Let's Encrypt ACME challenges

## URL Routing

| URL Path | Application | Socket |
|----------|-------------|--------|
| `/construtora/` | Construtora | construtora.sock |
| `/medicos/` | Medicos | medicos.sock |
| `/emprestimos/` | Emprestimos | emprestimos.sock |
| `/sisu/` | SISU | sisu.sock |
| `/maismedicos/` | Mais Médicos | maismedicos.sock |
| `/rotinas/` | Rotinas | rotinas.sock |
| `/tds/` | TDS (IoT) | tds.sock |
| `/eleicao/` | Eleição | eleicao.sock |
| `/static/app/` | Static files | Direct serve |
| `/media/app/` | Media files | Direct serve |
| `/` | Static site | Direct serve |

## Deployment

The configuration is deployed to `/etc/nginx/sites-available/onkoto.com.br` and enabled via symlink to `/etc/nginx/sites-enabled/`.

After changes:
```bash
sudo nginx -t
sudo nginx -s reload
```

## Security Considerations

- HTTPS enforced with HSTS
- Security headers applied
- Client certificate validation disabled
- Large request support for file uploads

## Monitoring

- Access logs: `/var/log/nginx/onkoto-access.log`
- Error logs: `/var/log/nginx/onkoto-error.log`
- Gunicorn logs per application in `/var/server-app/logs/apps/app_name/`

## Troubleshooting

See [Troubleshooting Guide](../guides/troubleshooting.md) for common nginx-related issues.