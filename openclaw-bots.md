# Creating a New OpenClaw Bot (Step-by-Step)

Replace BOTNAME with your bot's name (e.g., chicabot) and pick unused ports.

## 1. Build the image (only needed once or on updates)

```bash
cd ~/repos/openclaw
docker build -t openclaw:local -f Dockerfile .
```

## 2. Create and fix permissions on the config directory

```bash
mkdir -p ~/.openclaw-BOTNAME/workspace
sudo chown -R 1000:1000 ~/.openclaw-BOTNAME
```

## 3. Run onboarding

```bash
cd ~/repos/openclaw
export COMPOSE_PROJECT_NAME=BOTNAME
export OPENCLAW_CONFIG_DIR=~/.openclaw-BOTNAME
export OPENCLAW_WORKSPACE_DIR=~/.openclaw-BOTNAME/workspace
export OPENCLAW_HOME_VOLUME="BOTNAME_home"
export OPENCLAW_GATEWAY_PORT="127.0.0.1:PORT_A"
export OPENCLAW_BRIDGE_PORT="127.0.0.1:PORT_B"
dockercompose run --rm openclaw-cli onboard
```

When prompted:
- Onboarding mode: Manual
- Config handling: Update values (or fresh if first time)
- Gateway bind: LAN (0.0.0.0)
- Gateway port: 18789 (keep default, Docker handles the mapping)
- Tailscale: Off

## 4. Add allowedOrigins to config

Edit ~/.openclaw-BOTNAME/openclaw.json and add inside the "gateway" block:

```json
"controlUi": {
  "allowedOrigins": ["http://localhost:PORT_A", "http://127.0.0.1:PORT_A"]
}
```

## 5. Start the gateway

COMPOSE_PROJECT_NAME=BOTNAME docker compose up -d openclaw-gateway

## 6. Access the dashboard (from your local machine)

```bash
ssh -N -L PORT_A:127.0.0.1:PORT_A charizard10@svc-01.local
```

Open http://localhost:PORT_A/ in your browser.

## 7. Approve device pairing

Get the token:

```bash
grep token ~/.openclaw-BOTNAME/openclaw.json
```
List pending devices:

```bash
COMPOSE_PROJECT_NAME=BOTNAME docker compose exec openclaw-gateway node dist/index.js
devices list --token "TOKEN"
```

Approve:
```bash
COMPOSE_PROJECT_NAME=BOTNAME docker compose exec openclaw-gateway node dist/index.js
devices approve REQUEST_ID --token "TOKEN"
```

---
Port allocation

```
┌───────────┬──────────────┬─────────────┐
│    Bot    │ Gateway Port │ Bridge Port │
├───────────┼──────────────┼─────────────┤
│ chicabot  │ 18791        │ 18792       │
├───────────┼──────────────┼─────────────┤
│ freddybot │ 18793        │ 18794       │
├───────────┼──────────────┼─────────────┤
│ next bot  │ 18795        │ 18796       │
└───────────┴──────────────┴─────────────┘
```

Daily management (optional aliases for .bashrc)

```bash
alias chicabot="COMPOSE_PROJECT_NAME=chicabot docker compose"
alias freddybot="COMPOSE_PROJECT_NAME=freddybot docker compose"
```
