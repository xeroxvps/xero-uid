# XEROX UID Operator

A responsive UID workspace for importing Facebook numeric IDs, fetching available public profile metadata, organizing saved entries, and running the frontend and API together from one production container.

[![Live](https://img.shields.io/badge/status-live-22c55e)](https://uids.xirxsms.xyz)
[![Render](https://img.shields.io/badge/deploy-Render-7c3aed)](https://xerox-uid.onrender.com)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](#license)

## Live application

- Primary domain: **https://uids.xirxsms.xyz**
- Render fallback: **https://xerox-uid.onrender.com**
- Health check: **https://xerox-uid.onrender.com/api/health**

Render's free instance can sleep when idle, so the first request after inactivity may take a short time.

## Features

- Import one UID per line or `uid|optional-password`
- Preserve duplicate input lines exactly as entered
- Fetch available public name, username, picture, follower count, and Instagram signal
- Full and compact card views
- Saved-items workspace, search, selection, copy, retry, and delete controls
- Light/dark themes and mobile-first interface
- Browser-only persistence for imported entries and preferences
- React single-page app and Flask API served by one Docker container
- Render Blueprint and health-check support

## Technology

| Layer | Technology |
| --- | --- |
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Backend | Python, Flask, Gunicorn |
| Package manager | pnpm 10.18.0 |
| Deployment | Docker and Render |
| Storage | Browser `localStorage` |

## Project layout

```text
.
├── artifacts/
│   ├── api-server/fb_server.py    # Production Flask API and SPA server
│   └── uid-web/                   # React/Vite frontend
├── Dockerfile                     # Multi-stage production image
├── render.yaml                    # Render Blueprint
├── requirements.txt               # Python runtime dependencies
├── pnpm-lock.yaml                 # Reproducible JS dependency lock
└── SECURITY.md                    # Security and disclosure policy
```

## Local setup

### Requirements

- Node.js 20 or newer
- pnpm 10.18.0
- Python 3.11 or newer

### Frontend

```bash
corepack enable
corepack prepare pnpm@10.18.0 --activate
pnpm install
pnpm --filter @workspace/uid-web run dev
```

The development UI starts on `http://localhost:5173` and proxies `/api` requests to `http://localhost:8080`.

### API

In a second terminal:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
PORT=8080 python artifacts/api-server/fb_server.py
```

### Termux

Keep the project under the Termux home directory, not `/storage/emulated/0`, because Android shared storage does not support the symlinks required by pnpm.

```bash
pkg update -y
pkg install -y nodejs-lts python git
npm install -g pnpm@10.18.0
cp -r /sdcard/Download/XEROTIC-UID "$HOME/xerox-uid"
cd "$HOME/xerox-uid"
pnpm install
pip install -r requirements.txt
```

Start the API:

```bash
cd "$HOME/xerox-uid"
PORT=8080 python artifacts/api-server/fb_server.py
```

Start the frontend in another Termux session:

```bash
cd "$HOME/xerox-uid"
pnpm --filter @workspace/uid-web run dev
```

## Production build

```bash
pnpm install --frozen-lockfile
pnpm --filter @workspace/uid-web run typecheck
pnpm --filter @workspace/uid-web run build
docker build -t xerox-uid .
docker run --rm -p 10000:10000 xerox-uid
```

Verify the container:

```bash
curl http://localhost:10000/api/health
```

Expected response:

```json
{"status":"ok"}
```

## Environment variables

| Variable | Required | Default | Purpose |
| --- | --- | --- | --- |
| `PORT` | No | `5002` locally; Render injects it | HTTP listen port |
| `STATIC_DIR` | No | Frontend build directory | Location of compiled SPA assets |
| `ALLOWED_ORIGINS` | No | Production, Render, and local URLs | Comma-separated CORS allowlist |
| `FB_DEFAULT_COOKIE` | No | Empty | Optional server-side Facebook cookie |

Copy `.env.example` when preparing a local environment. Never commit the completed `.env` file or any real cookie.

## API

### Health

```http
GET /api/health
```

### Fetch UID metadata

```http
POST /api/fb/uid/fetch
Content-Type: application/json
```

```json
{
  "uids": [
    { "uid": "100012345678901" }
  ]
}
```

Requests accept 1–100 items and each UID must contain 5–20 digits. The API does not persist imported values or echo passwords in its response.

## Deploy to Render

1. Fork or connect this repository in Render.
2. Choose **Blueprint** and select `render.yaml`, or create a Docker web service.
3. Keep the health check path as `/api/health`.
4. Add secrets such as `FB_DEFAULT_COOKIE` only through Render's Environment settings.
5. Deploy and confirm that the health endpoint returns HTTP `200`.

The included Dockerfile builds the web application in a Node stage, copies only the compiled assets and Python server into the runtime image, then runs Gunicorn as a non-root user.

## Security and privacy

- No credentials, cookies, imported entries, or local databases belong in Git.
- Imported data and preferences are stored in the current browser profile.
- A Facebook cookie entered in Settings is sensitive browser data; use a dedicated authorized account and clear it from Settings when no longer needed.
- Production HTTPS is provided by Render/Cloudflare.
- API payload size, UID format, and batch size are restricted.
- Outbound HTTPS certificate verification is enabled.
- CORS and browser security headers are configured by the Flask application.

See [SECURITY.md](SECURITY.md) for vulnerability reporting and secret-handling rules.

## Troubleshooting

### pnpm `EACCES` on Termux

Move the project into `$HOME`. Do not install dependencies inside `/sdcard` or `/storage/emulated/0`.

### Rollup Android package missing

Use the pinned pnpm version and reinstall inside Termux home:

```bash
rm -rf node_modules
pnpm install --force
```

### `ModuleNotFoundError`

Install the pinned Python requirements:

```bash
pip install -r requirements.txt
```

### Render first request is slow

The free service may be waking from sleep. Wait briefly and retry the health endpoint.

## Responsible use

Use this project only with data and accounts you own or are authorized to process. Follow applicable privacy laws, platform terms, and account-security requirements. The project is not affiliated with or endorsed by Facebook or Meta.

## License

Released under the MIT license. See the package metadata for the current license declaration.
