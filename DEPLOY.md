# Deploy pluffy-showcase

Follow the steps **in order**. Copy and paste each command exactly. Lines starting with `#` are notes, not commands.
If something fails, jump to **Troubleshooting** at the bottom.

## What you are deploying

| Part | This export |
|------|-------------|
| Website / app | Next.js |
| Backend | none (frontend only) |
| Database | – |
| Realtime | no |
| Background jobs | no |

## Deploy without Docker (a plain Linux server)

> Want the easy way? Re-export with Docker: `node mizumi.js export pluffy-showcase nextjs --docker`. This section is for people who prefer to install everything by hand.

### Step 1 — Install on the server
- Node.js 22
- Caddy (free web server that handles HTTPS): see caddyserver.com/docs/install

### Step 2 — Copy the project to the server
```bash
rsync -av --exclude node_modules --exclude vendor --exclude .venv --exclude target ./ root@YOUR_SERVER_IP:/srv/pluffy-showcase/
```

### Step 3 — Database
No backend in this export, so no database.

### Step 4 — Build and start the website
```bash
cd frontend
npm ci
npm run build
npm start        # listens on port 3001
```
It listens on port 3001.
### Step 5 — Make it one public website (HTTPS)
Create `/etc/caddy/Caddyfile` and replace `yourdomain.com`:
```caddyfile
yourdomain.com {
  handle { reverse_proxy localhost:3001 }
}
```
Then: `sudo systemctl reload caddy`. Point your domain's **A record** to the server first.

### Step 6 — Keep it running after you close the terminal
Create one service file per process, for example `/etc/systemd/system/pluffy-showcase-backend.service`:
```ini
[Unit]
Description=pluffy-showcase backend
After=network.target

[Service]
WorkingDirectory=/srv/pluffy-showcase/backend
ExecStart=THE START COMMAND FROM THE BACKEND STEP
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
```
Then: `sudo systemctl daemon-reload && sudo systemctl enable --now pluffy-showcase-backend`. Repeat for the website (change the name and the command).

## Troubleshooting

| Problem | What to do |
|---------|------------|
| Site shows 502 | The process behind Caddy is not running. `sudo systemctl status pluffy-showcase-backend` and `journalctl -u pluffy-showcase-backend -n 50` |
| HTTPS not working | A record must point to the server, ports 80 and 443 must be open |
| Command not found | A tool from Step 1 is missing or too old. Check versions |
