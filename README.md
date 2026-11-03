# Retro Bowl High School — experimental fan mod

This archive edits the supplied Retro Bowl College game and its existing data files. It is not an official release, an exact commercial-quality reproduction, or a fully playtested production build. The original game engine and artwork remain. Only `backend-worker.js` is a new file in the archive.

## Included changes

- 50 state conferences and 200 selected real schools, four schools per state. This is a curated selection, not an exhaustive US school database.
- Mississippi: Northeast Lauderdale, West Lauderdale, Meridian, Oak Grove. Northeast and West Lauderdale face each other in the regular-season finale.
- Twelve regular-season games: weeks 1–11 and week 16. Weeks 12–15 are open dates. Week 16 contains reciprocal rivalry pairings for every school. These are mod-assigned rivalries, not a researched list of official rivalries.
- State schedule or national schedule. National mode replaces the first six games with interstate opponents; the last six stay within the state. Choose the mode before generating a new season. Existing season fixtures do not change retroactively.
- A separate postseason tournament for each state. Two-, three-, or four-team brackets use state standings, play-ins/byes where applicable, and finals in weeks 17–20. Formats repeat across states; these are custom compact playoffs, not all 50 official association rules or classifications. There is no national championship bracket.
- Recruiting, coach hiring, tutoring, morale/conditioning/level-up meetings, and Shopping.txt prices reduced to roughly one quarter, rounded up. Not every incidental event or facility price is discounted; no real-money purchase system is changed.
- Freshman-to-senior progression uses ages 14–17 instead of 18–21.
- School names, mod uniform presets, state guide and high-school wording. Colors, ratings, and weather are presets, not verified school statistics. Some original college-specific graphics and story text remain.
- Separate RBHS save namespace. Start a fresh career. Do not import old college saves.
- Local two-player exhibition through the engine's existing human-vs-human setting. Select the two schools in HS MENU. Share the mouse/touchscreen and alternate possession. This does not add human-controlled defense or simultaneous two-keyboard controls.
- Experimental online head-to-head: the host runs this same exhibition engine, streams its canvas through WebRTC, and accepts guest pointer input for home-team possessions. Host plays away offense; guest plays home offense. Defense is simulated. Host selects both schools and handles pregame/menu navigation. Video only, no streamed game audio. Host must keep the game open and visible.

## Run the game

Extract the ZIP and serve the entire folder from a static web host. Keep `index.html` and `html5game/` together. Opening index.html directly with `file://` is not supported because the engine reads data files over HTTP.

For local single-device play, run `python -m http.server 8080` inside this folder and open `http://localhost:8080`. Online play requires HTTPS, or localhost for development.

The original portal loader was removed from index.html so this copy can start directly on your host. Original portal SDK files are retained but the page no longer loads them.

## Deploy the single Cloudflare Worker

1. Create a Cloudflare Worker using ES-module syntax. Replace its contents with `backend-worker.js`.
2. Create one Workers KV namespace and bind it to the Worker as `RBHS_KV`.
3. Add the text environment variable `ALLOWED_ORIGIN` equal to your exact game origin, such as `https://your-game.example`, without a trailing slash. This is the address hosting the game, not the backend address.
4. Deploy the Worker and assign its custom domain `backend.arthyion.com` in your Cloudflare account. This archive does not deploy it or modify DNS.
5. From the configured game origin, `https://backend.arthyion.com/api/health` should respond with `ok: true`. Opening it directly in the address bar has no Origin header and is intentionally rejected.
6. In HS MENU choose Host game. Copy the generated invite and share it privately. The guest opens it and clicks Join game. Once connected, the host selects Play in the original exhibition screen.

API: POST `/api/rooms` with an offer creates a room and returns host/guest bearer tokens. Authenticated GET `/api/rooms/{id}` returns an offer to the guest or an answer to the host. Guest-only POST `/api/rooms/{id}/answer` submits the answer. Room and answer records expire after one hour. Closing a connection stops local media; room records remain until expiry. Create a new invite after disconnecting.

KV is used only for signaling. It is eventually consistent, so room creation or answers may take a minute to appear in another location. If Join reports the room unavailable, wait and retry. There is one guest per invite; do not distribute the link publicly. Invitations are bearer capabilities, not user accounts. No matchmaking, ranked results, server-authoritative anti-cheat, or durable reconnect support is included. Apply Cloudflare traffic limits appropriate to your deployment before opening room creation to the public.

Live video and input use WebRTC, not KV polling. A public STUN server is configured. Some networks require a TURN relay. In Connection settings enter your own ICE servers array, including temporary TURN credentials if needed, on both devices. The Worker does not provision TURN. Do not place permanent private TURN credentials in a publicly shared build.

## Verification and remaining limitations

Passed locally:
- Syntax checks for the engine, embedded page JavaScript and backend.
- Both schedule modes across all twelve template years: no self-games or double bookings; twelve games per team; six interstate games in national mode; all finale rivalry pairings.
- Every state's bracket dependency graph across twenty randomized standings/results scenarios: same-state participants, no double bookings, and fifty champions.
- Targeted execution of edited engine cost, age, postseason seed and schedule-loader functions using lightweight GML API stand-ins.
- Worker request tests with a mock KV store: CORS, invalid payloads, capability authentication, host/guest separation and offer/answer exchange.
- Archive manifest: all original files remain; backend-worker.js is the only added file.

Not verified: full browser startup/rendering, a full played career, exhibition gameplay, real Cloudflare deployment, and a live two-device WebRTC match. The build environment had no browser binary and its browser download was blocked. Those checks are required before calling this production-ready. The mock KV test does not model eventual-consistency delays or Cloudflare quotas. Long school names may need visual adjustment in original game screens.

Reference: Lauderdale County schools https://www.lauderdale.k12.ms.us/apps/pages/index.jsp?pREC_ID=1743092&type=d&uREC_ID=1543380
Cloudflare KV consistency: https://developers.cloudflare.com/kv/concepts/how-kv-works/
Cloudflare KV writes: https://developers.cloudflare.com/kv/api/write-key-value-pairs/
