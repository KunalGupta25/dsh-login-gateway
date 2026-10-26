# Setup Guide

This page explains how to configure the username, ports, and reverse proxy after installing `dsh-login-gateway`. The published `cordis.patch.yml` does not contain a password.

## First Access

1. Confirm that you have run:

   ```bash
   dsh plugin --profile web add github:kunalgupta25/dsh-login-gateway
   dsh web --trusted-host YOUR_PUBLIC_IP --trusted-host your.example.com
   ```

2. Open `http://YOUR_PUBLIC_IP:8080/dsh` in your browser. The default port is **8080** and must differ from the local `dsh web` port, or the plugin will refuse to start.

3. If you have not set a password, find `generated login` in the terminal or service log where `dsh web` is running, or open `$DSH_HOME/cloud-gateway-state.json` (permissions `0600`).

4. After logging in, open the official **Settings → Gateway** page and select **Gateway Settings**. You can change the username, password, listen address, port, path, and whether to trust Nginx. Settings are stored only in the local state file and are not written back to the plugin package. Press Esc or click outside the dialog to close it. You can also log out from the same page.

5. To use the workspace in portrait mode on a phone, install the community plugin [dsh-web-mobile](https://github.com/mexiaosqwq/dsh-web-mobile):

   ```bash
   dsh plugin --profile web add github:mexiaosqwq/dsh-web-mobile
   ```

   It does not conflict with this plugin. The gateway handles login, while the portrait plugin handles the narrow-screen layout. The desktop layout is almost unchanged.

## Configuration Methods, in Priority Order

1. Environment variables: `DSH_CLOUD_USERNAME` / `DSH_CLOUD_PASSWORD` / `DSH_CLOUD_SECRET`
   These are useful with systemd. Fields locked by environment variables cannot be changed in the settings page.
2. A local profile override at `~/.dsh/profiles/web/cordis.patch.yml`
3. The **Gateway Settings** page after login, or the random password generated at first startup

The profile patch replaces the entire `config` block for that entry. Do not provide only one field:

```yaml
# ~/.dsh/profiles/web/cordis.patch.yml
- id: cloud-gateway
  config:
    listenHost: 0.0.0.0
    listenPort: 8080
    basePath: /dsh
    username: admin
    password: change-me-to-a-long-password
    trustProxy: false
```

If you do not set `secret`, the session key is persisted in `$DSH_HOME/cloud-gateway-state.json`, so a restart will not log out every user.

## Fields

| Field | Default | Meaning |
|---|---|---|
| `listenHost` | `0.0.0.0` | Public listen address. Use `0.0.0.0` on a cloud server or `127.0.0.1` for local debugging |
| `listenPort` | `8080` | Public port. It must differ from the `dsh web` port |
| `basePath` | `/dsh` | Browser access prefix |
| `username` | `admin` | Login username |
| `password` | empty generates one | Login password. At least 8 characters are recommended |
| `secret` | empty generates and persists one | Session HMAC key |
| `upstreamHost` | `webStartup.host` | Local Harness address |
| `upstreamPort` | `webStartup.port` | Local Harness port |
| `trustProxy` | `false` | Enable only behind a trusted reverse proxy to trust `X-Forwarded-*` headers |
| `secureCookie` | automatic | `true` always enables Secure; empty enables it only for HTTPS requests |
| `cookiePath` | `/` | Cookie path. Keep `/` when the site root also serves `/assets`, `/plugins`, and `/api` |

## `--trusted-host`

You must declare the public host at startup. Otherwise, the official fence will block regular model endpoints and WebSocket connections after login:

```bash
dsh web --trusted-host YOUR_PUBLIC_IP --trusted-host your.example.com
```

The model, provider directory, and Agent preset controls in the settings page use the privileged `settings.describe` API, which accepts only loopback requests. After login, this plugin rewrites those requests to the local Host and removes the browser Fetch Metadata that would trigger a 403, so the settings page also works from the cloud gateway.

## Using Nginx

The plugin listens on `0.0.0.0:8080` by default. To use ports 80/443 without a port in the URL, set `trustProxy: true` and proxy all of these paths to the gateway:

- `/dsh`
- `/assets/`
- `/plugins/`
- `/api`
- `/dsh-image-gen` (the image-generation plugin's image RPC, served at the site root rather than under `/dsh`)
- `/query-balance` (the `dsh-balance` balance query, also served at the site root)
- `/favicon.svg` `/favicon.ico` `/manifest.webmanifest`

If you proxy only `/dsh` and omit the root-level static resources and APIs, the page will be blank after login. Image previews request `/dsh-image-gen/...`; without that route, the image is generated but cannot be displayed. The balance plugin requests `/query-balance`; without that route, the official balance may work while the interface reports that the balance is unavailable.

```nginx
location /dsh { proxy_pass http://127.0.0.1:8080; }
location /assets/ { proxy_pass http://127.0.0.1:8080; }
location /plugins/ { proxy_pass http://127.0.0.1:8080; }
location /api { proxy_pass http://127.0.0.1:8080; }
location /dsh-image-gen { proxy_pass http://127.0.0.1:8080; }
location /query-balance { proxy_pass http://127.0.0.1:8080; }
```

Put TLS in front of the gateway in production. This plugin provides a login layer, not HTTPS. Over HTTP, the username and password travel in plain text.

## Dropping Documents

The official input accepts only png/jpeg/webp/gif files. When you drag a PDF, Markdown file, or text file from the browser, this plugin saves it to `$DSH_HOME/uploads/` and inserts an `@filename` resource reference into the conversation. The interface shows only the filename link, not the full path. Image drops continue through the official attachment bar.

## Security Notes

- Do not put a real password in the `cordis.patch.yml` that you plan to publish.
- `X-Forwarded-For` is not trusted by default, which prevents forged client IPs from bypassing login rate limits.
- `/assets/` and `/plugins/` require login.
- Session cookies use `HttpOnly` and `SameSite=Lax`; HTTPS requests also receive `Secure`.
- On a shared domain, `cookiePath: /` sends the session to other sites on the same host. A dedicated subdomain is safer.
