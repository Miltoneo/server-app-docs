# Troubleshooting

This section contains common issues and their solutions for the server-app ecosystem.

## Nginx 404 Errors for Django Applications

### Problem
Django applications return `404 Not Found` errors when accessed via nginx, even though the services are running.

### Symptoms
- HTTP requests to application URLs (e.g., `https://onkoto.com.br/sisu/`) return 404
- Gunicorn services are active and running
- Socket files exist but may be incorrect type (regular file instead of socket)

### Root Cause
The issue occurs when systemd socket activation is enabled for Django services, but the services are configured with `--bind` to a specific Unix socket path. This creates a conflict:

- Socket activation listens on `/run/app.sock`
- `--bind` attempts to bind to `/var/server-app/run/app.sock`
- Nginx upstream points to `/var/server-app/run/app.sock`
- Result: nginx cannot connect to the backend

Additionally, leftover regular files from failed socket creation can prevent proper socket binding.

### Solution
For applications with socket activation enabled (check with `systemctl is-enabled app.socket`):

1. **Disable socket activation:**
   ```bash
   sudo systemctl disable app.socket
   sudo systemctl stop app.socket
   ```

2. **Remove any existing incorrect socket file:**
   ```bash
   rm /var/server-app/run/app.sock  # if it exists as regular file
   ```

3. **Restart the service:**
   ```bash
   sudo systemctl restart app.service
   ```

4. **Verify:**
   - Service is active: `systemctl status app.service`
   - Socket file exists and is type `srwxrwxrwx`: `ls -la /var/server-app/run/app.sock`
   - No `TriggeredBy` in service status (indicates direct binding, not socket activation)

### Affected Applications
This issue affected: sisu, emprestimos, rotinas

Working applications (construtora, medicos, etc.) have socket activation disabled and bind directly.

### Prevention
Ensure consistency between systemd configuration and nginx upstreams. Applications should either:
- Use socket activation without `--bind` in service
- Use direct binding with `--bind` and disabled socket activation

### Related Files
- `/etc/systemd/system/app.service`
- `/etc/systemd/system/app.socket`
- `/etc/nginx/sites-available/onkoto.com.br`