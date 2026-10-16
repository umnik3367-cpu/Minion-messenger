# 🍌 Minion Messenger

A **production-grade, full-stack instant messenger** — the best ideas from Telegram, WhatsApp, Discord, Signal and Snapchat, fused into one TypeScript application with a built-in AI sidekick.

```
React 18 + TypeScript + Tailwind + Framer Motion   ⇄   Node 20 + Express + Socket.IO   ⇄   PostgreSQL + Prisma
```

---

## ✨ Feature highlights

### 💬 Messaging
- Realtime 1:1 chats, groups (up to 100k members) and broadcast channels
- Cursor-paginated history with infinite scroll
- Message types: text, image, video, voice, audio, documents, stickers, GIFs, locations
- Replies with quoted preview, edit, soft delete, forward to many chats, pin, save (bookmarks)
- Emoji + quick reactions, per-message hover actions
- **Voice messages** recorded in-browser (`MediaRecorder`) with live waveform
- Typing indicators, read receipts, unread counters, mute & pin per chat
- Global search across people, chats, messages and files

### 📞 WebRTC calls
- Voice & video calling with full signaling over Socket.IO (offer/answer/ICE relay)
- Incoming/outgoing/active call UI, mute mic, toggle camera, call timer
- Call history persisted per user (`MISSED` / `REJECTED` / `ENDED` + duration)

### 🤖 Minion AI — **currently disabled**
- The neural models are switched **off** (`AI_ENABLED=false`) because no working API key is available
- The assistant is hidden in the UI and the API answers `503 AI_DISABLED` instead of pretending
- The whole engine stays in the codebase: set `AI_ENABLED=true` + a key and it comes back
- When enabled it supports chat threads, rewrite/translate/summarize/brainstorm/analyze, image generation,
  a **model switcher** fed by the gateway's `/models`, and **multi-key failover**
  (`MINION_API_KEYS=key1,key2` rotates on `401` dead key / `402` no balance / `429` rate limit / `5xx`)

### 🌍 Languages
- **English and Русский** for the whole interface (auth, chats, settings, admin, AI panel)
- Switch instantly — the picker is on the login screen and in **Settings → Themes**
- The choice is stored on the account (`User.language`) and follows you to new devices
- Minion AI replies in the selected language

### 📧 Real e-mail — notifications *and* password reset
Four transports, first configured one wins: **SMTP**, **Resend**, **Mailgun**, **SendGrid**.
`GET /health` reports which one is live; without any, letters are printed to the console instead.

**Sign-up needs no e-mail confirmation** — an account is usable the moment it is created.

**Transactional (always sent, always logged):**
- Password-reset code
- Both are written to the `EmailNotification` audit table

**Notifications (smart, never spammy):**
- New message → e-mail **only when the recipient is offline or in Do‑Not‑Disturb**
- Contact requests and incoming calls → e-mail nudge
- Respects per‑chat mute, the user's **E‑mail notifications** switch, and throttles to
  **one letter per chat per 15 minutes** (`EMAIL_DIGEST_WINDOW_MIN`)
- Written in the recipient's own language (English / Русский)
- Every attempt (SENT / FAILED / SKIPPED) is auditable at `GET /api/admin/emails`

Try it locally without any provider:
```bash
npm run mail:catcher     # terminal 1 — a real SMTP server writing to .mailbox/
SMTP_HOST=127.0.0.1 SMTP_PORT=2525 SMTP_USER=dev SMTP_PASS=dev npm run start --workspace server   # terminal 2
npm run mail:test        # terminal 3 — 17 checks
```

### 📞 Calls that actually work
`npm run call:test` runs two live Socket.IO clients through a complete call:
presence → `call:invite` → `call:accept` → SDP offer/answer relay → ICE candidates →
`call:reject` path → hang-up → history with duration → and a third user is blocked from
touching someone else's call (18 checks). Media (getUserMedia → RTCPeerConnection) is plain
WebRTC and needs two real browsers or devices for a manual end‑to‑end check.

### 🐣 Stickers
- Upload any image → **automatic background removal** + sticker border via `sharp`
- 512×512 Telegram-compatible output, sticker packs, install/uninstall, discovery feed

### 📸 Status / Stories
- 24-hour text, image and video stories with automatic expiry
- Per-story progress player, viewers list for your own stories
- Realtime "new story" push to your contacts

### 🎨 UX & design
- **8 hand-crafted themes** (dark & light) driven by CSS variables — instant, no reload
- Glassmorphism, gradient brand identity, animated transitions (Framer Motion)
- Fully responsive: mobile-first chat, sidebar drawer, adaptive layout
- Toasts, modals, skeletons, empty states, avatars with presence dots

### 🔐 Security & moderation
- JWT access tokens (15 min) + **rotating opaque refresh tokens** (hash-stored, 30 days)
- bcrypt password hashing (12 rounds) + strength policy
- `tokenVersion` revocation → force-logout every device
- Privacy matrix (last seen / avatar / phone / calls → everyone · contacts · nobody)
- Rate limiting (global, auth brute-force, AI, uploads), Helmet, CORS, Zod validation everywhere
- Admin panel: stats, user ban/promote, reports moderation, AI usage, audit log

---

## 🚀 Quick start

### 1. Prerequisites
- **Node.js ≥ 18.18** (built and tested on Node 24)
- **PostgreSQL ≥ 14 is optional for local development** — if none is running, Minion boots an
  embedded PostgreSQL 17 cluster automatically (no Docker, no admin rights, stored in `.pgdata/`).
  For production, point `DATABASE_URL` at a managed PostgreSQL.

### 2. Install
```bash
npm install
```

### 3. Configure
```bash
cp .env.example .env      # Windows: copy .env.example .env
```
Then edit `.env`:

```env
# Core
PORT=4000
CLIENT_ORIGIN=http://localhost:5173

# Local database (leave as-is to use the embedded dev cluster; no PostgreSQL install needed)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/minion_messenger?schema=public

# Auth secrets (generate: node -e "console.log(require('crypto').randomBytes(48).toString('hex'))")
JWT_ACCESS_SECRET=…
JWT_REFRESH_SECRET=…

# 🤖 Minion AI — any OpenAI-compatible gateway, with key failover
MINION_API_URL=https://api.kosareva.cloud/v1
MINION_API_KEYS=      # comma-separated: key1,key2 → automatic failover
# MINION_API_KEY=     # single-key alternative
MINION_MODEL=gpt-4-1
MINION_MODELS=gpt-4-1,gpt-4-1-mini,alice-ai,gigachat-3-pro   # curated shortcuts
MINION_DISCOVER_MODELS=true   # fill the switcher from the gateway's /models
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_AI_KEY=

# 📧 Real e-mail (SMTP) — reset & notification letters actually arrive
SMTP_HOST=            # smtp.gmail.com · smtp.mailgun.org · smtp.sendgrid.net · smtp.resend.com
SMTP_PORT=587
SMTP_SECURE=false    # true for port 465
SMTP_USER=
SMTP_PASS=
MAIL_FROM=Minion Messenger <no-reply@minion.me>

# 🌍 Languages
DEFAULT_LANGUAGE=en
SUPPORTED_LANGUAGES=en,ru
```

> `MINION_API_KEY` works with **any OpenAI-compatible endpoint** — point `MINION_API_URL` at your gateway (OpenAI, Groq, Together, OpenRouter, a self-hosted vLLM…) and it just works.
>
> ⚠️ **Rotate any key you paste into a chat.** A key shared in a conversation, a screenshot or a commit is compromised — create a fresh one in the provider's dashboard. `.env` is git-ignored; never commit it.

### 4. Database + demo data
```bash
npm run db:setup      # starts a database if needed, applies migrations, seeds demo data
```

### 5. Run
```bash
npm run dev           # database + API (:4000) + client (:5173) — one command
```
Open **http://127.0.0.1:5173** (the dev server is bound to IPv4 loopback; `localhost` may resolve to IPv6-only on Windows).

> If any call fails with "Failed to fetch", the browser could not reach the API — a red banner appears in the app telling you to start the server with `npm run dev`. `concurrently` also restarts a crashed dev process automatically.

> Using your own PostgreSQL? Create it with UTF-8 (required for emoji/unicode content):
> `CREATE DATABASE minion_messenger ENCODING 'UTF8';` — then point `DATABASE_URL` at it and run `npm run db:setup`.

### Demo accounts (created by the seed)
| E-mail | Password | Role |
|---|---|---|
| `admin@minion.me` | `Password123!` | admin |
| `amy@minion.me` | `Password123!` | user |
| `ben@minion.me` | `Password123!` | user |
| `zoe@minion.me` | `Password123!` | user |

Sign in as **amy** in one browser and **ben** in another (or a private window) to see realtime delivery, typing indicators and calls.
---

## 🏗 Project structure

```
minion-messenger/
├── client/                      # React 18 + Vite + Tailwind
│   ├── index.html
│   ├── vite.config.ts           # @/ alias, /api + /socket.io proxy
│   ├── tailwind.config.js       # semantic color tokens from CSS variables
│   └── src/
│       ├── main.tsx  App.tsx
│       ├── components/
│       │   ├── ui/              # Button, Input, Modal, Avatar, Spinner, Toasts
│       │   ├── auth/            # AuthPage (sign in / sign up / reset)
│       │   ├── sidebar/         # Sidebar (chat list, search, stories)
│       │   ├── chat/            # ChatWindow, MessageList, MessageBubble,
│       │   │                    # MessageComposer, VoiceNote, ChatInfoModal,
│       │   │                    # NewChatModal, ForwardModal, SearchModal
│       │   ├── calls/           # CallOverlay (WebRTC UI)
│       │   ├── stories/         # StoriesBar, StoryViewer, CreateStoryModal
│       │   ├── stickers/        # StickerPicker (bg-removal uploader)
│       │   ├── ai/              # MinionAI panel
│       │   ├── settings/        # SettingsPanel (profile/themes/alerts/privacy/contacts)
│       │   └── admin/           # AdminPanel
│       ├── config/themes.ts     # 8 themes + applyTheme()
│       ├── hooks/               # useVoiceRecorder, useWebRTC, useDebounce…
│       ├── services/            # api.ts (fetch + token refresh), socket.ts, *.service.ts
│       ├── store/               # Auth / Chat / Call / Theme / Toast contexts
│       ├── types/               # Shared DTO types
│       └── utils/               # cn, format helpers
├── server/                      # Node + Express + Socket.IO + Prisma
│   └── src/
│       ├── index.ts             # bootstrap: helmet, cors, routes, socket, shutdown
│       ├── config/              # env (zod-validated), prisma singleton
│       ├── middleware/          # auth, validate, rateLimit, upload, error
│       ├── modules/             # auth, users, contacts, chats, messages, stories,
│       │                        # stickers, calls, uploads, search, reports, ai, admin
│       ├── realtime/            # io.ts (singleton + emit helpers), socket.handlers.ts
│       └── utils/               # jwt, password, http, shared, mail, storage
├── database/prisma/
│   ├── schema.prisma            # 24 models / 16 enums
│   └── seed.ts                  # demo users, contacts, chats, channel, sticker pack
├── .env / .env.example
└── package.json                 # npm workspaces + orchestration scripts
```

---

## 🛰 API surface (abridged)

| Method | Route | Purpose |
|---|---|---|
| POST | `/api/auth/register` · `/login` · `/refresh` · `/logout` | session lifecycle |
| POST | `/api/auth/forgot-password` · `/reset-password` | recovery |
| GET/PATCH | `/api/users/me`, `/api/users/:id` | profile, privacy, avatar |
| GET/POST | `/api/contacts`, `/api/contacts/requests` | contacts, requests, block, favourite |
| GET/POST | `/api/chats`, `/api/chats/direct`, `/api/chats/:id/members` | chats, groups, channels |
| POST | `/api/messages/chats/:id/messages` | send (cursor-paginated history) |
| POST | `/api/messages/:id/{reactions,pin,forward,save}` | message actions |
| POST | `/api/stories`, `/api/stories/:id/view` | status/stories |
| POST | `/api/stickers/process` | background removal + border |
| POST | `/api/calls`, `PATCH /api/calls/:id/end` | call records |
| POST | `/api/uploads` | multipart uploads → attachment ids |
| GET | `/api/search?q=` | global search |
| POST | `/api/ai/{chat,rewrite,translate,summarize,ideas,analyze,image}` | Minion AI |
| GET | `/api/admin/{stats,users,reports,ai-usage,audit}` | admin (ADMIN only) |

**Realtime events** (Socket.IO, authenticated via handshake token):
`message:new`, `message:updated`, `message:deleted`, `message:pinned`, `chat:updated`, `chat:joined`, `chat:removed`, `chat:members:changed`, `typing:start/stop`, `chat:read`, `presence:update`, `notification:new`, `contact:request`, `story:new`, `call:invite/accept/reject/end/signal`.

---

## 📜 Scripts

| Command | Description |
|---|---|
| `npm run dev` | Run **database + API + client** together (concurrently) |
| `npm run build` | Type-check + build server and client for production |
| `npm run typecheck` | Strict TypeScript checks for both workspaces |
| `npm run db:setup` | Start a DB if needed → migrate → seed demo data |
| `npm run db:start` / `db:stop` / `db:status` | Control the embedded PostgreSQL |
| `npm run smoke` | End-to-end API test (health, login, chats, send, search, auth) |
| `npm run call:test` | Full call flow with two live Socket.IO clients (18 checks) |
| `npm run mail:test` | E-mail pipeline: no-confirmation sign-up, notifications, throttling, admin log (17 checks) |
| `npm run mail:catcher` | Local SMTP server that captures letters into `.mailbox/` |
| `npm run ai:mock` | Local OpenAI-compatible mock (add `-- --fail-first` to test key failover) |
| `npm run prisma:studio` | Browse the database in Prisma Studio |
| `npm run start` | Run the compiled server (`server/dist`) |

---

## 🛡 Notes, assumptions & production checklist

- **Password-reset codes** are logged to the console (`utils/mail.ts`). Wire a transactional provider (Resend/SES/nodemailer) into `sendMail()` for production — one function, no other changes.
- **Storage** defaults to local disk (`server/uploads/`). `utils/storage.ts` has the S3 seam — drop in `@aws-sdk/client-s3` and set `STORAGE_DRIVER=s3`.
- **Push notifications** use the browser Notification API + in-app sounds. For background push, add Web Push/VAPID using the existing `PushSubscription` model.
- **AI provider of last resort** is chosen automatically; add as many keys as you like, only the first is used.
- Before going live: set strong `JWT_*_SECRET`s, `CLIENT_ORIGIN`, HTTPS (WSS for Socket.IO), a managed `DATABASE_URL`, and put the API behind a reverse proxy (nginx/Caddy) with `STORAGE_DRIVER=s3`.
- Socket.IO runs single-instance (in-memory adapter). For horizontal scaling, switch to `@socket.io/redis-adapter` — the emit helpers (`realtime/io.ts`) stay unchanged.
- `STORAGE_DRIVER=s3`, transactional e-mail and Web Push are deliberately left as one-line integration seams rather than half-implemented SDK calls.

### Troubleshooting
| Symptom | Fix |
|---|---|
| **`Failed to fetch` on login / registration** | The browser can't reach the API. Check the red banner: the dev server or API is down → run `npm run dev` and reload. Use **`http://127.0.0.1:5173`** (Vite is bound to IPv4; `localhost` can resolve to IPv6-only on Windows). |
| Everything is fine but the page is blank | Hard reload (Ctrl+F5) — an old bundle from a previous session can linger |
| `Can't reach database server at localhost:5432` | Run `npm run dev` (it starts the embedded DB) or `npm run db:start`, then `npm run db:setup` |
| Migration fails with `22P05 … no equivalent in encoding WIN1251` | Your PostgreSQL cluster isn't UTF-8 — recreate it: `CREATE DATABASE minion_messenger ENCODING 'UTF8';` (the embedded dev DB is always UTF-8) |
| AI: `The AI assistant is currently disabled` | Expected — the neural models are switched off. Set `AI_ENABLED=true` + a key to bring them back |
| AI: `На балансе недостаточно средств` | The gateway account has no credit — top up at <https://kosareva.cloud/panel/> |
| AI: `key not allowed to access model` | That key is restricted to a subset of models (key #2 serves 14 models) |
| No e-mail arrives | `GET /health` shows `mail: not configured` → set `SMTP_*` / `RESEND_API_KEY` / `MAILGUN_*` / `SENDGRID_API_KEY`, or run `npm run mail:catcher` for local testing |
| UI language doesn't switch | Hard-reload after choosing — the choice is saved on your account |

---

🍌 **Minion Messenger** — TypeScript end-to-end, strict mode, a schema designed to grow, and zero placeholder features.