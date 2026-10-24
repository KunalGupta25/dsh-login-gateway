# dsh-login-gateway

## Overview

Official `dsh web` only binds to localhost (`127.0.0.1`) and cannot be exposed directly to public networks. This plugin provides an authenticated entry point with a username/password login wall and reverse proxies authenticated traffic to your local DeepSeek Harness instance.

It is designed for deploying Harness on cloud servers while securely accessing the workspace via browser. By default, it listens on port `8080` under the `/dsh` path prefix to avoid conflicting with the default web port (`3080`).

Repository topics: `dsh-plugin`, `dsh-category-security`. The distributed package contains no hardcoded credentials, domains, or public IPs; credentials are generated and stored exclusively on your host machine.

## Installation

Requires [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness).

```bash
dsh plugin --profile web add github:kunalgupta25/dsh-login-gateway
dsh web --trusted-host YOUR_PUBLIC_IP --trusted-host your.example.com
```

Access in your browser:

```text
http://YOUR_PUBLIC_IP:8080/dsh
```

To install from a local path:

```bash
dsh plugin --profile web add ./dsh-login-gateway
```

## Mobile Access

This plugin focuses on authentication and proxying and preserves the official desktop layout. 

If you primarily access your workspace from a mobile browser, consider pairing it with the community portrait layout plugin [dsh-web-mobile](https://github.com/mexiaosqwq/dsh-web-mobile) in the same `web` profile:

```bash
dsh plugin --profile web add github:kunalgupta25/dsh-login-gateway
dsh plugin --profile web add github:mexiaosqwq/dsh-web-mobile
dsh web --trusted-host YOUR_PUBLIC_IP --trusted-host your.example.com
```

`dsh-login-gateway` manages authentication and routing, while `dsh-web-mobile` (package name `@dsh-external/dsh-mobile-nav`) provides mobile drawers, full-width chat layouts, and touch-adapted settings without affecting desktop displays.

## Quick Start & Configuration

For full setup instructions, configuration fields, and Nginx configurations, see [docs/setup.md](docs/setup.md).

1. **Initial Credentials**: If a password was not preconfigured, find the generated credentials in the `dsh web` console output labeled `generated login`.
2. **Gateway Settings**: After logging in, navigate to **Settings → Gateway** and click **Gateway Settings** to change username, password, port, and proxy options. You can also log out from this panel.
3. **Trusted Hosts**: The startup command must include `--trusted-host <host>`, or official API endpoints will reject incoming traffic.
4. **Reverse Proxy / Nginx**: If placing Nginx or Caddy in front, enable "Trust Reverse Proxy" and proxy `/dsh`, `/assets/`, `/plugins/`, `/api`, `/dsh-image-gen`, and `/query-balance` to the gateway port.
5. **Document Drops**: Dragging non-image documents (PDF, Markdown, text) into the browser uploads them to the server and inserts `@filename` references directly into the input prompt. Standard image attachments (PNG, JPEG, WebP, GIF) continue through the native media bar.

## Documentation

- [Setup Guide](docs/setup.md)

## Development

```bash
npm install
npm test
```

## License

MIT
