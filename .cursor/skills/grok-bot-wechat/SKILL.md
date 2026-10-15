---
name: Grok Bot 接入微信
description: >-
  Use when the user wants to connect Grok Bot to WeChat (ClawBot / iLink), set
  up or repair the box forwarder, bind QR for bot_token, wire a webhook Routine,
  or diagnose session expiry (-14).
---
# Grok Bot 接入微信

Connect WeChat ClawBot (iLink) to Grok Bot so inbound WeChat text wakes a Routine, and replies go back through a local forwarder on the box.

This skill is the reusable recipe. Keep account-specific values (webhook URL, keys, bot_token, allowlisted WeChat user ids, machine ids) out of the skill; put those in env, state files, or the Routine panel.

## Architecture

1. **WeChat ClawBot (iLink)** — phone-bound bot session; long-poll `getupdates`, send via `sendmessage`.
2. **Box forwarder** — Node process on the box (default `http://127.0.0.1:17890`) that polls iLink, POSTs signed wakes to a Grok Bot webhook Routine, and exposes `POST /reply` for outbound text.
3. **Webhook Routine** — trigger `{ "type": "webhook" }`; prompt tells the agent how to treat inbound WeChat text and how to reply via the forwarder.

Quota for the Grok Bot agent is the user's Grok Bot plan. Do not route replies through a third-party LLM billing path.

## Prerequisites

- User wants WeChat ↔ this (or a named) Grok Bot agent.
- Box can run Node and keep a long-lived process.
- User can scan a ClawBot QR on their phone WeChat.

## Steps

### 1. Create the webhook Routine

Read the Cursor-managed `routines` skill, then create a Routine with:

- **trigger:** `{ "type": "webhook" }` (no schedule).
- **name:** something clear like `微信下旨入口`.
- **prompt (intent, not tool schemas):** On webhook wake from WeChat/iLink, treat the payload's `text` as a message from the user via WeChat. Answer in the Grok Bot chat as usual for the task. To send a reply back into WeChat, POST plain text to the box forwarder's `/reply` with `to_user_id` and `context_token` from the wake (or stored context). Strip Markdown to plain text before sending. Only process allowlisted senders when an allowlist is configured. Never log or paste webhook keys, bot_token, or context_token into chat. Deliver the result to the user in this chat; when the wake is a duplicate or empty, stay quiet.

After save, point the user at the Routine panel links from `<automation_status>` for **Webhook URL**, **Webhook key**, and **Authorization header**. You never see or need the key yourself; the forwarder gets it as env from the user or a prior setup.

### 2. Place the forwarder on the box

Canonical layout (reuse if already present under `/home/box/wechat-bridge/`):

- `forwarder.mjs` — iLink poll + webhook POST + local HTTP API
- `start.sh` — `exec node forwarder.mjs` (no secrets in the file)
- `package.json` + `node_modules` as needed
- `state/` — `bot.json`, `cursor.json`, `contexts.json`, `allowlist.json`, per-user `ctx-*.token` (mode 0600)

Listen only on `127.0.0.1` (default port `17890`). Never bind publicly.

**Local HTTP API:**

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | `hasBotToken`, `polling`, `webhookConfigured`, `sessionExpired` |
| GET | `/status` | richer status |
| POST | `/reply` | `{ "to_user_id", "context_token", "text" }` |
| POST | `/set-bot-token` | `{ "bot_token", "baseurl?" }` |
| POST | `/qr/start` | start ClawBot QR (`bot_type=3`) |
| GET | `/qr/status?qrcode=` | poll until `confirmed` + save `bot_token` |

**Env (required for wakes):**

- `WECHAT_WEBHOOK_URL` — https webhook endpoint from the Routine panel
- `WECHAT_WEBHOOK_SECRET` — webhook key / Bearer material from the panel
- `PORT` — optional, default 17890

Signing: Standard Webhooks style (`webhook-id`, `webhook-timestamp`, `webhook-signature: v1,…`) plus `Authorization: Bearer …` as required by Grok Bot webhooks. Never log URL, secret, bot_token, or context_token.

**iLink base:** `https://ilinkai.weixin.qq.com`. Outbound only: long-poll `ilink/bot/getupdates`, send `ilink/bot/sendmessage`. Headers include `AuthorizationType: ilink_bot_token` and `Authorization: Bearer <bot_token>` when set, plus a random `X-WECHAT-UIN`.

If the forwarder tree is missing, recreate it from this recipe (same API and security rules) rather than inventing a different port or public bind.

### 3. Start the forwarder

```bash
cd /home/box/wechat-bridge
# export WECHAT_WEBHOOK_URL and WECHAT_WEBHOOK_SECRET from the Routine panel (user provides or already in env)
./start.sh
```

Run detached / with a pid file as needed so it survives the shell. Confirm `GET http://127.0.0.1:17890/health` returns JSON. Until QR bind, expect `hasBotToken: false` and no polling.

### 4. Phone QR bind (bot_token)

1. `POST /qr/start` → show the user the QR (image or URL from the response).
2. User scans with WeChat and confirms ClawBot.
3. Poll `GET /qr/status?qrcode=…` until `status: confirmed` (forwarder saves `bot_token` into `state/bot.json`).
4. Health should show `hasBotToken: true` and `polling: true`.

On session expiry **ret/errcode -14**: stop blind retry, set `sessionExpired`, ask the user to re-scan QR (same `/qr/start` flow). Do not thrash getupdates.

### 5. Allowlist and plain text

- Optional `state/allowlist.json`: JSON array of WeChat `from_user_id` strings. Non-empty list drops everyone else.
- Inbound: only message_type inbound text/voice-transcription; skip empty.
- Outbound: strip Markdown/code fences to plain text; chunk ~1800 chars per sendmessage.

### 6. End-to-end check

1. User sends a short WeChat text to ClawBot.
2. Forwarder wakes the Routine (webhook 200/202).
3. Agent handles the wake; reply via `POST /reply` with plain text.
4. User sees the reply in WeChat.

### 7. Acceptance checklist

- [ ] Allowlist honored when configured
- [ ] Replies are plain text (no raw Markdown)
- [ ] Forwarder restart keeps `bot_token` / cursor from `state/`
- [ ] On **-14**, stop polling loop thrash; prompt re-login via QR
- [ ] No secrets in logs, chat, README, or start.sh
- [ ] Listen only on 127.0.0.1

## Repair / diagnose

1. `curl -s http://127.0.0.1:17890/health` — check `hasBotToken`, `webhookConfigured`, `polling`, `sessionExpired`, `lastError`.
2. Missing webhook env → re-export from Routine panel links; restart forwarder.
3. `sessionExpired` or -14 → QR rebind; clear expired flag on successful `/set-bot-token` or QR confirm.
4. Wakes not arriving → confirm Routine enabled, URL/secret match, HTTPS only.
5. Replies fail → need fresh `context_token` from the latest inbound message.

## Never

- Put webhook keys, bot_token, or context_token in skill text, git, or user-visible logs.
- Bind the forwarder on `0.0.0.0` or a public port.
- Auto-reply on WeChat without going through the agent's normal send rules for consequential actions.
- Promise OpenClaw / third-party xAI billing as the path; this bridge uses Grok Bot quota via the webhook Routine.
