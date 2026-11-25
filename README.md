# PRIME MD — Pairing Portal

Central pairing portal for **PRIME MD** (purple-royal edition):

- **Pair page (`index.html`)** — server list with live user counts and limits.
  Pick a server, enter a WhatsApp number, and the portal asks the bot server
  for a pairing code (the code is created on the bot server itself — the portal
  only proxies: `/api/proxy` → the bot's `/api/status` + `/api/code`).
- **Admin panel (`/admin`)** — after login you can add / edit / delete servers
  (name, bot URL, user limit). Login credentials come from the `ADMIN_USER` /
  `ADMIN_PASS` environment variables; the session is kept in a signed httpOnly
  cookie. Server changes are written to the portal's live storage (Vercel KV)
  through the same store layer the original portal used, with the bundled
  `data/servers.json` as the starting point / fallback.

## Deploy (Vercel)

1. Import this repo into Vercel.
2. Project → Storage → create a **KV (Upstash)** store and connect it to the
   project (required for saving servers from the admin panel — without it the
   list stays whatever is bundled in `data/servers.json`).
3. Project → Settings → Environment Variables:
   - `ADMIN_USER` — admin username
   - `ADMIN_PASS` — admin password
4. Deploy. Then open `/admin`, log in, and confirm SERVER 01 points at your
   PRIME MD bot domain (default: `https://web-production-5c642de2.up.railway.app`).

## Bot side (PRIME MD API)

The portal talks to each bot server over these endpoints:

- `GET /api/status` → `{ ok, bot, serverId, serverName, sessions, limit, pairingPending, commands, uptimeSec }`
  — the portal reads the live user count from the `sessions` field.
- `GET /api/code?number=<number>` → `{ ok, status, code, message }` where
  `status` is one of:
  - `code_ready` — `code` holds the pairing code (shown big, with copy support)
  - `already_paired` — the number is already connected; a friendly note is shown
  - `rate_limited` — too many attempts; the bot's message asks the user to wait
  - `server_full` — the server's user limit is reached
  - `try_again` / `invalid_number` — shown as a plain error message

The portal also enforces each server's configured user limit before asking
the bot for a new code, using the live count from `/api/status`.
