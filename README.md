# ChickenStrike2Servers — Cloudflare edition

This package replaces the always-on Node.js signaling server with **Cloudflare Workers + SQLite-backed Durable Objects**.

## What it does

- `POST /api/rooms` reserves a 6-character room for 24 hours and returns `room`, `inviteUrl`, `wsUrl`, and `expiresAt`.
- `GET /api/rooms/:code` checks the reservation.
- `GET /ws?room=CODE` is the WebSocket signaling endpoint used by the patched game.
- `/invite/CODE` shows a small invite page; `/game/?room=CODE` opens the game with the code prefilled.
- Durable Object alarms expire rooms after 24 hours. Rooms are stored in Cloudflare SQLite-backed Durable Objects.
- WebSocket hibernation keeps idle connections connected without requiring a continuously running Node process.

**Important:** this is still WebRTC peer-to-peer gameplay. The player who hosts the match must keep Chicken Strike open and connected. If the host closes the game, the match stops; the 24-hour reservation does not run the game simulation on Cloudflare.

## Deploy from a computer or phone

1. Create a new GitHub repository and upload the contents of this folder (not the ZIP as a single file).
2. Open the repository in Cloudflare Workers & Pages, or use Wrangler from a development environment.
3. Install dependencies with `npm install`.
4. Deploy with `npx wrangler deploy`.
5. Cloudflare will give you a URL like `https://chickenstrike2servers.<your-subdomain>.workers.dev`.
6. Set this URL as the server URL in the Android app. The app's `POST /api/rooms` request is supported.

For a direct Wrangler deployment, the commands are:

```bash
npm install
npx wrangler deploy
```

## Testing

- Open `/health` — should return `{ "ok": true, ... }`.
- Create a room with `POST /api/rooms` — should return a room code and invite URL.
- Open the invite URL — it should show the room code and an **Open game** link.

## Notes

- Free Workers plan limits apply. Current Cloudflare docs list 100,000 Workers requests/day, and Durable Objects have their own daily request and duration allocations. Exceeding free allocations can cause requests to fail until limits reset. Check current usage in the Cloudflare dashboard.
- The 24-hour expiration is enforced by a Durable Object alarm. Alarms run independently of user traffic.
- If the Worker is deleted or its Durable Object data is deleted, reservations are lost.
- The Android project is separate; point its server URL field at the deployed `https://...workers.dev` URL.
